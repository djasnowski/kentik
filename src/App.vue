<template>
  <v-app>
    <RangeSlider @update-years="updateYears" />
    <Map :selectedYears="selectedYears" />
  </v-app>
</template>

<script setup>
import { ref } from "vue";
import Map from "./components/Map.vue";
import RangeSlider from "./components/RangeSlider.vue";

const selectedYears = ref([0, 4]);

const updateYears = (newYears) => {
  selectedYears.value = newYears;
};
</script>
