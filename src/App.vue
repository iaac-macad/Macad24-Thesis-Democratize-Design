<script setup>
import { ref, onMounted, onUnmounted, computed, watch } from "vue";
import { loadRhino } from "@/scripts/compute.js";

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue";
import GeometryView2 from "./components/GeometryView5.vue";
import SliderInput from "./components/SliderInput.vue";
import SliderInput02 from "./components/SliderInput02.vue";
import SliderInput03 from "./components/SliderInput03.vue";
import SliderInput04 from "./components/SliderInput04.vue";
import SliderInput05 from "./components/SliderInput05.vue";
import SliderInput06 from "./components/SliderInput06.vue";
import SliderInput07 from "./components/SliderInput07.vue";
import DropdownSelector from "./components/DropdownSelector.vue";  // Restored DropdownSelector (commented out below)
// import ComputeButton from "./components/ComputeButton.vue";
import Switch from "./components/Switch03.vue";
import Upload3dm from "./components/Upload3dm.vue";
import { download } from "@/scripts/compute.js";
import CollapsiblePanel from "./components/CollapsiblePanel.vue";
import TextInput from './components/TextInput.vue';

import BarChart from './components/BarChart.vue'; // Import the BarChart component
import PieChart from './components/PieChart.vue';
import PieChart2 from './components/PieChart2.vue';

import LoadingSpinner from "./components/LoadingSpinner.vue"; // Import the spinner

// Import Grasshopper definition file
// import def from './assets/osm_v16_3h.gh';
import def from './assets/model_v3.gh';

// Part 01: Define reactive variables for text input names and values
const textInputName1 = ref('Building_Number');
const textInputValue1 = ref('400');

const textInputName2 = ref('Street_Name');
const textInputValue2 = ref('CATALINA ST');

const textInputName3 = ref('City');
const textInputValue3 = ref('LOS ANGELES');

const textInputName4 = ref('State');
const textInputValue4 = ref('CA');

const textInputName5 = ref('ZIP_Code');
const textInputValue5 = ref('90020-3758');

// Method to handle updates from TextInput components
function handleTextUpdate(value, label) {
  console.log(`Updated: ${label} - ${value}`);
  if (!label) {
    console.warn('Received an undefined label:', value);
    return; // Exit early if the label is undefined
  }
  
  console.log(`Updated Value from ${label}:`, value);

  if (label === textInputName1.value) textInputValue1.value = value;
  else if (label === textInputName2.value) textInputValue2.value = value;
  else if (label === textInputName3.value) textInputValue3.value = value;
  else if (label === textInputName4.value) textInputValue4.value = value;
  else if (label === textInputName5.value) textInputValue5.value = value;
}

console.log('Text Input Names:', textInputName1.value, textInputName2.value, textInputName3.value, textInputName4.value, textInputName5.value);


// Part 02 Inputs
const firstSliderName = ref("Urban_Context"); // must match the Input name in your GH definition!
const firstSliderValue = ref(0.7); // default slider value
const secondSliderName = ref("Beneficial_Radiation"); // must match the Input name in your GH definition!
const secondSliderValue = ref(0.5); // default slider value
const thirdSliderName = ref("Sky_View_Factor"); // must match the Input name in your GH definition!
const thirdSliderValue = ref(0.0); // default slider value

const switchName = ref("Run_Massing_Study");
const switchValue = ref(false);
// const switchName2 = ref("2B_Run Initial Aggr");
// const switchValue2 = ref(false);
const switchName3 = ref("Run_Environmental_Analysis");
const switchValue3 = ref(false);

const switchName6 = ref("Preview_Massing");
const switchValue6 = ref(false);

// Part 03 Inputs
const forthSliderName = ref("1_bedroom");
const forthSliderValue = ref(50); 
const fifthSliderName = ref("2_bedroom"); 
const fifthSliderValue = ref(50); 
const sixthSliderName = ref("3_bedroom");
const sixthSliderValue = ref(50); 

const switchName4 = ref("Run_Building_Generation");
const switchValue4 = ref(false);
const switchName5 = ref("Generate_Vertical_Circulation");
const switchValue5 = ref(false);

const FloorSliderName = ref("Floor_Number");
const FloorSliderValue = ref(0); 


