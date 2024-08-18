<script setup>
import { ref, watch } from "vue";

// Define props coming from parent component
const props = defineProps({
  label: String,
  initialValue: Boolean,
});

// Define events that will be emitted to parent
const emits = defineEmits(["update"]);

// Reactive value that holds the toggle state
// const isOn = ref(props.initialValue);
const isOn = ref(false);


// Watch for changes in initialValue prop to ensure reactivity
watch(() => props.initialValue, (newVal) => {
  isOn.value = newVal;
});

// Function called when the switch is toggled
function toggleSwitch() {
  isOn.value = !isOn.value;
  emits("update", isOn.value);
  console.log(isOn.value)
}
</script>

<template>
  <div class="switch-container">
    <label class="switch-label">{{ label }}</label>
    <div class="switch" v-bind:class="{ 'switch-on': isOn, 'switch-off': !isOn }" v-on:click="toggleSwitch">
      <div class="switch-toggle"></div>
    </div>
    <span class="switch-status">{{ isOn ? 'On' : 'Off' }}</span>
  </div>
</template>

<style scoped>
.switch-container {
  display: flex;
  align-items: center;
  font-family: Roboto Mono, monospace;
}

.switch-label {
  margin-right: 10px;
  font-size: 14px;
  font-weight: lighter;
  color: #1b314f;
}

.switch {
  width: 50px;
  height: 25px;
  border-radius: 15px;
  background-color: grey;
  position: relative;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.switch-on {
  background-color: #4caf50;
}

.switch-off {
  background-color: #ccc;
}

.switch-toggle {
  width: 20px;
  height: 20px;
  background-color: grey;
  border-radius: 50%;
  position: absolute;
  top: 2.5px;
  left: 2.5px;
  transition: left 0.3s ease;
}

.switch-on .switch-toggle {
  left: 27.5px;
}

.switch-status {
  margin-left: 10px;
  font-size: 14px;
  font-weight: lighter;
  color: white;
}
</style>