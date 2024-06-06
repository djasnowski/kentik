<template>
  <v-range-slider
    class="slider"
    v-model="selectedYears"
    :min="0"
    :max="4"
    :step="1"
    :ticks="ticks"
    show-ticks="always"
    @update:model-value="handleSliderChange"
  />
</template>

<script lang="ts" setup>
import { ref, watch, defineProps, defineEmits, withDefaults } from "vue";

interface Props {
  modelValue: number[];
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: () => [0, 4],
});

const emit = defineEmits<{
  (e: "update-years", value: number[]): void;
}>();

const selectedYears = ref<number[]>(props.modelValue);

const handleSliderChange = (value: number[]) => {
  selectedYears.value = value;
  emit("update-years", value);
};

watch(
  () => props.modelValue,
  (newValue) => {
    selectedYears.value = newValue;
  }
);

const ticks: Record<number, string> = {
  0: "2010",
  1: "2011",
  2: "2012",
  3: "2013",
  4: "2014",
};
</script>

<style>
.slider {
  position: absolute;
  top: 1em;
  background: white;
  box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
  border-radius: 15px;
  padding: 0.75em 2em;
  left: 1em;
  width: 400px;
  z-index: 100;
}

.v-slider-track__tick-label {
  margin-top: 1em !important;
}
</style>
