<template>
  <div id="app">
    <button
      class="compute-button"
      @click="toggleCompute"
      :disabled="isDisabled"
    >
      {{ titlec }} {{ isActive ? "Stop" : "Start" }}
    </button>
  </div>
</template>

<script setup>
import { ref } from "vue";

const emits = defineEmits(["toggleCompute"]);
const props = defineProps(["title", "isDisabled"]);

const titlec = ref(props.title);
const isActive = ref(false);  // State to track if it's computing

function toggleCompute() {
  isActive.value = !isActive.value;  // Toggle between true and false
  emits("toggleCompute", isActive.value);  // Emit the current state
}
</script>

<style scoped>
button,
.customUpload {
  width: 100%;
  font-family: 'Roboto Mono', monospace;
  background-color: grey;
  border: #1897ff;
  color: white;
  font-weight: bold;
  padding: 10px 15px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 10px;
  transition: all 0.1s ease-in-out;
}

button:hover,
.customUpload:hover {
  background-color: magenta;
  box-shadow: 3px 3px rgb(46, 46, 46);
}
</style>
