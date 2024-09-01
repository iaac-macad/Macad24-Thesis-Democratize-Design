<template>
  <div id="app">
    <button
      class="compute-button"
      @click="toggleEmitting"
      :disabled="isDisabled"
    >{{ titlec }}</button>
  </div>
</template>


<script setup>
import { ref } from "vue";

const emits = defineEmits(["toggleCompute"]);

const props = defineProps(["title", "isDisabled"]);
const titlec = ref(props.title);
const isActive = ref(false);
let isEmitting = ref(false); // State to check if currently emitting
let intervalId = null;  // To store the interval ID

function toggleEmitting() {
  if (isEmitting.value) {
    stopEmitting(); // Stop emitting if already emitting
  } else {
    startEmitting(); // Start emitting if not already emitting
  }
}

function startEmitting() {
  isEmitting.value = true;
  isActive.value = true;
  emitContinuously();  // Start emitting immediately
  // Set interval to emit continuously
  intervalId = setInterval(() => {
    emitContinuously();
  }, 100);  // Adjust the interval time as needed (in milliseconds)
}

function stopEmitting() {
  isEmitting.value = false;
  isActive.value = false;
  clearInterval(intervalId);  // Stop the interval
  emits("toggleCompute", isActive.value);  // Emit once when stopping
}

function emitContinuously() {
  emits("toggleCompute", isActive.value);  // Emit the event
}
</script>


<style scoped>

button,   .customUpload {
    width: 100%;
    font-family: 'Roboto Mono', monospace;
    /* background-color: burlywood; */
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
  
  
  button:hover,  .customUpload:hover {
    /* background-color: green; */
    background-color: magenta;
    box-shadow: 3px 3px rgb(46, 46, 46);
  }


</style>