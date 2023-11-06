<template>
  <div>
    <highchart :options="chartOptions" :modules="[chartData.chart.type]"></highchart>
  </div>
</template>

<script setup>
import { ref } from "vue";
const props = defineProps(['chartData'])

const chartOptions = ref({
  chart: props.chartData.chart,
  title: props.chartData.title,
  subtitle:  props.chartData.subtitle,
  plotOptions: {
    networkgraph: {
      keys: props.chartData.keys,
      layoutAlgorithm: {
        enableSimulation: props.chartData.enableSimulation,
        linkLength: props.chartData.linkLength,
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
  series: props.chartData.series
});
</script>
