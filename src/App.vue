<script setup>
import { ref, onBeforeMount, computed, watch } from "vue";
import { loadRhino } from "@/scripts/compute.js";

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue";
import GeometryView2 from "./components/GeometryView3.vue";
import SliderInput from "./components/SliderInput.vue";
import DropdownSelector from "./components/DropdownSelector.vue";
import ComputeButton from "./components/ComputeButton.vue";
import Switch from "./components/Switch02.vue";
import Upload3dm from "./components/Upload3dm.vue";
import { download } from "@/scripts/compute.js";

// Import Grasshopper definition file
import def from './assets/osm_v16_3h.gh';

// Define reactive references
const sliderName = ref("Floor");
const sliderValue = ref(0);

const switchName = ref("Run LB");
const switchValue = ref(false);

const switchName2 = ref("Run Initial Agg");
const switchValue2 = ref(false);

// let dropdownName = ref("Run LB")
// let dropdownIndex = ref(0)

// let dropdownName2 = ref("Run Initial Agg")
// let dropdownIndex2 = ref(0)

// const dropdownOptions = [
//   { label: "Not Run", value: 0 },
//   { label: "Run", value: 1 },
// ];

// const dropdownOptions2 = [
//   { label: "Not Run", value: 0 },
//   { label: "Run", value: 1 },
// ];

const encodedFile = ref(null);
const isButtonDisabled = ref(false);

const path = def;
const metadata = ref([]);
const compute = ref(false);

// Function to update values based on parameter names
function updateValue(newValue, parameterName) {
  if (parameterName === sliderName.value) {
    sliderValue.value = newValue;
  } else if (parameterName === switchName.value) {
    switchValue.value = newValue;
  } else if (parameterName === switchName2.value) {
    switchValue2.value = newValue;
  } 
  // else if (parameterName === dropdownName.value) {
  //   dropdownIndex.value = newValue;
  // } 
  // else if (parameterName === dropdownName2.value) {
  //   dropdownIndex2.value = newValue;
  // }
  console.log(`${parameterName} updated to: ${newValue}`);
}

// Function to update 3dm data
function update3dmData(newData) {
  encodedFile.value = newData;
  isButtonDisabled.value = false;
  compute.value = true;
}

// Function to receive metadata
function receiveMetadata(newValue) {
  console.log("Received metadata:", newValue);
  metadata.value = newValue;
}

// Function to run compute process
function runCompute(newVal) {
  compute.value = newVal;
}

// Computed ref for compute data
const computeData = computed(() => {
  let file = encodedFile.value || "empty";
  const dataObject = {
    ["encodedFile"]: file,
    [sliderName.value]: Number(sliderValue.value),
    [switchName.value]: Boolean(switchValue.value),
    [switchName2.value]: Boolean(switchValue2.value),
    // [dropdownName.value]: Number(dropdownIndex.value),
    // [dropdownName2.value]: Number(dropdownIndex2.value),
  };
  console.log("Computed data:", dataObject); // Debugging
  return dataObject;
});

// Watch reactive values for debugging
watch(
  [encodedFile, sliderValue, switchValue, switchValue2],
  () => {
    console.log("Values updated:", {
      encodedFile: encodedFile.value,
      sliderValue: sliderValue.value,
      switchValue: switchValue.value,
      switchValue2: switchValue2.value,
      // dropdownIndex: dropdownIndex.value,
      // dropdownIndex2: dropdownIndex2.value,
    });
  },
  { deep: true }
);
</script>

