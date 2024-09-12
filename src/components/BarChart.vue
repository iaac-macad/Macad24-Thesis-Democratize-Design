<template>
  <div class="chart-container" v-if="shouldRenderChart">
    <!-- ECharts component for rendering the bar chart -->
    <v-chart :option="chartOptions" autoresize />
  </div>
  <div v-else>
    <p>A bar chart will be displayed here once the floor data is received.</p>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue';
import { use } from 'echarts/core';
import VChart from 'vue-echarts';
import { BarChart } from 'echarts/charts';
import { GridComponent, TooltipComponent, TitleComponent } from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';

// Register necessary ECharts components
use([BarChart, GridComponent, TooltipComponent, TitleComponent, CanvasRenderer]);

// Define props to accept metadata as an array of numbers
const props = defineProps({
  metadata: {
    type: Array,
    required: false,  // Set required to false
    default: () => null  // Default to null if not provided
  }
});

// Computed property to check if metadata is present
const shouldRenderChart = computed(() => Array.isArray(props.metadata) && props.metadata.length > 0);

// Define reactive options for the chart
const chartOptions = ref({
  title: {
    text: 'Room Numbers Per Floor',  // Title text
    left: 'center',  // Center the title horizontally
    top: '5%',  // Position it slightly from the top
    textStyle: {
      color: '#FFFFFF',  // White color for the title
      fontFamily: 'Roboto',  // Use Roboto font
      fontSize: 14,  // Font size for the title
    },
  },
  grid: {
    left: '15%',  // Adjust left padding to provide more space for y-axis title
    right: '10%',  // Adjust right padding
    top: '20%',  // Adjust top padding to make space for the title
    bottom: '15%',
  },
  xAxis: {
    type: 'category',
    data: ['1 BED', '2 BED', '3 BED'], // Labels for the data points
    axisLabel: {
      color: '#FFFFFF',  // White text color
      fontFamily: 'Roboto',  // Use Roboto font
    },
  },
  yAxis: {
    type: 'value',
    nameLocation: 'middle',  // Align title to the center
    nameTextStyle: {
      color: '#FFFFFF',  // White text color for axis name
      fontFamily: 'Roboto',  // Use Roboto font
      fontSize: 14,  // Adjust font size as needed
    },
    axisLabel: {
      color: '#FFFFFF',  // White text color
      fontFamily: 'Roboto',  // Use Roboto font
    },
  },
  series: [
    {
      type: 'bar',
      data: (props.metadata || []).map((value, index) => ({
        value,  // The value of the bar
        itemStyle: {
          color: ['rgb(255, 128, 0)', 'rgb(0, 170, 255)', 'rgb(255, 0, 255)'][index]  // Different color for each bar
        }
      })),
      showBackground: true,
      backgroundStyle: {
        color: 'rgba(180, 180, 180, 0.2)',
      },
      label: {
        show: true,
        position: 'top',
        color: '#FFFFFF',  // White text color for bar labels
        fontFamily: 'Roboto',  // Use Roboto font
      },
    },
  ],
  textStyle: {
    color: '#FFFFFF',  // Global white text color
    fontFamily: 'Roboto',  // Global font family
  },
});

// Watch for changes in props to update the chart dynamically
watch(
  () => props.metadata,
  (newMetadata) => {
    if (Array.isArray(newMetadata) && newMetadata.length > 0) {
      // Update with new data and apply colors to each bar
      chartOptions.value.series[0].data = newMetadata.map((value, index) => ({
        value,
        itemStyle: {
          color: ['rgb(255, 128, 0)', 'rgb(0, 170, 255)', 'rgb(255, 0, 255)'][index],  // Re-apply colors
        },
      }));
    } else {
      chartOptions.value.series[0].data = [];  // Clear the data if metadata is not present
    }
  }
);
</script>

<style scoped>
.chart-container {
  width: 100%; /* Ensure full width */
  height: 250px; /* Set a fixed height */
  max-width: 600px; /* Optionally set a maximum width */
  margin: 0 auto; /* Center the chart */
}

/* Ensure Roboto font is loaded */
body {
  font-family: 'Roboto', monospace; /* Apply Roboto font */
}
</style>