const encodedFile = ref(null);
const isButtonDisabled = ref(false);

const path = def;
const metadata = ref([]);
const compute = ref(false);

const seventhSliderName = ref("Target_Footprint_Area"); 
const seventhSliderValue = ref(1700); 
const eighthSliderName = ref("Target_GIA");
const eigthSliderValue = ref(5400); 

// Initialize cache for metadata for each slider position
const sliderMetadataCache = ref({});  // New cache object

// Reactive references for total values
const totalValue1 = ref(0);
const totalValue2 = ref(0);
const totalValue3 = ref(0);
const totalValue4 = ref(0);
const totalValue5 = ref(0);
const totalValue6 = ref(0);



function updateValue(newValue, parameterName) {
  console.log(`updateValue called with newValue: ${newValue}, parameterName: ${parameterName}`); // Debugging output

  if (parameterName === firstSliderName.value) {
    firstSliderValue.value = newValue;
  } else if (parameterName === secondSliderName.value) {
    secondSliderValue.value = newValue;
  } else if (parameterName === thirdSliderName.value) {
    thirdSliderValue.value = newValue;
  } else if (parameterName === forthSliderName.value) {
    forthSliderValue.value = newValue;
  } else if (parameterName === fifthSliderName.value) {
    fifthSliderValue.value = newValue;
  } else if (parameterName === sixthSliderName.value) {
    sixthSliderValue.value = newValue;
  } else if (parameterName === seventhSliderName.value) {
    seventhSliderValue.value = newValue;
  } else if (parameterName === eighthSliderName.value) {
    eigthSliderValue.value = newValue;


  } else if (parameterName === switchName.value) {
    switchValue.value = newValue;
  // } else if (parameterName === switchName2.value) {
  //   switchValue2.value = newValue;
  } else if (parameterName === switchName3.value) {
    switchValue3.value = newValue;
  } else if (parameterName === switchName4.value) {
    switchValue4.value = newValue;
  } else if (parameterName === switchName5.value) {
    switchValue5.value = newValue;
  } else if (parameterName === switchName6.value) {
    switchValue6.value = newValue;
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
    [forthSliderName.value]: Number(forthSliderValue.value),
    [fifthSliderName.value]: Number(fifthSliderValue.value),
    [sixthSliderName.value]: Number(sixthSliderValue.value),
    [seventhSliderName.value]: Number(seventhSliderValue.value),
    [eighthSliderName.value]: Number(eigthSliderValue.value),



    [switchName.value]: Boolean(switchValue.value),
    // [switchName2.value]: Boolean(switchValue2.value),
    [switchName3.value]: Boolean(switchValue3.value),
    [switchName4.value]: Boolean(switchValue4.value),
    [switchName5.value]: Boolean(switchValue5.value),
    [switchName6.value]: Boolean(switchValue6.value),
    [FloorSliderName.value]: Number(FloorSliderValue.value),
    [textInputName1.value]: textInputValue1.value,
    [textInputName2.value]: textInputValue2.value,
    [textInputName3.value]: textInputValue3.value,
    [textInputName4.value]: textInputValue4.value,
    [textInputName5.value]: textInputValue5.value,
  };
  console.log("Computed data:", dataObject);
  return dataObject;
});

