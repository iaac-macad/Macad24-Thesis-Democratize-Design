<script setup>
import { ref, onBeforeMount, computed, watch } from "vue";
import { loadRhino } from "@/scripts/compute.js";

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue";
import GeometryView2 from "./components/GeometryView4.vue";
import SliderInput from "./components/SliderInput.vue";
import DropdownSelector from "./components/DropdownSelector.vue";  // Restored DropdownSelector (commented out below)
import ComputeButton from "./components/ComputeButton.vue";
import Switch from "./components/Switch02.vue";
import Upload3dm from "./components/Upload3dm.vue";
import { download } from "@/scripts/compute.js";

// Import Grasshopper definition file
// import def from './assets/osm_v16_3h.gh';
import def from './assets/osm_v17.gh';

// Define reactive references
const sliderName = ref("2D_UrbanField");
const sliderValue = ref(0.7);

const sliderName2 = ref("2E_RadField");
const sliderValue2 = ref(0.5);

const sliderName3 = ref("2F_SVField");
const sliderValue3 = ref(0.0);

// Metahopper toggle for initial aggregation not working. Making these true by default in the GH script.
// const switchName = ref("2A_Init Init Aggr");
// const switchValue = ref(false);

// const switchName2 = ref("2B_Run Initial Aggr");
// const switchValue2 = ref(false);

const switchName3 = ref("2C_Run LB");
const switchValue3 = ref(false);

const switchName9 = ref("3C_Record");
const switchValue9 = ref(false);

// Dropdown configurations (commented out for future use)
// let dropdownName = ref("Run LB");
// let dropdownIndex = ref(0);

// let dropdownName2 = ref("Run Initial Agg");
// let dropdownIndex2 = ref(0);

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

// Counter Output
const counterValue = ref(0); // New reactive reference to store counter output

// Function to update values based on parameter names
function updateValue(newValue, parameterName) {
  if (parameterName === sliderName.value) {
    sliderValue.value = newValue;
  } else if (parameterName === sliderName2.value) {
    sliderValue2.value = newValue;
  } else if (parameterName === sliderName3.value) {
    sliderValue3.value = newValue;
  // } else if (parameterName === switchName.value) {
  //   switchValue.value = newValue;
  // } else if (parameterName === switchName2.value) {
  //   switchValue2.value = newValue;
  } else if (parameterName === switchName3.value) {
    switchValue3.value = newValue;
  } else if (parameterName === switchName9.value) {
    switchValue9.value = newValue;
  } 
  // Uncomment these if dropdowns are used
  // else if (parameterName === dropdownName.value) {
  //   dropdownIndex.value = newValue;
  // } 
  // else if (parameterName === dropdownName2.value) {
  //   dropdownIndex2.value = newValue;
  // }
  console.log(`${parameterName} updated to: ${newValue}`);
}

// Function to run the counter
async function runCounter() {
  console.log('Attempting to run counter...');
  console.log('Switch Value 9:', switchValue9.value);
  console.log('Metadata[3]:', metadata.value[3]);

  // Ensure the switch is on and the metadata has a valid value
  if (switchValue9.value && metadata.value[3] && metadata.value[3].value > 0) {
    console.log('Counter conditions met. Starting counter...');
    let current = 0;
    const increment = 1;
    const delay = 1000;  // 1 second delay

    const intervalId = setInterval(async () => {
      if (current <= metadata.value[3].value) {
        try {
          // Make the POST request to the Hops component
          const response = await fetch('http://localhost:5000/', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify({
              current: current,
              increment: increment,
              target: metadata.value[3].value,
              run: switchValue9.value,
            }),
          });

          if (!response.ok) {
            throw new Error(`Server error: ${response.statusText}`);
          }

          // Parse the response
          const result = await response.json();
          current = result.next_value;

          // Update the counter value
          counterValue.value = current;

          console.log('Current Value:', current);

          // Check if the count has reached the target value
          if (current >= metadata.value[3].value) {
            console.log('Counting complete.');
            clearInterval(intervalId);  // Stop the interval
          }
        } catch (error) {
          console.error('Error with Hops computation:', error.message);
          clearInterval(intervalId);  // Stop the interval on error
        }
      } else {
        clearInterval(intervalId);  // Stop the interval if condition fails
      }
    }, delay);
  } else {
    console.log('Counter conditions not met. Check switch and metadata values.');
  }
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
  console.log('Compute triggered, starting counter...');
  compute.value = newVal;
  runCounter();  // Call runCounter within the compute process
}

// Computed ref for compute data
const computeData = computed(() => {
  let file = encodedFile.value || "empty";
  const dataObject = {
    ["encodedFile"]: file,
    [sliderName.value]: Number(sliderValue.value),
    [sliderName2.value]: Number(sliderValue2.value),
    [sliderName3.value]: Number(sliderValue3.value),
    // [switchName.value]: Boolean(switchValue.value),
    // [switchName2.value]: Boolean(switchValue2.value),
    [switchName3.value]: Boolean(switchValue3.value),
    [switchName9.value]: Boolean(switchValue9.value),
    // Uncomment these if dropdowns are used
    // [dropdownName.value]: Number(dropdownIndex.value),
    // [dropdownName2.value]: Number(dropdownIndex2.value),
    counterValue: counterValue.value, // Include counter value
  };
  console.log("Computed data:", dataObject); // Debugging
  return dataObject;
});

// Watch reactive values for debugging
watch(
  [encodedFile, sliderValue, sliderValue2, sliderValue3, switchValue3, switchValue9, counterValue],
  () => {
    console.log("Values updated:", {
      encodedFile: encodedFile.value,
      sliderValue: sliderValue.value,
      sliderValue2: sliderValue2.value,
      sliderValue3: sliderValue3.value,
      // switchValue: switchValue.value,
      // switchValue2: switchValue2.value,
      switchValue3: switchValue3.value,
      switchValue9: switchValue9.value,
      counterValue: counterValue.value,
      // Uncomment these if dropdowns are used
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

      <!-- Switch components with correct value and event binding -->
      <!-- <Switch 
        :label="switchName" 
        :initialValue="switchValue" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <Switch 
        :label="switchName2" 
        :initialValue="switchValue2" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      /> -->
      <Switch 
        :label="switchName3" 
        :initialValue="switchValue3" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <SliderInput 
        :title="sliderName" 
        :min="0.0" 
        :max="1.0" 
        :step="0.1" 
        :val="sliderValue" 
        @update="(newVal) => updateValue(newVal, sliderName.value)" 
      />
      <SliderInput 
        :title="sliderName2" 
        :min="0.0" 
        :max="1.0" 
        :step="0.1" 
        :val="sliderValue2" 
        @update="(newVal) => updateValue(newVal, sliderName2.value)" 
      />
      <SliderInput 
        :title="sliderName3" 
        :min="0.0" 
        :max="1.0" 
        :step="0.1" 
        :val="sliderValue3" 
        @update="(newVal) => updateValue(newVal, sliderName3.value)" 
      />



      <Switch 
        :label="switchName9" 
        :initialValue="switchValue9" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <!-- ComputeButton components, ensure they are uncommented -->
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

        <!-- Display Counter Output Here -->
        <p id="para">Counter Output:</p>
        <div id="para2">{{ counterValue }}</div> <!-- Counter output added here -->
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