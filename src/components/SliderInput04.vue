<script setup>
import { ref, defineProps, defineEmits } from "vue";

const props = defineProps(['title']);
const titlec = ref(props.title);
const emits = defineEmits(['update']);

var sliderValue = ref(0.0);

function sendValueUpdate() {
  emits("update", sliderValue.value, titlec.value);
}
</script>

<template>
  <div class="slider-container">
    <input
      type="range"
      class="modern-range"
      min="0"
      max="1"
      step="0.1"
      val="0.0"
      v-model="sliderValue"
      @mouseup="sendValueUpdate"
    />
    <label class="input-title">{{ title }}: {{ sliderValue }}</label>
  </div>
</template>

<style scoped>
.slider-container {
  width: 100%; /* Ensure the container is full width of its parent */
  margin: 10px 0;
}

.input-title {
  display: block;
  font-family: 'Roboto Mono', monospace; /* Match font to your app */
  font-size: 14px;
  white-space: nowrap; /* Prevent text wrapping */
  text-align: center; /* Center the label */
  margin: 5px 0; /* Add vertical space above and below */
}

.modern-range {
  -webkit-appearance: none;
  width: 100%; /* Ensure the slider is full width of its container */
  background: linear-gradient(90deg, #535253, hsl(0, 0%, 100%));
  height: 5px;
  border-radius: 15px;
  font-family: 'Courier New', Courier, monospace;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 15px;
  background-color: rgb(80, 80, 80);
  cursor: pointer;
}
</style>


<!-- <template>
  <div>
    <form class="definition-input">
      <div class="input-container">
        <input
          type="range"
          class="modern-range"
          min="0"
          max="1"
          step="0.1"
          val="0.5"
          v-model="sliderValue"
          @mouseup="sendValueUpdate"
        />
        <label class="input-title">{{ title }}: {{ sliderValue }}</label>
      </div>
    </form>
  </div>
</template>

<style scoped>
.input-container {
  position: relative;
}

.input-title {
  position: absolute;
  top: -40%;
  left: 50%;
  transform: translateX(-50%);
  font-family: Larabiefont, Arial, sans-serif;
  font-size: x-small;
  white-space: nowrap; /* Prevent text wrapping */
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  background: linear-gradient(90deg, hsl(0, 0%, 100%), #535253);
  height: 5px;
  border-radius: 15px;
  margin: 10px 0px;
  font-family: 'Courier New', Courier, monospace;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 15px;
  background-color: rgb(80, 80, 80);
  cursor: pointer;
}
</style> -->