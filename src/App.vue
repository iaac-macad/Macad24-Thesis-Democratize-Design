<script setup>
// Understanding ref article: https://blog.logrocket.com/understanding-vue-refs/#:~:text=Ref%20s%20are%20Vue.,element%20in%20your%20Vue%20instance.
// When ref attribute is added to element, this element then can be referenced
// in template. It is sort of templatecement of getElementById (but better)
import { ref, onBeforeMount, computed, watch } from "vue";

// Import other Vue components in order to add them to a template.
import GeometryView4 from "./components/GeometryView4.vue";
import SliderInput from "./components/SliderInput.vue";
import Switch from "./components/Switch02.vue";
import ComputeButton from "./components/ComputeButton.vue";

//Import and define Store
import {useComputeStore} from "@/stores/computeStore.js"
const computeStore = useComputeStore()

import def from './assets/osm_v17.gh';

//-------------------------------------------------------------------------------------------------------------------------------------------

//Part 01 Inputs 

//Need to add the text input for the site selection 

//Part 02 inputs 

const firstSliderName = ref("2D_UrbanField") //must match the Input name in your GH definition!
const firstSliderValue = ref(0.7) //default slider value

const secondSliderName = ref("2E_RadField") //must match the Input name in your GH definition!
const secondSliderValue = ref(0.5) //default slider value

const thirdSliderName = ref("2F_SVField") //must match the Input name in your GH definition!
const thirdSliderValue = ref(0.0) //default slider value

const switchName = ref("2A_Init Init Aggr");
const switchValue = ref(false);

const switchName2 = ref("2B_Run Initial Aggr");
const switchValue2 = ref(false);

const switchName3 = ref("2C_Run LB");
const switchValue3 = ref(false);

//Part 03 inputs 

const switchName4 = ref("3A_Init Final Agg");
const switchValue4 = ref(false);

const switchName5 = ref("3B_Cores");
const switchValue5 = ref(false);

const switchName6 = ref("3C_Reset");
const switchValue6 = ref(false);

const switchName7 = ref("3D_Record");
const switchValue7 = ref(false);

// Part 04 - Counter Output

const countername = ref("4_Floor")
const counterValue = ref(0); // New reactive reference to store counter output

///-----------------------------------------------------------------------------------------------------------------------------------------------
const encodedFile = ref(null);
const isButtonDisabled = ref(false);

let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


///-----------------------------------------------------------------------------------------------------------------------------------------------


function updateValue(newValue, parameterName) {
  console.log(parameterName)

  if (parameterName === firstSliderName.value) {
    firstSliderValue.value = newValue
  } 
  
  else if (parameterName === secondSliderName.value) {
    secondSliderValue.value = newValue
  }
  
  else if (parameterName === thirdSliderName.value) {
    thirdSliderValue.value = newValue
  }

  else if (parameterName === switchName.value) {
    switchValue.value = newValue;
  } 

  else if (parameterName === switchName2.value) {
    switchValue2.value = newValue;
  } 

  else if (parameterName === switchName3.value) {
    switchValue3.value = newValue;
  } 

  else if (parameterName === switchName4.value) {
    switchValue4.value = newValue;
  } 

  else if (parameterName === switchName5.value) {
    switchValue5.value = newValue;
  } 

  else if (parameterName === switchName6.value) {
    switchValue6.value = newValue;
  } 
  
  else if (parameterName === switchName7.value) {
    switchValue7.value = newValue;
  } 

  else if (parameterName === countername.value) {
    countername.value = newValue;
  } 

  console.log(`${parameterName} updated to: ${newValue}`);

}

//--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

// Watch effect to trigger runCounter when switchValue7 is true
watch(switchValue7, (newVal) => {
  if (newVal) {
    console.log('Switch 07 is true, triggering compute process...');
    runCompute(true);  // Trigger compute with newVal as true
  }
});

// Function to run compute process
function runCompute(newVal) {
  console.log('Compute triggered, starting counter...');
  compute.value = newVal;
  runCounter();  // Call runCounter within the compute process
}



//-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}


// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
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

  console.log("Computed data:", dataObject); // Debugging
  return dataObject;
});

//--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

// Watch reactive values for debugging
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

onBeforeMount( () => {
})

//--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


</script>

<!-- Template is a HTML-based syntax that allows you to bind the rendered DOM elements
with data, objects, functions etc. -->
<template>
  <div id="appwindow">
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo.png" />
      <p id="intro">Generate housing project using graph theory and aggregation</p>
      <ComputeButton 
        title="Start" 
        @click="() => runCompute(true)" 
        :isDisabled="isButtonDisabled"/>  
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