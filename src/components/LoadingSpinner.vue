<template>
  <div v-if="isVisible" class="spinner-overlay">
    <div class="spinner circles">
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
    </div>
  </div>
</template>

<script setup>
// Props to control visibility
const props = defineProps({
  isVisible: {
    type: Boolean,
    default: false,
  },
});
</script>

<style scoped>
.spinner-overlay {
  position: fixed; /* Cover the entire screen */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000; /* Higher z-index to ensure it's on top of other elements */
}

/* Spinner styles */
.spinner {
  width: 100px;
  height: 100px;
  margin: 30px auto;
  position: relative;
  animation: rotateit 1.3s linear infinite;
}

@keyframes rotateit {
  from {
    transform: rotate(360deg);
  }
  to {
    transform: rotate(0deg);
  }
}

/* Circle styles */
.spinner.circles div {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: black;
  position: absolute;
  top: 35px;
  left: 45px;
}

.spinner.circles div:nth-child(1) {
  transform: rotate(0deg) translate(0, -35px) scale(1.4);
}
.spinner.circles div:nth-child(2) {
  transform: rotate(45deg) translate(0, -35px) scale(1.2);
  opacity: 0.7;
}
.spinner.circles div:nth-child(3) {
  transform: rotate(90deg) translate(0, -35px) scale(1.1);
  opacity: 0.6;
}
.spinner.circles div:nth-child(4) {
  transform: rotate(135deg) translate(0, -35px) scale(0.9);
  opacity: 0.5;
}
.spinner.circles div:nth-child(5) {
  transform: rotate(180deg) translate(0, -35px) scale(0.7);
  opacity: 0.4;
}
.spinner.circles div:nth-child(6) {
  transform: rotate(225deg) translate(0, -35px) scale(0.5);
  opacity: 0.3;
}
.spinner.circles div:nth-child(7) {
  transform: rotate(270deg) translate(0, -35px) scale(0.3);
  opacity: 0.2;
}
.spinner.circles div:nth-child(8) {
  transform: rotate(315deg) translate(0, -35px) scale(0.1);
  opacity: 0.1;
}
</style>
