<template>
  <div class="container">
    <div class="chart-container">
      <canvas ref="myChart" width="950" height="500"></canvas>
    </div>
    <div class="table-container">
      <table v-if="data" class="custom-table">
        <thead>
          <tr class="header-row">
            <th></th>
            <th>AAPL</th>
            <th>MSFT</th>
            <th>TSLA</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="odd-row">Annualized Return</td>
            <td class="odd-row">{{ data.AAPL?.annualized_return }}</td>
            <td class="odd-row">{{ data.MSFT?.annualized_return }}</td>
            <td class="odd-row">{{ data.TSLA?.annualized_return }}</td>
          </tr>
          <tr>
            <td class="even-row">Annualized Volatility</td>
            <td class="even-row">{{ data.AAPL?.annualized_volatility }}</td>
            <td class="even-row">{{ data.MSFT?.annualized_volatility }}</td>
            <td class="even-row">{{ data.TSLA?.annualized_volatility }}</td>
          </tr>
          <tr>
            <td class="odd-row">Cumulative Return</td>
            <td class="odd-row">{{ data.AAPL?.cumulative_return }}</td>
            <td class="odd-row">{{ data.MSFT?.cumulative_return }}</td>
            <td class="odd-row">{{ data.TSLA?.cumulative_return }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
  flex-wrap: wrap;
}

.chart-container {
  flex: 1;
}

.table-container {
  flex: 1;
  padding-left: 20px;
  padding-top: 50px;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.custom-table thead {
  background-color: #f2f2f2;
}

.custom-table th,
.custom-table td {
  padding: 10px;
  border-bottom: 1px solid #ccc;
}

.custom-table .header-row th {
  background-color: grey;
  color: white;
}

.custom-table .even-row {
  background-color: #f9f9f9;
}

.custom-table .odd-row {
  background-color: #e9e9e9;
}
</style>

<script>
import axios from 'axios';
import { Chart, LineController, LineElement, PointElement, LinearScale, Title, CategoryScale, Legend } from 'chart.js';

Chart.register(LineController, LineElement, PointElement, LinearScale, Title, CategoryScale, Legend);

export default {
  data() {
    return {
      data: null
    };
  },
  mounted() {
    axios.get('https://backend-stocks-ivor.herokuapp.com/')
      .then(response => {
        const stocksData = response.data;
        this.data = stocksData; // Assign the received API data to the 'data' property
        this.renderChart();
      })
      .catch(error => {
        console.error(error);
      });
  },
  methods: {
    renderChart() {
      if (!this.$refs.myChart || !this.data) { // Check if the 'data' property is defined
        return;
      }

      const labels = this.data.AAPL.dates;
      const datasets = ['AAPL', 'MSFT', 'TSLA'].map(key => {
        return {
          label: key,
          data: this.data[key].performance,
          fill: false,
          hoverBackgroundColor: "rgba(255,99,132,0.4)",
          hoverBorderColor: '#' + Math.floor(Math.random() * 16777215).toString(16),
          borderColor: '#' + Math.floor(Math.random() * 16777215).toString(16), // random color
        }
      });
      const chartData = {
        labels: labels,
        datasets: datasets
      };

      this.$nextTick(() => {
        const options = {
          responsive: false,
          title: {
            display: true,
            text: 'Stock Performance'
          },
          scales: {
            x: {
              display: true,
              title: {
                display: true,
                text: 'Date [Day.Month.Year]'
              }
            },
            y: {
              display: true,
              title: {
                display: true,
                text: 'Price [$]'
              }
            }
          },
          plugins: {
            title: {
              display: true,
              text: 'Stock Performance'
            },
            legend: {
              display: true,
              position: 'chartArea',
              align: 'end',
              labels: {
                color: '#000000'
              }
            },
            tooltip: {
              enabled: true,
              mode: 'nearest',
              intersect: true,
              callbacks: {
                title: function(tooltipItem) {
                  return 'Date: ' + tooltipItem[0].xLabel;
                }
              }
            }
          }
        };

        const ctx = this.$refs.myChart.getContext('2d');
        new Chart(ctx, {
          type: 'line',
          data: chartData,
          options: options
        });
      });
    }
  }
};
</script>




