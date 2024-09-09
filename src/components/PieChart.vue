<template>
  <div class="chart-container" v-if="shouldRenderChart">
    <!-- ECharts component for rendering the semi-circle pie chart -->
    <v-chart :option="chartOptions" autoresize />
  </div>
  <div v-else>
    <p>No data available to display the chart.</p>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue';
import { use } from 'echarts/core';
import VChart from 'vue-echarts';
import { PieChart } from 'echarts/charts';
import { TooltipComponent, LegendComponent, TitleComponent } from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';

// Register necessary ECharts components
use([PieChart, TooltipComponent, LegendComponent, TitleComponent, CanvasRenderer]);

// Define props to accept data for the semi-circle pie chart
const props = defineProps({
  data: {
    type: Array,
    required: true,  // Set required to true
  }
});

// Computed property to check if data is present
const shouldRenderChart = computed(() => Array.isArray(props.data) && props.data.length > 0);

// Debugging: Log received data
console.log('Received data for PieChart:', props.data);

// Define reactive options for the semi-circle pie chart
const chartOptions = ref({
  tooltip: {
    trigger: 'item'
  },
  legend: {
    top: '5%',
    left: 'center',
    textStyle: {
      color: '#FFFFFF',  // White color for the legend text
      fontFamily: 'Roboto',  // Use Roboto font
    }
  },
  series: [
    {
      name: 'Access From',
      type: 'pie',
      radius: ['40%', '70%'],  // Donut chart style
      center: ['50%', '70%'],  // Center vertically towards the bottom
      startAngle: 180,  // Start angle for semi-circle
      endAngle: 360,  // End angle for semi-circle
      avoidLabelOverlap: false,
      label: {
        show: false,
        position: 'center'
      },
      emphasis: {
        label: {
          show: true,
          fontSize: 20,
          fontWeight: 'bold'
        }
      },
      labelLine: {
        show: false
      },
      data: props.data || [],  // Use data from props directly or an empty array if not provided
    }
  ]
});

// Watch for changes in props to update the chart dynamically
watch(
  () => props.data,
  (newData) => {
    if (Array.isArray(newData) && newData.length > 0) {  // Check if data exists and is not empty
      console.log('Updating chart data:', newData);  // Debugging log
      chartOptions.value.series[0].data = newData;  // Update with new data directly
    } else {
      chartOptions.value.series[0].data = [];  // Clear the data if not present
    }
  },
  { immediate: true }
);
</script>

<style scoped>
.chart-container {
  width: 100%; /* Ensure full width */
  height: 300px; /* Set a fixed height */
  max-width: 600px; /* Optionally set a maximum width */
  margin: 0 auto; /* Center the chart */
}

/* Ensure Roboto font is loaded */
body {
  font-family: 'Roboto', monospace; /* Apply Roboto font */
}
</style>
