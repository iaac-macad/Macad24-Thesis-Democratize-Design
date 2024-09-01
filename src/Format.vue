<script setup>
import { ref, computed, watch } from "vue";

// Import other Vue components
import GeometryView4 from "./components/GeometryView4.vue";
import SliderInput from "./components/SliderInput.vue";
import Switch from "./components/Switch02.vue";
import ComputeButton from "./components/ComputeButton.vue";

// Import and define Store
import { useComputeStore } from "@/stores/computeStore.js";
const computeStore = useComputeStore();

import def from './assets/osm_v17.gh';

// Part 02 Inputs
const firstSliderName = ref("2D_UrbanField"); // must match the Input name in your GH definition!
const firstSliderValue = ref(0.7); // default slider value
const secondSliderName = ref("2E_RadField"); // must match the Input name in your GH definition!
const secondSliderValue = ref(0.5); // default slider value
const thirdSliderName = ref("2F_SVField"); // must match the Input name in your GH definition!
const thirdSliderValue = ref(0.0); // default slider value

const switchName = ref("2A_Init Init Aggr");
const switchValue = ref(false);
const switchName2 = ref("2B_Run Initial Aggr");
const switchValue2 = ref(false);
const switchName3 = ref("2C_Run LB");
const switchValue3 = ref(false);

const switchName4 = ref("3A_Init Final Agg");
const switchValue4 = ref(false);
const switchName5 = ref("3B_Cores");
const switchValue5 = ref(false);
const switchName6 = ref("3C_Reset");
const switchValue6 = ref(false);
const switchName7 = ref("3D_Record");
const switchValue7 = ref(false);

const countername = ref("4_Floor");
const counterValue = ref(0); // New reactive reference to store counter output

let path = def; // path to the Grasshopper definition
let data = ref({});
let metadata = ref([]);
let compute = ref(false);

// Define isButtonDisabled as a computed property based on your requirements
const isButtonDisabled = computed(() => {
  // Example logic: disable button when metadata is empty
  return metadata.value.length === 0;
});

function updateValue(newValue, parameterName) {
  if (parameterName === firstSliderName.value) {
    firstSliderValue.value = newValue;
  } else if (parameterName === secondSliderName.value) {
    secondSliderValue.value = newValue;
  } else if (parameterName === thirdSliderName.value) {
    thirdSliderValue.value = newValue;
  } else if (parameterName === switchName.value) {
    switchValue.value = newValue;
  } else if (parameterName === switchName2.value) {
    switchValue2.value = newValue;
  } else if (parameterName === switchName3.value) {
    switchValue3.value = newValue;
  } else if (parameterName === switchName4.value) {
    switchValue4.value = newValue;
  } else if (parameterName === switchName5.value) {
    switchValue5.value = newValue;
  } else if (parameterName === switchName6.value) {
    switchValue6.value = newValue;
  } else if (parameterName === switchName7.value) {
    switchValue7.value = newValue;
  } else if (parameterName === countername.value) {
    countername.value = newValue;
  }
  console.log(`${parameterName} updated to: ${newValue}`);
}

// Function to run compute process
function runCompute() {
  console.log('Compute triggered, starting counter...');
  compute.value = true;
  runCounter();  // Call runCounter within the compute process
}

async function runCounter() {
  console.log('Attempting to run counter...');
  console.log('Switch Value 7:', switchValue7.value);
  console.log('Metadata[3]:', metadata.value[3]);

  if (switchValue7.value && metadata.value[3] && metadata.value[3].value > 0) {
    console.log('Counter conditions met. Starting counter...');
    let current = 0;
    const increment = 1;
    const delay = 1000;  // 1 second delay
    const targetValue = metadata.value[3].value;

    const intervalId = setInterval(async () => {
      if (current <= targetValue) {
        try {
          const response = await fetch('http://localhost:8081/', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify({
              current: current,
              increment: increment,
              target: targetValue,
              run: switchValue7.value,
            }),
          });

          if (!response.ok) {
            throw new Error(`Server error: ${response.statusText}`);
          }

          const result = await response.json();
          current = result.next_value;
          counterValue.value = current;

          console.log('Current Value:', current);

          if (current >= targetValue) {
            console.log('Counting complete.');
            clearInterval(intervalId);
          }
        } catch (error) {
          console.error('Error with Hops computation:', error.toString());
          clearInterval(intervalId);
        }
      } else {
        clearInterval(intervalId);
      }
    }, delay);
  } else {
    console.log('Counter conditions not met. Check switch and metadata values.');
  }
}