// Watch for changes in all relevant variables
watch(
  [
    firstSliderValue,
    secondSliderValue,
    thirdSliderValue,
    forthSliderValue,
    fifthSliderValue,
    sixthSliderValue,
    seventhSliderValue,
    eigthSliderValue,


    switchValue,
    // switchValue2,
    switchValue3,
    switchValue4,
    switchValue5,
    switchValue6,
    FloorSliderValue,
    textInputValue1,
    textInputValue2,
    textInputValue3,
    textInputValue4,
    textInputValue5
  ],
  ([
    first,
    second,
    third,
    forth,
    fifth,
    sixth,
    seventh,
    eigth,
    switch1,
    // switch2,
    switch3,
    switch4,
    switch5,
    switch6,
    floor,
    input1,
    input2,
    input3,
    input4,
    input5
  ]) => {
    console.log("Values updated:", {
      firstSliderName: first,
      secondSliderName: second,
      thirdSliderName: third,
      forthSliderName: forth,
      fifthSliderName: fifth,
      sixthSliderName: sixth,
      seventhSliderName: seventh,
      eigthSliderName: eigth,

      switchName: switch1,
      // switchName2: switch2,
      switchName3: switch3,
      switchName4: switch4,
      switchName5: switch5,
      switchName6: switch6,
      textInputName1: input1,
      textInputName2: input2,
      textInputName3: input3,
      textInputName4: input4,
      textInputName5: input5,
    });
    
    if (metadata.value[3]) {
      console.log("FloorSliderValue updated:", floor);
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
      value1: parseFloat(metadata.value[0].value) || 0,  // Ensure the value is a number
      value2: parseFloat(metadata.value[1].value) || 0,  // Ensure the value is a number
      value3: parseFloat(metadata.value[2].value) || 0,  // Ensure the value is a number
      value4: parseFloat(metadata.value[9].value) || 0,  // Ensure the value is a number
      value5: parseFloat(metadata.value[10].value) || 0,  // Ensure the value is a number
      value6: parseFloat(metadata.value[17].value) || 0,  // Ensure the value is a number
    };
  } else {
    console.error('Not enough metadata available to fetch data.');
    return {
      position: position,
      value1: 0,
      value2: 0,
      value3: 0,
      value4: 0,
      value5: 0,
      value6: 0,
    };
  }
}





// Function to calculate the total of all cached metadata values for each column
function calculateTotal(sliderKey) {
  console.log('Calculating totals for slider key:', sliderKey);

  // Reset totals
  totalValue1.value = 0;
  totalValue2.value = 0;
  totalValue3.value = 0;
  totalValue4.value = 0;
  totalValue5.value = 0;
  totalValue6.value = 0;

  // Ensure there is metadata to calculate
  if (sliderMetadataCache.value[sliderKey]) {
    // Iterate over each cached metadata entry for the slider
    Object.values(sliderMetadataCache.value[sliderKey]).forEach(metadata => {
      totalValue1.value += parseFloat(metadata.value1) || 0;  // Ensure numeric value
      totalValue2.value += parseFloat(metadata.value2) || 0;  // Ensure numeric value
      totalValue3.value += parseFloat(metadata.value3) || 0;  // Ensure numeric value
      totalValue4.value += parseFloat(metadata.value4) || 0;  // Ensure numeric value
      totalValue5.value += parseFloat(metadata.value5) || 0;  // Ensure numeric value
      totalValue6.value += parseFloat(metadata.value6) || 0;  // Ensure numeric value
    });
  } else {
    console.warn('No metadata found for slider key:', sliderKey);
  }

  // Output the totals
  console.log('Total of value1:', totalValue1.value);
  console.log('Total of value2:', totalValue2.value);
  console.log('Total of value3:', totalValue3.value);
  console.log('Total of value4:', totalValue4.value);
  console.log('Total of value5:', totalValue5.value);
  console.log('Total of value6:', totalValue6.value);
}



const pieChartData = computed(() => {
  if (metadata.value && metadata.value.length >= 3){
  // Check if metadata.value exists and its length is at least 3
  return [
    { value: parseFloat(totalValue1.value) || 0, name: 'Total 1 Beds' },
    { value: parseFloat(totalValue2.value) || 0, name: 'Total 2 Beds' },
    { value: parseFloat(totalValue3.value) || 0, name: 'Total 3 Beds' },
  ];
  }
});


console.log('Total of value1:', totalValue1.value);
console.log('Total of value2:', totalValue2.value);
console.log('Total of value3:', totalValue3.value);
console.log('Total of value4:', totalValue4.value);
console.log('Total of value5:', totalValue5.value);
console.log('Total of value6:', totalValue6.value);

const Total_Area_ = totalValue5.value


// Log the values to check reactivity
console.log('Computed pieChartData:', pieChartData.value);


// Calculate percentage  areas
const Living_space = (totalValue4.value / totalValue5.value) * 100;

const Common_Area = (totalValue4.value / totalValue6.value) * 100;

const non_net = 100 - (Common_Area + Living_space)



console.log('Percentage of living space:', Living_space.toFixed(2) + '%');
console.log('Percentage of Common_Area:',Common_Area.toFixed(2) + '%');
console.log('Percentage of Common_Area:',non_net.toFixed(2) + '%');

