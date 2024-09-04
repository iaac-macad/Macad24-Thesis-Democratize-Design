<script setup>
import { ref, defineProps, defineEmits, watch } from "vue";

const props = defineProps(['title', 'max']);
const titlec = ref(props.title);
const emits = defineEmits(['update']);

var sliderValue = ref(1);

function sendValueUpdate() {
  emits("update", sliderValue.value, titlec.value);
}

// Watch for prop changes to keep sliderValue in sync if necessary
watch(() => props.value, (newVal) => {
  sliderValue.value = newVal;
});


</script>

<template>
  <div>
    <form class="definition-input">
      <div class="input-container">
        <input
          type="range"
          class="modern-range"
          min="0"
          :max="props.max"
          step="1"
          v-model="sliderValue"
          @mouseup="sendValueUpdate"  
        />
        <label class="input-title">{{ props.title }}: {{ sliderValue }}</label> 
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
</style>