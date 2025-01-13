<template>
  <div>
    <h3>Sales Data Visualization</h3>
    <div class="container">
      <div class="controls">
        <label for="chartType">Choose Chart Type:</label>
        <!--here binding the chart type to its variable and than updating the chart when it changes-->
        <select id="chartType" v-model="chartType" @change="updateChartType">
          <option value="bar">Bar Chart</option>
          <option value="line">Line Chart</option>
          <option value="pie">Pie Chart</option>
        </select>
      </div>

      <canvas id="sales-chart"></canvas>

      <div class="input-data">
        <label for="customData">Enter Custom Sales Data (comma-separated):</label>
        <!--here the custom data which the users type will bind to customsalesdata variable -->
        <input
          id="customData"
          type="text"
          v-model="customSalesData"
          placeholder="e.g., 500, 800, 400, 900, 700, 600, ..."
        />
        <!--here when the users enters the input tahan the sales data will be updated through this button-->
        <button @click="updateCustomData">Update Data</button>
      </div>
        <!--here the csv file will from user can be uploaded -->
      <div class="csv-upload">
        <label for="csvUpload">Upload CSV File:</label>
        <!--here when the file is uploaded handlefileupload will be triggered to handle the csv file-->
        <input id="csvUpload" type="file" accept=".csv" @change="handleFileUpload" />
      </div>

      <div class="insights">
        <!--for Explanability or action insight purpose-->
        <p><strong>Total Sales:</strong> {{ totalSales }} USD</p>
        <p><strong>Average Sales per Month:</strong> {{ averageSales.toFixed(2) }} USD</p>
        <p><strong>Best Month:</strong> {{ bestMonth }} ({{ highestSales }} USD)</p>
      </div>
    </div>
  </div>
</template>

<script>
//importing css, chart.js and d3.js 
import '@/assets/Styles/SalesChart.css';
import { Chart } from 'chart.js/auto';
import * as d3 from 'd3';

export default {
  data() {
    return {
      chart: null,
      //generated manual data for displaying the charts 
      salesData: [500, 800, 400, 900, 700, 600, 450, 850, 750, 950, 650, 550],
      labels: ['January','February','March','April', 'May','June','July','August','September','October','November','December',],
      //to show the data or input by the user 
      chartType: 'bar',
      customSalesData: '',
    };
  },
  //for calculation of the insights of salesdata enterd by users 
  computed: {
    //used array to add all sales data
    totalSales() {
      if (!Array.isArray(this.salesData) || this.salesData.length === 0) {
        return 0;
      }
      return this.salesData.reduce((sum, value) => sum + value, 0);
    },
    //for calculating the average sales by dividing with the length of sales data which is months
    averageSales() {
      if (!Array.isArray(this.salesData) || this.salesData.length === 0) {
        return 0;
      }
      return this.totalSales / this.salesData.length;
    },
    //for finding the highest month sales using maximum with the highest values too.
    bestMonth() {
      if (!Array.isArray(this.salesData) || this.salesData.length === 0) {
        return 'N/A';
      }
      const maxSales = Math.max(...this.salesData);
      const index = this.salesData.indexOf(maxSales);
      return this.labels[index] || 'N/A';
    },
    highestSales() {
      if (!Array.isArray(this.salesData) || this.salesData.length === 0) {
        return 0;
      }
      return Math.max(...this.salesData);
    },
  },
  mounted() {
    //component is created in webpage 
    this.createChart();
  },
  methods: {
    //used for updating charts got from chart.jsand for deleting the previous charts to avoid conflict
    createChart() {
      if (this.chart) {
        this.chart.destroy();
      }
      //setting up the charts to display
      const ctx = document.getElementById('sales-chart').getContext('2d');
      this.chart = new Chart(ctx, {
        type: this.chartType,
        data: {
          labels: this.labels,
          datasets: [
            {
              label: 'Sales (in Rupees)',
              data: this.salesData,
              backgroundColor: [
                '#f8f9fa','#e9ecef','#dee2e6','#ced4da','#adb5bd','#6c757d','#495057','#343a40','#212529','#adb5bd','#e9ecef','#f8f9fa',
              ],
              borderColor: '#212529',
              borderWidth: 1,
            },
          ],
        },
        options: {
          //making chart responsive 
          responsive: true,
          plugins: {
            legend: {
              display: true,
              labels: {
                color: '#333',
              },
            },
            //setting up the hover data information
            tooltip: {
              backgroundColor: '#000',
              titleColor: '#fff',
              bodyColor: '#eee',
              borderColor: '#212529',
              borderWidth: 1,
              callbacks: {
                title: (tooltipItems) => {
                  return `Month: ${tooltipItems[0].label}`;
                },
                label: (tooltipItem) => {
                  return `Sales: $${tooltipItem.raw}`;
                },
              },
            },
          },
        },
      });
    },
    updateChartType() {
      this.createChart();
    },
    updateCustomData() {
      if (this.customSalesData.trim() === '') {
        alert('Please enter valid data!');
        return;
      }

      const newData = this.customSalesData
      //changing values of string to array
        .split(',')
        .map((value) => parseInt(value.trim(), 10));
      if (newData.some(isNaN)) {
        alert('Invalid input. Please enter only numbers separated by commas.');
        return;
      }

      if (newData.length !== this.labels.length) {
        alert(`Please enter ${this.labels.length} values to match the months.`);
        return;
      }

      this.salesData = newData;
      this.createChart();
    },
    //for uploading and handling the csv files 
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const text = e.target.result;
          this.parseCSV(text);
        };
        reader.readAsText(file);
      }
    },
    parseCSV(data) {
      try {
        //using d3 to parse the data of csv
        const parsedData = d3.csvParse(data);

        if (!parsedData || parsedData.length === 0) {
          alert('Uploaded file is empty or invalid.');
          return;
        }

        // here the parsing will be done assuming two columns are only there and in such a way that first column is label and the other is data
        this.labels = parsedData.map((row) => row[parsedData.columns[0]]);
        this.salesData = parsedData.map((row) => parseFloat(row[parsedData.columns[1]]) || 0);

        this.createChart();
      } catch (error) {
        console.error('Error parsing CSV:', error);
        alert('An error occurred while parsing the CSV file. Please check the file format.');
      }
    },
  },
};
</script>