function receiveMetadata(newValue) {
  console.log(newValue);
  metadata.value = newValue;
}

const computeData = computed(() => {
  const dataObject = {  // Ensure dataObject is defined
    [firstSliderName.value]: Number(firstSliderValue.value),
    [secondSliderName.value]: Number(secondSliderValue.value),
    [thirdSliderName.value]: Number(thirdSliderValue.value),
    [switchName.value]: Boolean(switchValue.value),
    [switchName2.value]: Boolean(switchValue2.value),
    [switchName3.value]: Boolean(switchValue3.value),
    [switchName4.value]: Boolean(switchValue4.value),
    [switchName5.value]: Boolean(switchValue5.value),
    [switchName6.value]: Boolean(switchValue6.value),
    [switchName7.value]: Boolean(switchValue7.value),
    [countername.value]: Number(counterValue.value),
  };
  console.log("Computed data:", dataObject);
  return dataObject;
});

watch(
  [firstSliderValue, secondSliderValue, thirdSliderValue, switchValue, switchValue2, switchValue3, switchValue4, switchValue5, switchValue6, switchValue7, counterValue],
  () => {
    console.log("Values updated:", {
      firstSliderName: firstSliderValue.value,
      secondSliderName: secondSliderValue.value,
      thirdSliderName: thirdSliderValue.value,
      switchName: switchValue.value,
      switchName2: switchValue2.value,
      switchName3: switchValue3.value,
      switchName4: switchValue4.value,
      switchName5: switchValue5.value,
      switchName6: switchValue6.value,
      switchName7: switchValue7.value,
      countername: counterValue.value,
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
      <ComputeButton 
        title="Start" 
        :isDisabled="isButtonDisabled"
        @toggleCompute="runCompute" 
      />  
      <p id="intro">Choose location, enter program requirements, and steps below.</p>

      <!-- -------------------Part 02 ------------------------- -->

      <SliderInput :title="firstSliderName" @update="updateValue"/> 

      <SliderInput :title="secondSliderName" @update="updateValue"/> 

      <SliderInput :title = "thirdSliderName" @update="updateValue"/> 

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

      <Switch 
        :label="switchName3" 
        :initialValue="switchValue3" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <!-- -------------------Part 03 ------------------------- -->

      <Switch 
        :label="switchName4" 
        :initialValue="switchValue4" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <Switch 
        :label="switchName5" 
        :initialValue="switchValue5" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <Switch 
        :label="switchName6" 
        :initialValue="switchValue6" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <Switch 
        :label="switchName7" 
        :initialValue="switchValue7" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

    </div>

    <div id="viewerwindow" v-if="metadata && metadata.length > 0">
      <div id="Construction" class="data1">
        <p id="para">1-bedroom apartments:</p>
        <div id="para2" v-if="metadata[0]">{{ metadata[0].value }}</div>

        <p id="para">2-bedroom apartments:</p>
        <div id="para2" v-if="metadata[1]">{{ metadata[1].value }}</div>

        <p id="para">3-bedroom apartments:</p>
        <div id="para2" v-if="metadata[2]">{{ metadata[2].value }}</div>

        <p id="para">Test:</p>
        <div id="para2" v-if="metadata[3]">{{ metadata[3].value }}</div>

        <!-- Display Counter Output Here -->
        <p id="para">Counter Output:</p>
        <div id="para2">{{ counterValue }}</div> <!-- Counter output added here -->
      </div>

      <div id="viewer" class="geometry">
        <GeometryView4 
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
  height: 800px; /* Fixed height for the sidebar */
  padding: 20px;
  overflow-y: auto; /* Adds vertical scroll bar when content exceeds the height */
  overflow-x: hidden; /* Prevents horizontal scroll bar */
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