<script setup>
import { ref, onBeforeMount, computed, watch } from "vue";
import { loadRhino } from "@/scripts/compute.js";

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue";
import GeometryView2 from "./components/GeometryView4.vue";
import SliderInput from "./components/SliderInput.vue";
import SliderInput02 from "./components/SliderInput02.vue";
import DropdownSelector from "./components/DropdownSelector.vue";  // Restored DropdownSelector (commented out below)
// import ComputeButton from "./components/ComputeButton.vue";
import Switch from "./components/Switch02.vue";
import Upload3dm from "./components/Upload3dm.vue";
import { download } from "@/scripts/compute.js";

// Import Grasshopper definition file
// import def from './assets/osm_v16_3h.gh';
import def from './assets/osm_v20.gh';



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


const FloorSliderName = ref("4_Floor");
const FloorSliderValue = ref(0); 


const encodedFile = ref(null);
const isButtonDisabled = ref(false);

const path = def;
const metadata = ref([]);
const compute = ref(false);

// Initialize cache for metadata for each slider position
const sliderMetadataCache = ref({});  // New cache object



function updateValue(newValue, parameterName) {
  console.log(`updateValue called with newValue: ${newValue}, parameterName: ${parameterName}`); // Debugging output

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
  } else if (parameterName === FloorSliderName.value) {
    FloorSliderValue.value = newValue;  // Ensure FloorSliderValue is updated

    console.log(`FloorSliderValue updated to: ${FloorSliderValue.value}`);  // Debugging output

    // Additional logic for caching only for FloorSlider and when metadata[3] exists
    if (metadata.value[3]) {
      const sliderPosition = Number(FloorSliderValue.value); // Get the new slider position value
      const sliderKey = FloorSliderName.value; 

      // Initialize the cache for the slider if it doesn't exist
      if (!sliderMetadataCache.value[sliderKey]) {
        sliderMetadataCache.value[sliderKey] = {};
      }

      // Check if metadata for this slider position is already cached
      if (sliderMetadataCache.value[sliderKey][sliderPosition]) {
        console.log('Using cached metadata:', sliderMetadataCache.value[sliderKey][sliderPosition]);
      } else {
        // Fetch new metadata
        const newMetadata = fetchMetadata(sliderPosition); // Fetch metadata based on position
        sliderMetadataCache.value[sliderKey][sliderPosition] = newMetadata;
        console.log('Fetched and cached new metadata:', newMetadata);
      }

      // Call the total calculation function
      calculateTotal(sliderKey);
    }
  } else {
    console.warn(`Parameter name ${parameterName} does not match any known slider or switch.`); // Debugging output
  }

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
    [FloorSliderName.value]: Number(FloorSliderValue.value),
  };
  console.log("Computed data:", dataObject);
  return dataObject;
});

watch(
  [
    firstSliderValue,
    secondSliderValue,
    thirdSliderValue,
    switchValue,
    switchValue2,
    switchValue3,
    switchValue4,
    switchValue5,
    FloorSliderValue // Add FloorSliderValue to the watch list
  ],
  ([first, second, third, switch1, switch2, switch3, switch4, switch5, floor]) => {
    // Log values for other sliders and switches
    console.log("Values updated:", {
      firstSliderName: first,
      secondSliderName: second,
      thirdSliderName: third,
      switchName: switch1,
      switchName2: switch2,
      switchName3: switch3,
      switchName4: switch4,
      switchName5: switch5,
    });

    // Only watch the FloorSliderValue if metadata.value[3] exists
    if (metadata.value[3]) {
      console.log("FloorSliderValue updated:", floor);
      // Here you can add additional logic that should only run when metadata[3] exists
      // For example, updating something related to the FloorSlider
    }
  },
  { deep: true }
);


// Simulated function to fetch metadata (replace with actual implementation)
function fetchMetadata(position) {
  // Check if metadata is valid and contains at least three elements
  if (metadata.value.length >= 3) {
    return {
      position: position,
      value1: metadata.value[0].value,  // Ensure to access the actual data within metadata
      value2: metadata.value[1].value,  // Ensure to access the actual data within metadata
      value3: metadata.value[2].value,  // Ensure to access the actual data within metadata
    };
  } else {
    console.error('Not enough metadata available to fetch data.');
    return {
      position: position,
      value1: 0,
      value2: 0,
      value3: 0,
    };
  }
}

/// Function to calculate the total of all cached metadata values for each column
function calculateTotal(sliderKey) {
  // Initialize totals for each metadata column
  let totalValue1 = 0;
  let totalValue2 = 0;
  let totalValue3 = 0;

  // Ensure there is metadata to calculate
  if (sliderMetadataCache.value[sliderKey]) {
    // Iterate over each cached metadata entry for the slider
    Object.values(sliderMetadataCache.value[sliderKey]).forEach(metadata => {
      totalValue1 += metadata.value1;
      totalValue2 += metadata.value2;
      totalValue3 += metadata.value3;
    });
  } else {
    console.warn('No metadata found for slider key:', sliderKey);
  }

  // Output the totals
  console.log('Total of value1:', totalValue1);
  console.log('Total of value2:', totalValue2);
  console.log('Total of value3:', totalValue3);

  // You can add additional logic here if needed, such as updating UI elements
}

</script>


<template>
  <div id="appwindow">
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo.png" />
      <p id="intro">Generate housing project using graph theory and aggregation</p>

      <!-- <ComputeButton 
        title="Compute" 
        @click="() => runCompute(true)" 
        :isDisabled="isButtonDisabled" 
      /> -->
      
      <p id="intro">Choose location, enter program requirements, and steps below.</p>

      <!-- -------------------Part 02 ------------------------- -->

      <SliderInput :title="firstSliderName" @update="updateValue"/> 

      <SliderInput :title="secondSliderName" @update="updateValue"/> 

      <SliderInput :title = "thirdSliderName" @update="updateValue"/> 

      <!-- Switch components with correct value and event binding -->

      <p id="para">2A_Init Init Aggr:</p>
      <Switch 
        :label="switchName" 
        :initialValue="switchValue" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <p id="para">2B_Run Initial Aggr:</p>
      <Switch 
        :label="switchName2" 
        :initialValue="switchValue2" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <p id="para">2C_Run LB:</p>
      <Switch 
        :label="switchName3" 
        :initialValue="switchValue3" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />

      <!-- -------------------Part 03 ------------------------- -->
      <p id="para">3A_Init Final Agg:</p>
      <Switch 
        :label="switchName4" 
        :initialValue="switchValue4" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <p id="para">3B_Cores</p>
      <Switch 
        :label="switchName5" 
        :initialValue="switchValue5" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />


      <div id="para2" v-if="metadata[3]">{{ metadata[3].value }}
        <SliderInput02
          :value="FloorSliderValue"  
          :label="FloorSliderName.value"
          :max="metadata[3].value"
          :title="FloorSliderName"
          @update="updateValue" 
        />


      </div>


      <!-- ComputeButton components, ensure they are uncommented -->

      <!-- <ComputeButton 
        title="Download 3dm" 
        @click="download('CustomStick')" 
      >Download 3dm</ComputeButton> -->
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