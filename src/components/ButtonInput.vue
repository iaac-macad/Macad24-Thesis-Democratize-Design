<template>
  <div id="app">
    <button 
    @mousedown="onButtonClick(true)"
    @mouseup="onButtonClick(false)"
    
    
    >{{ titlec }}</button>
  </div>
</template>

<script setup>
import { ref } from "vue";

const emits = defineEmits(["toggleCompute"]);

const props = defineProps(["title"]);
const titlec = ref(props.title);
const isActive = ref(false);

async function onButtonClick(state) {
  isActive.value = state
  emits("toggleCompute", isActive.value );
}
</script>
