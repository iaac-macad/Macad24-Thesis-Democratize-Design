<script setup>
import { ref, watch } from "vue";

// Define props coming from parent component
const props = defineProps({
  label: String,
  initialValue: {
    type: Boolean,
    default: false,
  },
});

// Define events that will be emitted to parent
const emits = defineEmits(["update"]);

// Reactive value that holds the toggle state
const isOn = ref(props.initialValue);

// Watch for changes in initialValue prop to ensure reactivity
watch(
  () => props.initialValue,
  (newVal) => {
    isOn.value = newVal;
  },
  { immediate: true }
);

// Function called when the switch is toggled
function toggleSwitch() {
  isOn.value = !isOn.value;
  emits("update", isOn.value, props.label);  // Emit both value and label
  console.log(`Switch ${props.label} toggled to: ${isOn.value}`);
}
</script>

<template>
  <div class="switch-container">
    <span class="switch-label">{{ props.label }}</span>
    <div class="switch-content">
      <div 
        class="switch" 
        :class="{ 'switch-on': isOn, 'switch-off': !isOn }" 
        @click="toggleSwitch"
        role="switch"
        :aria-checked="isOn"
        tabindex="0"
        @keyup.space="toggleSwitch"
      >
        <div class="switch-toggle"></div>
      </div>
      <span class="switch-status">{{ isOn ? 'On' : 'Off' }}</span>
    </div>
  </div>
</template>


<style scoped>
.switch-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family: Roboto Mono, monospace;
}

.switch-label {
  font-size: 14px;
  font-weight: lighter;
  color: white;
}

.switch-content {
  display: flex;
  align-items: center;
  justify-content: flex-end;  /* Aligns the switch and status to the right */
  flex-grow: 1;  /* Ensures it takes up remaining space */
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
  background-color: white;
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