<script setup>
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue"
import GeometryView2 from "./components/GeometryView3.vue"
import SliderInput from "./components/SliderInput.vue"
import DropdownSelector from "./components/DropdownSelector.vue"
import ComputeButton from "./components/ComputeButton.vue"
import Switch from "./components/Switch.vue"
import Upload3dm from "./components/Upload3dm.vue"
import { download } from "@/scripts/compute.js"

// import def from './assets/stick08.gh' 
import def from './assets/osm_v15_3h.gh' 

let sliderName = ref("Floor"); //must match the Input name in your GH definition!
let sliderValue = ref(0); //default slider value

// let switchName = ref("Run LB"); 
// let switchValue = ref(false); 

// let switchName2 = ref("Run Initial Agg");
// let switchValue2 = ref(false);

let dropdownName = ref("Run LB")
let dropdownIndex = ref(0)

let dropdownName2 = ref("Run Initial Agg")
let dropdownIndex2 = ref(0)

const dropdownOptions = [
  { label: "Not Run", value: 0 },
  { label: "Run", value: 1 },
];

const dropdownOptions2 = [
  { label: "Not Run", value: 0 },
  { label: "Run", value: 1 },
];


let encodedFile = ref(null);
let isButtonDisabled = ref(false)

///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


function updateValue(newValue, parameterName) {

  if (parameterName === sliderName.value) {
    sliderValue.value = newValue
  }

  // else if (parameterName === switchName.value) {
  //   switchValue.value = newValue
  // }

  // else if (parameterName === switchName2.value) {
  //   switchValue2.value = newValue
  // }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  else if (parameterName === dropdownName2.value) {
    dropdownIndex2.value = newValue
  }

  console.log(parameterName + " : " + newValue)
}

function update3dmData(newData) {
  encodedFile.value = newData
  isButtonDisabled.value = false
  compute.value = true

}

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}


function runCompute(newVal) {
  compute.value = newVal
}


// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  let file
  if (!encodedFile.value) {
      file = "empty"
  }
  else file = encodedFile.value
  // console.log(encodedFile.value, file)
  data = {
    ["encodedFile"]: file,
    [sliderName.value]: Number(sliderValue.value),
    // [switchName.value]: Boolean(switchValue.value),
    // [switchName2.value]: Boolean(switchValue2.value),
    [dropdownName.value]: Number(dropdownIndex.value),
    [dropdownName2.value]: Number(dropdownIndex2.value),
  };

  return data

})

</script>



<template>
  <div id="appwindow" >
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo.png" />
      <p id="intro">Generate housing project using graph theory and aggregation</p>
      <p id="intro">Choose location, enter program requirements, and steps below.</p>

      <!-- <Upload3dm @encoded3dm="update3dmData" /> -->

      <DropdownSelector :title="dropdownName" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue"/>
      <DropdownSelector :title="dropdownName2" :options="dropdownOptions2" :val="dropdownIndex2" @update="updateValue"/>

      <!-- <Switch :title="switchName" :val=false @update="updateValue"></Switch>
      <Switch :title="switchName2" :val=false @update="updateValue"></Switch> -->

      <!-- <Switch :label="switchName" :initialValue="switchValue" @update="(newVal) => updateValue(newVal, switchName.value)" />
      <Switch :label="switchName2" :initialValue="switchValue2" @update="(newVal) => updateValue(newVal, switchName2.value)" /> -->
        
      <SliderInput :title="sliderName" :min="0" :max="20" :step="1" :val="0" @update="updateValue"></SliderInput>
      <!-- <SliderInput :title="sliderName2" :min="3.0" :max="10.0" :step="0.1" :val="4.2" @update="updateValue"></SliderInput> -->

      <ComputeButton title="Compute" @click="runCompute" :isDisabled="isButtonDisabled" />
      <ComputeButton title="Download 3dm" @click="download('CustomStick')">Download 3dm</ComputeButton>

    </div>



    <div id="viewerwindow">
      
      <div id="Construction" class="data1">
        <p id="para">1-bedroom apartments:</p>
        <div id="para2" v-if="metadata[0]">{{ metadata[0].value }}</div>

        <p id="para">2-bedroom apartments:</p>
        <div id="para2" v-if="metadata[1]">{{ metadata[1].value }}</div>

        <p id="para">3-bedroom apartments:</p>
        <div id="para2" v-if="metadata[2]">{{ metadata[2].value }}</div>

      </div>


      <div id="viewer" class="geometry">
        <GeometryView2 :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
      </div>

    </div>

  </div>
</template>
  
  
<style scoped>

/* #appwindow {
  display: flex;
  width: 100%;
  height: 100%;
  padding: 20px;
  background-color:#82bd9d
} */


#appwindow {
  display: flex;
  width: 100vw;
  height: 100vh;
  padding: 20px;
  /* background-color:#82bd9d; */
  background-color: white;
}

.container{
/* background-color: #cccccc8e; */
background-color: black;
border-style: solid;
border-color: white;
border-radius: 25px;
border-width: 2px;
}


.data1 {
  /* background-color: #cccccc8e; */
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

/* #sidebar {
  width: 310px;
  padding: 20px;
} */

#sidebar {
  width: 310px;
  height: 800px;
  padding: 20px;
}

#intro{
  font-family: Roboto Mono, monospace;
  font-size: 14px;
}


#para{
  font-family: Roboto Mono, monospace;
  font-size: 16px;
  text-align: left;
  margin-bottom: 0px;
}

#para2{
  font-family: Roboto Mono, monospace;
  font-style: italic;
  font-size: 14px;
  text-align: left;
  margin-bottom: 25px;
}

/* #viewerwindow {
  display: flex;
  flex-direction: column;
  width: calc(100% - 580px);
  height: calc(100% - 40px);
  gap: 10px;
  padding: 0px 20px;
  border-color: white;
  border-radius: 25px;
  background-color: #83be9e;
} */

#viewerwindow {
  display: flex;
  flex-direction: column;
  width: calc(100% - 580px);
  height: 100vh;
  gap: 10px;
  padding: 0px 20px;
  border-color: white;
  /* border-radius: 25px; */
  /* background-color: #83BE9E; */
  background-color: white;
}

/* #viewer {
  width: 100%;
  height: 100%;
} */

#viewer {
  width: 100%;
  height: 100vh;
  align-items: center;
  border-radius: 25px;
}


/* #Construction {
  position: absolute;
  text-align: left;
  margin: 0px 0px;
} */

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
  background: linear-gradient(90deg, #ffffff, #ff0080);
  height: 17px;
  border-radius: 15px;
  margin: 10px 0px;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 15px;
  background-color: black;
  cursor: pointer;
}
</style>