const pieChart2Data = computed(() => {
  if (metadata.value && metadata.value.length >= 3){
  // Calculate percentages for usable and non-usable areas
  const Living_space = (totalValue4.value / totalValue5.value) * 100;

  const Common_Area = (totalValue4.value / totalValue6.value) * 100;

  const non_net = 100 - (Common_Area + Living_space)

  // Return data formatted for the pie chart
  return [
    { value: parseFloat(Living_space.toFixed(2)) || 0, name: 'Living Space' },
    { value: parseFloat(Common_Area.toFixed(2)) || 0, name: 'Common Area' },
    { value: parseFloat(non_net.toFixed(2)) || 0, name: 'Non-Net Area' },
  ];
  }
});


// Reactive variable to control spinner visibility
const isComputing = ref(false);

// Function to handle custom logs and toggle spinner
function handleCustomLog(message) {
  console.log(message); // Call the original console.log

  if (message === "Running compute...") {
    isComputing.value = true; // Show spinner
  } else if (message === "Compute done.") {
    isComputing.value = false; // Hide spinner
  }
}

// Event listeners to toggle spinner visibility
function handleComputeStart() {
  isComputing.value = true; // Show spinner
}

function handleComputeEnd() {
  isComputing.value = false; // Hide spinner
}

// Setup event listeners in Vue lifecycle hooks
onMounted(() => {
  window.addEventListener("compute-start", handleComputeStart);
  window.addEventListener("compute-end", handleComputeEnd);
});

onUnmounted(() => {
  window.removeEventListener("compute-start", handleComputeStart);
  window.removeEventListener("compute-end", handleComputeEnd);
});


// Ensure metadata exists and contains valid numeric values
const validMetadata = computed(() => {
  if (metadata.value && metadata.value.length >= 3) {
    return [
      metadata.value[0]?.value ?? 0,
      metadata.value[1]?.value ?? 0,
      metadata.value[2]?.value ?? 0
    ];
  }
  return null; // Return null if metadata is invalid or missing
});
// // Example usage for demonstration
// handleCustomLog("Running compute..."); // This would be in your actual compute start logic
// setTimeout(() => handleCustomLog("Compute done."), 3000); // This simulates compute end after 3 seconds
// Computed property to check for error messages in metadata[12], metadata[13], and metadata[14]
const errorMessage = computed(() => {
  const errors = [12, 13, 14, 15, 16]
    .map(index => metadata.value[index]?.value) // Retrieve values if they exist
    .filter(Boolean); // Filter out falsy values (e.g., undefined, null, empty string)

  return errors.length > 0 ? errors.join('<br>') : 'Everything looks okay!';
});

</script>


