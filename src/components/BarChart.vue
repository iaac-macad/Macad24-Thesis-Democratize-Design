<template>
  <div>
    <canvas ref="barChart"></canvas>
  </div>
</template>

<script setup>
import { onMounted, ref, watch } from "vue";
import { Chart, registerables } from "chart.js";
Chart.register(...registerables);

const barChart = ref(null);
const props = defineProps(['metadata']);

onMounted(() => {
  renderChart();
});

watch(props.metadata, renderChart);

function renderChart() {
  if (!barChart.value) return;

  const context = barChart.value.getContext("2d");
  new Chart(context, {
    type: "bar",
    data: {
      labels: ["Metadata 0", "Metadata 1", "Metadata 2"],
      datasets: [{
        label: "Values",
        data: props.metadata.map(item => parseFloat(item.value) || 0),
        backgroundColor: ["#FF6384", "#36A2EB", "#FFCE56"]
      }]
    },
    options: {
      responsive: true,
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });
}
</script>

<style scoped>
canvas {
  max-width: 100%;
}
</style>
