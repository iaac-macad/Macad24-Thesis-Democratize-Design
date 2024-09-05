<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps(['label']);  // Accepts a label prop for the input
const emits = defineEmits(['update']);

console.log('Received label prop in TextInput:', props.label); // Add this log

// Reactive variable to store the text input value
const textValue = ref('');

// Function to emit the updated value to the parent component
function sendTextUpdate() {
  if (props.label === undefined) {
    console.warn('Label prop is undefined. Check parent binding.');
  } else {
    emits('update', textValue.value, props.label);  // Use props.label directly
  }
}
</script>


<template>
  <div class="text-input-container">
    <label class="input-title">{{ label }}:</label>
    <input
      type="text"
      class="modern-text-input"
      v-model="textValue"
      @input="sendTextUpdate"
      placeholder="Enter text here"
    />
  </div>
</template>


<style scoped>
.text-input-container {
  width: 100%; /* Full width of its parent */
  margin: 10px 0;
  display: flex;
  flex-direction: column; /* Stack label and input vertically */
  align-items: center; /* Center align the items */
}

.input-title {
  font-family: 'Roboto Mono', monospace; /* Match font to your app */
  font-size: 14px;
  margin: 5px 0; /* Add vertical space above and below */
  text-align: center; /* Center the label */
}

.modern-text-input {
  width: 100%; /* Full width for the input */
  padding: 5px; /* Add padding for better UX */
  font-family: 'Roboto Mono', monospace; /* Match font to your app */
  font-size: 14px; /* Consistent font size */
  border: 2px solid #ddd; /* Light border for input */
  border-radius: 5px; /* Rounded corners */
  outline: none; /* Remove default outline */
  transition: border-color 0.3s ease; /* Smooth transition on focus */
}

.modern-text-input:focus {
  border-color: #535253; /* Darker border color on focus */
}
</style>
