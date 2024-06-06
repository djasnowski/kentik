<template>
  <div class="map-wrap">
    <div class="map" ref="mapContainer"></div>
    <TooltipChart
      v-if="tooltipData"
      :data="tooltipData"
      ref="tooltip"
      class="tooltip"
      @close="closeTooltip"
      :years="selectedYears"
    />
  </div>
</template>

<script setup lang="ts">
import { shallowRef, ref, onMounted, onUnmounted, nextTick, watch } from "vue";
import { Map, MapStyle, config } from "@maptiler/sdk";
import "@maptiler/sdk/dist/maptiler-sdk.css";
import TooltipChart from "./TooltipChart.vue";
import debounce from "lodash.debounce";
import type { Feature, FeatureCollection, Geometry } from "geojson";
import { DataProps } from "../types";

const props = defineProps<{ selectedYears: number[] }>();

const mapContainer = shallowRef<HTMLDivElement | null>(null);
const map = shallowRef<Map | null>(null);
const tooltipData = ref<DataProps | null>(null);
const tooltip = ref<InstanceType<typeof TooltipChart> | null>(null);
const originalData = ref<FeatureCollection<Geometry, DataProps> | null>(null);

const maptilerApiKey = import.meta.env.VITE_MAPTILER_API_KEY;

const initialState = {
  lng: -77.0369,
  lat: 38.9072,
  zoom: 12,
};

const addGeoJsonLayer = (
  map: Map,
  data: FeatureCollection<Geometry, DataProps>,
  selectedYears: number[]
) => {
  const years = ["2010", "2011", "2012", "2013", "2014"];
  const selectedYearsRange = years.slice(
    selectedYears[0],
    selectedYears[1] + 1
  );

  const updatedData = JSON.parse(JSON.stringify(data));

  // Sets any non-null value to a number
  // so our total is always a number :-)
  updatedData.features.forEach((feature: Feature<Geometry, DataProps>) => {
    feature.properties.total = selectedYearsRange.reduce((sum, year) => {
      const value = feature.properties[year] as number;
      return sum + (value || 0);
    }, 0);
  });

  if (map.getLayer("geojson-layer")) {
    map.removeLayer("geojson-layer");
    map.removeSource("geojson-source");
  }

  map.addSource("geojson-source", {
    type: "geojson",
    data: updatedData,
  });

  map.addLayer({
    id: "geojson-layer",
    source: "geojson-source",
    type: "fill-extrusion",
    paint: {
      "fill-extrusion-color": [
        "interpolate",
        ["linear"],
        ["get", "total"],
        0,
        "#fff5eb",
        1,
        "#fee6ce",
        2,
        "#fdd0a2",
        3,
        "#fdae6b",
        4,
        "#fd8d3c",
        5,
        "#f16913",
        6,
        "#d94801",
        7,
        "#8c2d04",
      ],
      "fill-extrusion-opacity": 0.8,
      "fill-extrusion-height": [
        "interpolate",
        ["linear"],
        ["get", "total"],
        0,
        0,
        7,
        500,
      ],
    },
  });
};

const closeTooltip = () => {
  tooltipData.value = null;
};

onMounted(() => {
  config.apiKey = maptilerApiKey;

  map.value = new Map({
    container: mapContainer.value as HTMLDivElement,
    style: MapStyle.BASIC,
    center: [initialState.lng, initialState.lat],
    zoom: initialState.zoom,
    pitch: 60,
    hash: false,
  });

  map.value.on("load", () => {
    fetch("/data.geojson")
      .then((response) => response.json())
      .then((data: FeatureCollection<Geometry, DataProps>) => {
        originalData.value = JSON.parse(JSON.stringify(data));
        addGeoJsonLayer(map.value!, data, props.selectedYears);

        map.value?.on("click", "geojson-layer", (e) => {
          const features = e.features;
          if (!features?.length) {
            return;
          }

          const feature = features[0];
          const { x, y } = e.point;

          // Sets data to tooltipData to pass to TooltipChart
          tooltipData.value = feature.properties as DataProps;

          nextTick(() => {
            if (tooltip.value) {
              // Set tooltip to x,y on click
              const tooltipElement = tooltip.value.$el;

              tooltipElement.style.left = `${x}px`;
              tooltipElement.style.top = `${y}px`;
            }
          });
        });

        map.value?.on("mouseenter", "geojson-layer", () => {
          if (map.value) {
            // Set cursor to pointer on-hover of polygon
            map.value.getCanvas().style.cursor = "pointer";
          }
        });

        map.value?.on("mouseleave", "geojson-layer", () => {
          if (map.value) {
            // Set cursor back to original on leave of polygon
            map.value.getCanvas().style.cursor = "";
          }
        });
      })
      .catch((error) => {
        console.error("Error loading the GeoJSON data:", error);
      });
  });
});

const debouncedHandleSliderChange = debounce((selectedYears: number[]) => {
  if (originalData.value) {
    addGeoJsonLayer(map.value!, originalData.value, selectedYears);
  }
}, 300);

watch(
  () => props.selectedYears,
  (newValue) => {
    debouncedHandleSliderChange(newValue);
  }
);

onUnmounted(() => {
  map.value?.remove();
});
</script>

<style scoped>
.map-wrap {
  position: relative;
  width: 100%;
  height: 100vh;
}

.map {
  width: 100%;
  height: 100%;
}

.tooltip {
  position: absolute;
  background: white;
  box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  z-index: 10;
  border-radius: 10px;
}
</style>
