<template>
  <div class="tooltip_div">
    <div class="header">
      <strong>{{ data.name }} ({{ data.total ?? "0" }})</strong>
      <v-icon color="error" icon="mdi-close-box" @click="closeModal" />
    </div>
    <div v-if="data.total" ref="chartContainer" class="chart-container"></div>
    <div v-else>No data to show</div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, watch, ref, Ref } from "vue";
import * as d3 from "d3";

import { DataProps } from "../types";

interface Props {
  data: DataProps;
  years: number[];
}

const props = defineProps<Props>();

const emit = defineEmits<{
  (e: "close"): void;
}>();

const chartContainer: Ref<HTMLDivElement | null> = ref(null);

const closeModal = (): void => {
  emit("close");
};

const getMappedYears = (selectedYears: number[]): string[] => {
  const yearMap = ["2010", "2011", "2012", "2013", "2014"];
  return yearMap.slice(selectedYears[0], selectedYears[1] + 1);
};

const drawChart = (data: DataProps): void => {
  // Clear previous chart
  d3.select(chartContainer.value).selectAll("*").remove();

  const selectedYearKeys = getMappedYears(props.years);

  const chartData = Object.entries(data)
    .filter(([key, value]) => selectedYearKeys.includes(key) && value != null)
    .map(([year, value]) => ({ year, value: value as number }));

  const width = 400;
  const height = 300;
  const marginTop = 30;
  const marginRight = 0;
  const marginBottom = 30;
  const marginLeft = 40;

  const x = d3
    .scaleBand()
    .domain(chartData.map((d) => d.year))
    .range([marginLeft, width - marginRight])
    .padding(0.1);

  const y = d3
    .scaleLinear()
    .domain([0, d3.max(chartData, (d) => d.value)!])
    .range([height - marginBottom, marginTop]);

  const svg = d3
    .select(chartContainer.value)
    .append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .attr("style", "max-width: 100%; height: auto;");

  svg
    .append("g")
    .attr("fill", "steelblue")
    .selectAll("rect")
    .data(chartData)
    .join("rect")
    .attr("x", (d) => x(d.year)!)
    .attr("y", (d) => y(d.value))
    .attr("height", (d) => y(0) - y(d.value))
    .attr("width", x.bandwidth());

  svg
    .append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(x).tickSizeOuter(0));

  svg
    .append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y).ticks(5))
    .call((g) => g.select(".domain").remove());
};

onMounted(() => {
  drawChart(props.data);
});

watch(
  [() => props.data, () => props.years],
  ([newData]) => {
    drawChart(newData);
  },
  { deep: true }
);
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
  padding: 0 1em 0 0.5em;
}
.header {
  display: flex;
  width: 100%;
  min-width: 400px;
  border-radius: 10px 10px 0 0;
  background: #d8e5ec;
  padding: 0.5em 0.75em 0.5em 1em;
  justify-content: space-between;
  font-family: sans-serif;
}
</style>
