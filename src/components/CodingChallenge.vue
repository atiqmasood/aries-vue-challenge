<template>
  <div>
    <h1>Options Profit Calculator</h1>
    <div class="graph">
      <line-chart :data="chartData" :options="chartOptions"></line-chart>
    </div>
    <div class="max-p-l-break">
      <p>Max Profit: {{ maxProfit }}</p>
      <p>Max Loss: {{ maxLoss }}</p>
      <p>Break-even Points: {{ breakEvenPoints.join(", ") }}</p>
    </div>
  </div>
</template>

<script>
import "core-js/stable";
import { Line } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
} from "chart.js";
ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale
);

export default {
  name: "CodingChallenge",
  props: {
    optionsData: {
      type: Array,
      required: true,
      validator: function (value) {
        // throw error in the console if the options more then 4
        return value.length <= 4;
      },
    },
  },
  components: {
    LineChart: Line,
  },
  computed: {
    chartData() {
      return {
        labels: this.prices, // Prices of the underlying at expiry
        datasets: [
          {
            label: "Profit/Loss",
            data: this.profits,
            borderColor: "rgba(75, 192, 192, 1)",
            backgroundColor: "rgba(75, 192, 192, 0.2)",
            fill: false,
          },
        ],
      };
    },
    chartOptions() {
      return {
        responsive: true,
        scales: {
          x: {
            type: "linear",
            position: "bottom",
            title: {
              display: true,
              text: "Underlying Price at Expiry",
            },
          },
          y: {
            title: {
              display: true,
              text: "Profit/Loss",
            },
          },
        },
      };
    },
    prices() {
      const minPrice =
        Math.min(...this.optionsData.map((c) => c.strike_price)) - 20; // Extending the range 20 units below the minimum strike price
      const maxPrice =
        Math.max(...this.optionsData.map((c) => c.strike_price)) + 20; // Extending the range 20 units above the maximum strike price
      const prices = [];
      for (let i = minPrice; i <= maxPrice; i++) {
        prices.push(i);
      }
      return prices;
    },
    profits() {
      return this.prices.map((price) => {
        return this.optionsData.reduce((acc, contract) => {
          const strikePrice = contract.strike_price;
          const premium = (contract.bid + contract.ask) / 2;
          const type = contract.type;
          const longShort = contract.long_short;

          let profit = 0;

          if (type === "Call") {
            if (longShort === "long") {
              profit = Math.max(0, price - strikePrice) - premium;
            } else {
              profit = premium - Math.max(0, price - strikePrice);
            }
          } else if (type === "Put") {
            if (longShort === "long") {
              profit = Math.max(0, strikePrice - price) - premium;
            } else {
              profit = premium - Math.max(0, strikePrice - price);
            }
          }

          return acc + profit;
        }, 0);
      });
    },
    maxProfit() {
      return Math.max(...this.profits);
    },
    maxLoss() {
      return Math.min(...this.profits);
    },
    breakEvenPoints() {
      return this.prices.filter((price, index) => this.profits[index] === 0);
    },
  },
};
</script>

<style scoped>
h1 {
  text-align: center;
}
.graph {
  max-height: 400px;
  display: flex;
  justify-content: center;
}
.max-p-l-break {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 24px;
}
</style>
