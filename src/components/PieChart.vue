<template>
  <div>
    <canvas ref="pieChart"></canvas>
  </div>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import { Chart, registerables } from "chart.js";
Chart.register(...registerables);

const pieChart = ref(null);
const props = defineProps(['metadata']);

onMounted(() => {
  renderChart();
});

watch(props.metadata, renderChart);

function renderChart() {
  if (!pieChart.value) return;

  const context = pieChart.value.getContext("2d");
  new Chart(context, {
    type: "pie",
    data: {
      labels: ["Metadata 0", "Metadata 1", "Metadata 2"],
      datasets: [{
        data: props.metadata.map(item => parseFloat(item.value) || 0),
        backgroundColor: ["#FF6384", "#36A2EB", "#FFCE56"]
      }]
    },
    options: {
      responsive: true
    }
  });
}
</script>

<style scoped>
canvas {
  max-width: 100%;
}
</style>