<template>
  <div id="appwindow">
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo4.png" />
      <p id="intro">This feasability tool enables you to generate modular low-income housing designs utilizing graphs and combinatorial modeling. 
        Architectural expertise, environmental simulation, and typical space requirements are embedded in the algorithm.</p>

      <!-- <ComputeButton 
        title="Compute" 
        @click="() => runCompute(true)" 
        :isDisabled="isButtonDisabled" 
      /> -->
      
      <p id="intro">Follow the steps below.</p>
      <CollapsiblePanel title="Select Location">

        <TextInput :label="textInputName1" @update="handleTextUpdate" />
        <TextInput :label="textInputName2" @update="handleTextUpdate" />
        <TextInput :label="textInputName3" @update="handleTextUpdate" />
        <TextInput :label="textInputName4" @update="handleTextUpdate" />
        <TextInput :label="textInputName5" @update="handleTextUpdate" />

      </CollapsiblePanel>
      <!-- -------------------Part 02 ------------------------- -->

      <!-- Switch components with correct value and event binding -->

      <CollapsiblePanel title="Create Massing Volume">

    <div class="tooltip-container">
      <SliderInput06 
        :title="seventhSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">Please enter the target for the building footprint in sqaure meters </span>
    </div>

    <!-- Wrapper for each Switch with Tooltip -->
    <div class="tooltip-container">
      <Switch 
        :label="switchName" 
        :initialValue="switchValue" 
        @update="(newVal, label) => updateValue(newVal, label)"
      />
      <span class="tooltip-text">Turn this on when you are ready to run the initial aggregation</span>
    </div>

    <!-- <p id="para"></p>

    <div class="tooltip-container">
      <Switch 
        :label="switchName2" 
        :initialValue="switchValue2" 
        @update="(newVal, label) => updateValue(newVal, label)"
      />
      <span class="tooltip-text">Turn this on when you are ready to generate the mass</span>
    </div> -->

    <p id="para"></p>

    <!-- Tooltip for Sliders (unchanged) -->
    <div class="tooltip-container">
      <SliderInput 
        :title="firstSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is the urban context?</span>
    </div>

    <div class="tooltip-container">
      <SliderInput03 
        :title="secondSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is passive solar radiation design?</span>
    </div>

    <div class="tooltip-container">
      <SliderInput04 
        :title="thirdSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is view to sky?</span>
    </div>


    <div class="tooltip-container">
      <Switch 
        :label="switchName3" 
        :initialValue="switchValue3" 
        @update="(newVal, label) => updateValue(newVal, label)"
      />
      <span class="tooltip-text">Switch true if you want to consider solar analysis and sky view in the massing</span>
    </div>
  </CollapsiblePanel>

   <!-- Part 02: Visualize Massing Volume -->
   <CollapsiblePanel title="Visualize Massing Volume">
    <p id="para"></p>

    <!-- Wrapper for Switch with Tooltip -->
    <div class="tooltip-container">
      <Switch 
        :label="switchName6" 
        :initialValue="switchValue6" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <span class="tooltip-text">Toggle to visualize the massing of the apartment block</span>
    </div>
  </CollapsiblePanel>

  <!-- Part 03: Generate Floor Layouts -->
  <CollapsiblePanel title="Generate Floor Layouts">
    <p id="para"></p>

    <div class="tooltip-container">
      <SliderInput07 
        :title="eighthSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">Please enter the target GIA in sqaure meters </span>
    </div>

    <div class="tooltip-container">
      <SliderInput05 
        :title="forthSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is having 1-bedroom apartments?
        <img src="./assets/Single Block 01.png" alt="Tooltip Image" style="width: 200px; height: 200px; margin-right: 5px;" />
      </span>
    </div>

    <div class="tooltip-container">
      <SliderInput05 
        :title="fifthSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is having 2-bedroom apartments?
        <img src="./assets/Double block 01.png" alt="Tooltip Image" style="width: 200px; height: 200px; margin-right: 5px;" />
      </span>
    </div>

    <div class="tooltip-container">
      <SliderInput05 
        :title="sixthSliderName" 
        @update="updateValue"
      />
      <span class="tooltip-text">How important is having 3-bedroom apartments?
        <img src="./assets/tripple block 02.png" alt="Tooltip Image" style="width: 200px; height: 200px; margin-right: 5px;" />
      </span>
    </div>

    <!-- Wrapper for Switch with Tooltip -->
    <div class="tooltip-container">
      <Switch 
        :label="switchName5" 
        :initialValue="switchValue5" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <span class="tooltip-text">Enable this to generate the Circulation cores and corridors</span>
    </div>

    <!-- Wrapper for Switch with Tooltip -->
    <div class="tooltip-container">
      <Switch 
        :label="switchName4" 
        :initialValue="switchValue4" 
        @update="(newVal, label) => updateValue(newVal, label)"  
      />
      <span class="tooltip-text">Enable this to start the final aggregation</span>
    </div>

    <p id="para"></p>

    <!-- Conditional SliderInput with Tooltip -->
    <div id="para2" v-if="metadata[3] && switchValue4">
      <div class="tooltip-container">
        <SliderInput02
          :value="FloorSliderValue"  
          :label="FloorSliderName.value"
          :max="metadata[3].value"
          :title="FloorSliderName"
          @update="updateValue" 
        />
        <span class="tooltip-text">Adjust the slider to generate the floors</span>
      </div>
    </div>
  </CollapsiblePanel>

      <!-- ComputeButton components, ensure they are uncommented -->

      <!-- <ComputeButton 
        title="Download 3dm" 
        @click="download('CustomStick')" 
      >Download 3dm</ComputeButton> -->
    </div>



    <div id="viewerwindow">

      <div id="viewer" class="geometry">
        <div id="Construction" class="data1">

        <BarChart :metadata="validMetadata" />


        <PieChart :data="pieChartData" />


        <PieChart2 :data="pieChart2Data" />

        <p v-if="Total_Area_ > 1">
          Total Area: {{ Total_Area_ }}
        </p>
        <p v-else>
          Please wait for compute to finish for the data to display. Move through all floor slider positions to show the total values.
        </p>

        <p id="para">Error Log:</p>
        <div id="para2" v-html="errorMessage"></div>


      </div>
        <!-- Loading Spinner positioned in front of all content -->
        <LoadingSpinner :isVisible="isComputing" />
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
  box-sizing: border-box; /* Ensures padding is included in the width/height */
  overflow: hidden; /* Prevents scrollbars */
}

