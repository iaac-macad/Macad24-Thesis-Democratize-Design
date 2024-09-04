<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps(['title']);
const isOpen = ref(false);
const emits = defineEmits();

function togglePanel() {
  isOpen.value = !isOpen.value;
}

function handleClick() {
  togglePanel();
}
</script>

<template>
  <div class="collapsible-panel">
    <div class="panel-header" @click="handleClick">
      <span>{{ title }}</span>
      <span class="toggle-indicator">{{ isOpen ? '−' : '+' }}</span>
    </div>
    <div v-if="isOpen" class="panel-content">
      <slot></slot>
    </div>
  </div>
</template>

<style scoped>
.collapsible-panel {
  background-color: black;
  border-style: solid;
  border-color: white;
  border-radius: 25px;
  border-width: 1px; /* Changed from 2px to 1px to match .data1 */
  margin-bottom: 10px;
  padding: 10px;
}

.panel-header {
  background-color: black;
  color: white;
  padding: 5px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family: Roboto Mono, monospace;
  font-size: 14px;
  font-weight: lighter;
}

.panel-content {
  padding: 10px;
  background-color: black;
}

.toggle-indicator {
  font-size: 18px;
}

</style>
