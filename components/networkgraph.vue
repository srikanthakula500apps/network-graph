<template>
  <div class="container">
    <!-- Select Dropdown -->
    <v-select
      v-model="selectedFromValue"
      clearable
      label="Select"
      :items="uniqueFromValues"
      variant="outlined"
    ></v-select>
    <div
      @wheel.prevent="handleMouseWheel"
      @mousedown.prevent="startPan"
      @mousemove="pan"
      @mouseup="stopPan"
    >
      <highchart
        :style="{ transform: `scale(${zoom}) translate(${panX}px, ${panY}px)` }"
        :options="chartOptions"
        :modules="[chartData.chart.type]"
      ></highchart>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
const props = defineProps(["chartData"]);
const zoom = ref(1);
const isPanning = ref(false);
const panStartX = ref(0);
const panStartY = ref(0);
const panX = ref(0);
const panY = ref(0);
const selectedFromValue = ref("");

// Get the unique 'from' values from the data
const uniqueFromValues = computed(() => {
  const fromValues = new Set();
  props.chartData.series[0].data.forEach((item) => {
    fromValues.add(item[0]);
  });
  const unique = Array.from(fromValues).map((item) => item.replace("frontend-plugin-", ""))
  return unique;
});

// Filter the data based on the selected 'from' value
const filteredChartOptions = computed(() => {
  if (!selectedFromValue.value) {
    return props.chartData.series[0].data;
  } else {
    // Filter data based on the selected 'from' value
    const filteredSeries = props.chartData.series[0].data.filter(
      (item) => item[0].replace("frontend-plugin-", "") === selectedFromValue.value
);
    return filteredSeries;
  }
  
});
const filteredNodes = computed(() => {
  if (!selectedFromValue.value) {
    return props.chartData.series[0].nodes;
  } else {
    return [];
  }
});

const filteredHeight = computed(() => {
  if (!selectedFromValue.value) {
    return "100%";
  } else {
    return "30%";
  }
});
const filteredLength = computed(() => {
  if (!selectedFromValue.value) {
    return props.chartData.linkLength;
  } else {
    return 20;
  }
});

const chartOptions = ref({
  chart: {
    type: "networkgraph",
    marginTop: 20,
    height: filteredHeight,
  },
  title: props.chartData.title,
  subtitle: props.chartData.subtitle,

  plotOptions: {
    networkgraph: {
      keys: props.chartData.keys,
      layoutAlgorithm: {
        enableSimulation: props.chartData.enableSimulation,
        linkLength: filteredLength,
        friction: props.chartData.friction,

        initialPositions: function () {
          const chart = this.series[0].chart,
            width = chart.plotWidth,
            height = chart.plotHeight;
          this.nodes.forEach(function (node) {
            // If initial positions were set previously, use that

            // positions. Otherwise use random position:

            node.plotX =
              node.plotX === undefined ? Math.random() * width : node.plotX;
            node.plotY =
              node.plotY === undefined ? Math.random() * height : node.plotY;
          });
        },
      },
    },
  },

  series: props.chartData.series,
  series: [
    {
      dataLabels: {
        enabled: true,
        linkFormat: "",
        style: {
          fontSize: "0.8em",
          fontWeight: "normal",
        },
      },
      nodes: filteredNodes,
      data: filteredChartOptions,
    },
  ],
});

const handleMouseWheel = (event) => {
  // Adjust the zoom level based on the direction of the mouse wheel
  zoom.value += event.deltaY > 0 ? -0.1 : 0.1;

  // Ensure the zoom level stays within a certain range (optional)
  zoom.value = Math.min(Math.max(this.zoom, 0.5), 3);
};

const startPan = (event) => {
  isPanning.value = true;
  panStartX.value = event.clientX;
  panStartY.value = event.clientY;
};

const pan = (event) => {
  if (isPanning.value) {
    panX.value += event.clientX - panStartX.value;
    panY.value += event.clientY - panStartY.value;
    panStartX.value = event.clientX;
    panStartY.value = event.clientY;
  }
};

const stopPan = () => {
  isPanning.value = false;
};
</script>
<style scoped>
.container {
  margin-top: 10px;
}
</style>