<template>
  <div id="appwindow">
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo.png" />
      <p id="intro">Generate housing project using graph theory and aggregation</p>
      <p id="intro">Choose location, enter program requirements, and steps below.</p>

      <!-- <Upload3dm @encoded3dm="update3dmData" /> -->

      <!-- <DropdownSelector :title="dropdownName" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue"/>
      <DropdownSelector :title="dropdownName2" :options="dropdownOptions2" :val="dropdownIndex2" @update="updateValue"/> -->

      <!-- Switch components with correct value and event binding -->
      <Switch 
        :label="switchName" 
        :initialValue="switchValue" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <Switch 
        :label="switchName2" 
        :initialValue="switchValue2" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <!-- <Switch :label="switchName" :initialValue="switchValue" @update="(newVal) => updateValue(newVal, switchName.value)" />
      <Switch :label="switchName2" :initialValue="switchValue2" @update="(newVal) => updateValue(newVal, switchName2.value)" /> -->
        
      <SliderInput 
        :title="sliderName" 
        :min="0" 
        :max="20" 
        :step="1" 
        :val="sliderValue" 
        @update="(newVal) => updateValue(newVal, sliderName.value)" 
      />
      <!-- <SliderInput :title="sliderName2" :min="3.0" :max="10.0" :step="0.1" :val="4.2" @update="updateValue"></SliderInput> -->

      <!-- ComputeButton components -->
      <ComputeButton 
        title="Compute" 
        @click="() => runCompute(true)" 
        :isDisabled="isButtonDisabled" 
      />
      <ComputeButton 
        title="Download 3dm" 
        @click="download('CustomStick')" 
      >Download 3dm</ComputeButton>
    </div>

    <div id="viewerwindow">
      <div id="Construction" class="data1">
        <p id="para">1-bedroom apartments:</p>
        <div id="para2" v-if="metadata[0]">{{ metadata[0].value }}</div>

        <p id="para">2-bedroom apartments:</p>
        <div id="para2" v-if="metadata[1]">{{ metadata[1].value }}</div>

        <p id="para">3-bedroom apartments:</p>
        <div id="para2" v-if="metadata[2]">{{ metadata[2].value }}</div>

        <p id="para">Test:</p>
        <div id="para2" v-if="metadata[3]">{{ metadata[3].value }}</div>
      </div>

      <div id="viewer" class="geometry">
        <GeometryView2 
          :data="computeData" 
          :path="path" 
          :runCompute="compute" 
          @updateMetadata="receiveMetadata" 
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
#appwindow {
  display: flex;
  width: 100vw;
  height: 100vh;
  padding: 20px;
  background-color: white;
}

.container {
  background-color: black;
  border-style: solid;
  border-color: white;
  border-radius: 25px;
  border-width: 2px;
}

.data1 {
  background-color: rgba(0, 0, 0, 0.432);
  width: 30%;
  font-family: Roboto Mono, monospace;
  font-size: 16px;
  color: white;
  text-align: left;
  border-style: solid;
  border-color: white;
  border-radius: 25px;
  border-width: 1px;
  padding: 0px 20px;
}

#sidebar {
  width: 310px;
  height: 800px;
  padding: 20px;
}

#intro {
  font-family: Roboto Mono, monospace;
  font-size: 14px;
}

#para {
  font-family: Roboto Mono, monospace;
  font-size: 16px;
  text-align: left;
  margin-bottom: 0px;
}

#para2 {
  font-family: Roboto Mono, monospace;
  font-style: italic;
  font-size: 14px;
  text-align: left;
  margin-bottom: 25px;
}

#switch-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
}

#switch-label {
  margin-right: auto; /* Pushes the label to the left */
  font-size: 14px;
  font-weight: lighter;
  color: #1b314f;
}

#switch {
  margin-left: auto; /* Pushes the switch to the far right */
}

#viewerwindow {
  display: flex;
  flex-direction: column;
  width: calc(100% - 580px);
  height: 100vh;
  gap: 10px;
  padding: 0px 20px;
  border-color: white;
  background-color: white;
}

#viewer {
  width: 100%;
  height: 100vh;
  align-items: center;
  border-radius: 25px;
}

#Construction {
  position: absolute;
  text-align: left;
  margin: 0px 0px;
}

.mainlogo {
  width: 100%;
  height: 10%;
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  height: 10px;
  border-radius: 5px;
  background: linear-gradient(to right, #000 0%, #8a8a8a 50%, #6d6d6d 50%, #8d8d8d 100%);
  outline: none;
  margin: 8px 0px;
  background-image: -webkit-repeating-linear-gradient(left, #8a8a8a, #8a8a8a 1px, transparent 1px, transparent 4px);
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: grey;
  cursor: grab;
}
</style>