.container {
  background-color: black;
  border-style: solid;
  border-color: white;
  border-radius: 25px;
  border-width: 2px;
  /* overflow: auto; Ensure scrolling if content exceeds the viewport height */
  flex-grow: 1; /* Ensures container fills remaining space */
  overflow: hidden; /* Hide scrollbars */
}

.data1 {
  background-color: rgba(0, 0, 0, 0.432);
  width: 22%;
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
  min-height: 800px; /* Change fixed height to min-height */
  padding: 20px;
  overflow: auto; /* Allow scrolling if content overflows */
  position: relative; /* Ensure stacking context */
  z-index: 10; /* Higher z-index for sidebar */
}

#intro {
  font-family: Roboto Mono, monospace;
  font-size: 14px;
  text-align: justify;
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

#Construction {
  position: relative; /* Changed to relative for proper positioning */
  text-align: left;
  margin: 0px 0px;
  z-index: 15; /* Higher than viewer but below spinner */
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

.tooltip-container {
  position: relative; /* Positioning context for absolute tooltip */
  display: block; /* Full width to ensure components don't shrink */
  margin: 10px 0; /* Margin for spacing */
  width: 100%; /* Ensure container takes full width */
}

.tooltip-text {
  visibility: hidden; /* Initially hidden */
  width: 200px; /* Width of the tooltip */
  background-color: #333; /* Dark background */
  color: #fff; /* White text */
  text-align: center; /* Centered text */
  padding: 5px 0; /* Padding for text */
  border-radius: 6px; /* Rounded corners */
  position: absolute; /* Absolute position */
  z-index: 1; /* On top of other elements */
  bottom: 125%; /* Position above the element */
  left: 50%; /* Center align */
  transform: translateX(-50%); /* Center align */
  opacity: 0; /* Transparent initially */
  transition: opacity 0.3s; /* Smooth transition */
  font-family: 'Roboto Mono', monospace; /* Font for tooltip */
}

.tooltip-container:hover .tooltip-text {
  visibility: visible; /* Show on hover */
  opacity: 1; /* Fully opaque */
}

.slider {
  width: 100%; /* Ensure sliders take full width */
}

/* Spinner overlay styles to cover the entire geometry viewer */
.spinner-overlay {
  position: absolute; /* Position absolute to cover only the parent */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%; /* Match parent's full height */
  background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100; /* High z-index to ensure it's on top of the geometry viewer content */
  pointer-events: none; /* Allow interactions with underlying content */
}

#viewer {
  width: 100%;
  height: 100vh;
  align-items: center;
  position: relative; /* Ensure #viewer is relative for absolute positioning of spinner */
  z-index: 50; /* Lower z-index than spinner to stay behind it */
  overflow: hidden; /* Ensure spinner stays within boundaries */
}

#viewerwindow {
  display: flex;
  flex-direction: column;
  width: calc(100% - 450px);
  /* height: 100vh; */
  height: 100%;
  gap: 10px;
  padding: 0px 20px;
  border-color: white;
  background-color: white;
  position: relative; /* Add relative position for stacking context */
  right: 0; /* Move to the right side */
  top: 0; /* Align to the top */
  z-index: 40; /* Lower z-index to ensure it's behind the spinner */
  box-sizing: border-box; /* Ensure padding is included in width/height */
  overflow: hidden; /* Ensure no scrollbars */
}


</style>