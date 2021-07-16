<template>
  <bar-chart :chartdata="chartdata"
    :options="options" />
</template>

<script>
import BarChart from "./BarChart.vue";

export default {
  name: "Chart",
  components: { BarChart },
  props: ["data"],
  computed: {
    chartdata() {
      return {
        labels: ["Data since day 1"],
        datasets: [
          {
            label: "Total Confirmed",
            backgroundColor: "#0000ff",
            data: [this.data.TotalConfirmed],
          },
          {
            label: "Total Deaths",
            backgroundColor: "#ff0000",
            data: [this.data.TotalDeaths],
          },
          {
            label: "Total Recovered",
            backgroundColor: "#00ff00",
            data: [this.data.TotalRecovered],
          },
        ],
      };
    },
  },
  data: () => ({
    options: {
      responsive: true,
      maintainAspectRatio: false,
    },
  }),
};
</script>
