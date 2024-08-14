<script setup>
import { ref, onBeforeMount, computed, onMounted } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue"
import GeometryView2 from "./components/GeometryView.vue"
import SliderInput from "./components/SliderInput.vue"
import DropdownSelector from "./components/DropdownSelector.vue"
import ComputeButton from "./components/ComputeButton.vue"
import Upload3dm from "./components/Upload3dm.vue"
import { download } from "@/scripts/compute.js"




//Import and define Store
import {useComputeStore} from "@/stores/computeStore.js"
const computeStore = useComputeStore()





import def from './assets/stick08.gh' 

let sliderName = ref("Interest") //must match the Input name in your GH definition!
let sliderValue = ref(10) //default slider value

let sliderName2 = ref("Thickness (mm)") 
let sliderValue2 = ref(4.2) 

let dropdownName = ref("Stick Length")
let dropdownIndex = ref(2)

let dropdownName2 = ref("Birth Month")
let dropdownIndex2 = ref(3)

let dropdownName3 = ref("Birth Day")
let dropdownIndex3 = ref(25)

const dropdownOptions = [
  { label: "Short", value: 0 },
  { label: "Medium", value: 1 },
  { label: "Long", value: 2 },
  { label: "Custom", value: 3 },
];

const dropdownOptions2 = [
  { label: "Janurary", value: 1 },
  { label: "February", value: 2 },
  { label: "March", value: 3 },
  { label: "April", value: 4 },
  { label: "May", value: 5 },
  { label: "June", value: 6 },
  { label: "July", value: 7 },
  { label: "August", value: 8 },
  { label: "September", value: 9 },
  { label: "October", value: 10 },
  { label: "November", value: 11 },
  { label: "December", value: 12 },
];

const dropdownOptions3 = [
  { label: "1", value: 1 },
  { label: "2", value: 2 },
  { label: "3", value: 3 },
  { label: "4", value: 4 },
  { label: "5", value: 5 },
  { label: "6", value: 6 },
  { label: "7", value: 7 },
  { label: "8", value: 8 },
  { label: "9", value: 9 },
  { label: "10", value: 10 },
  { label: "11", value: 11 },
  { label: "12", value: 12 },
  { label: "13", value: 13 },
  { label: "14", value: 14 },
  { label: "15", value: 15 },
  { label: "16", value: 16 },
  { label: "17", value: 17 },
  { label: "18", value: 18 },
  { label: "19", value: 19 },
  { label: "20", value: 20 },
  { label: "21", value: 21 },
  { label: "22", value: 22 },
  { label: "23", value: 23 },
  { label: "24", value: 24 },
  { label: "25", value: 25 },
  { label: "26", value: 26 },
  { label: "27", value: 27 },
  { label: "28", value: 28 },
  { label: "29", value: 29 },
  { label: "30", value: 30 },
  { label: "31", value: 31 },
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

  else if (parameterName === sliderName2.value) {
    sliderValue2.value = newValue
  }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  else if (parameterName === dropdownName2.value) {
    dropdownIndex2.value = newValue
  }

  else if (parameterName === dropdownName3.value) {
    dropdownIndex3.value = newValue
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
    [sliderName2.value]: Number(sliderValue2.value),
    [dropdownName.value]: Number(dropdownIndex.value),
    [dropdownName2.value]: Number(dropdownIndex2.value),
    [dropdownName3.value]: Number(dropdownIndex3.value),
  };

  return data
})



onBeforeMount( () => {
})


onMounted(() => {
  loadRhino();
})


</script>


<template>
  <div id="appwindow" >
    <div id="sidebar" class="container">
      <img class="mainlogo" alt="logo" src="./assets/logo.png" />
      <p id="intro">Design a custom branch for your best friend. Use your dog's birthday to generate a one-of-a-kind stick!</p>
      <p id="intro">For more control, upload your 3dm file with a simple polyline (~1m long or less) and a basic brep or mesh shape. 
        The brep/mesh will control the direction of the smaller twigs. Use the inputs below to customize.</p>

      <Upload3dm @encoded3dm="update3dmData" />

      <DropdownSelector :title="dropdownName" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue"/>
      <DropdownSelector :title="dropdownName2" :options="dropdownOptions2" :val="dropdownIndex2" @update="updateValue"/>
      <DropdownSelector :title="dropdownName3" :options="dropdownOptions3" :val="dropdownIndex3" @update="updateValue"/>

      <SliderInput :title="sliderName" :min="6" :max="16" :step="1" :val="10" @update="updateValue"></SliderInput>
      <SliderInput :title="sliderName2" :min="3.0" :max="10.0" :step="0.1" :val="4.2" @update="updateValue"></SliderInput>

      <ComputeButton title="Compute" @click="runCompute" :isDisabled="isButtonDisabled" />
      <ComputeButton title="Download 3dm" @click="download('CustomStick')">Download 3dm</ComputeButton>

    </div>



    <div id="viewerwindow" >
      
      <div id="Construction" class="data1">
        <p id="para">Price:</p>
        <div id="para2" v-if="metadata[1]">${{ metadata[1].value }}</div>

        <p id="para">Ideal for Breeds like:</p>
        <div id="para2" v-if="metadata[2]">{{ metadata[2].value }}</div>

        <p id="para">Level of Car Scratch Damage:</p>
        <div id="para2" v-if="metadata[3]">{{ metadata[3].value }}</div>

      </div>


      <div id="viewer" class="geometry">
        <GeometryView2 :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
      </div>

    </div>

  </div>
</template>
  
  
<style scoped>

#appwindow {
  display: flex;
  width: 100%;
  height: 100%;
  padding: 20px;
  background-color:#82bd9d
}

.container{
background-color: #cccccc8e;
border-style: solid;
border-color: white;
border-radius: 25px;
border-width: 2px;
}


.data1 {
  background-color: #cccccc8e;
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

#viewerwindow {
  display: flex;
  flex-direction: column;
  width: calc(100% - 580px);
  height: calc(100% - 40px);
  gap: 10px;
  padding: 0px 20px;
  border-color: white;
  border-radius: 25px;
  background-color: #83be9e;
}

#viewer {
  width: 100%;
  height: 100%;
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











<!-- <script setup>
// Understanding ref article: https://blog.logrocket.com/understanding-vue-refs/#:~:text=Ref%20s%20are%20Vue.,element%20in%20your%20Vue%20instance.
// When ref attribute is added to element, this element then can be referenced
// in template. It is sort of templatecement of getElementById (but better)
import { ref, onBeforeMount, computed } from "vue";
import { loadRhino } from "@/scripts/compute.js";

// Import other Vue components in order to add them to a template.
import Header from "./components/Header.vue"
import GeometryView from "./components/GeometryView.vue"
import SliderInput from "./components/SliderInput.vue"
import ComputeButton from "./components/ComputeButton.vue"
import DropdownSelector from "./components/DropdownSelector.vue"
import Upload3dm from "./components/Upload3dm.vue"



import def from './assets/rhino_input.gh' //import Grasshopper definition for assets

let firstSliderName = ref("X") //must match the Input name in your GH definition!
let firstSliderValue = ref(1) //default slider value

let secondSliderName = ref("Y") //must match the Input name in your GH definition!
let secondSliderValue = ref(1) //default slider value

let thirdSliderName = ref("Z") //must match the Input name in your GH definition!
let thirdSliderValue = ref(1) //default slider value

let dropdownName = ref("Type")
let dropdownIndex = ref(0)

const dropdownOptions = [
  { label: "Mesh", value: 0 },
  { label: "Brep", value: 1},
  { label: "SubD", value: 2 },
  { label: "Curve", value: 3 },
  
];


let encodedFile = ref(null);
let isButtonDisabled = ref(true)

///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


function updateValue(newValue, parameterName) {

  if (parameterName === firstSliderName.value) {
    firstSliderValue.value = newValue
  } 
  
  else if (parameterName === secondSliderName.value) {
    secondSliderValue.value = newValue
  }

  else if (parameterName === thirdSliderName.value) {
    thirdSliderValue.value = newValue
  }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  console.log( parameterName + " : " + newValue)
}

function update3dmData(newData) {
  encodedFile.value = newData
  isButtonDisabled.value = false
  compute.value = true  
  
}


//receive metadata from GeometryView component
function receiveMetedata(newMetadata) {
  metadata.value = newMetadata
  runCompute.value = false
}

function runCompute(newVal){
  compute.value = newVal  


}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    ["encodedFile"]: encodedFile.value, 
    [firstSliderName.value]: Number(firstSliderValue.value),
    [secondSliderName.value]: Number(secondSliderValue.value),
    [thirdSliderName.value]: Number(thirdSliderValue.value),
    [dropdownName.value]: Number(dropdownIndex.value)
  };

  return data
})



</script>

<template>
  <Header title="Digital Tools for Cloud-based Data Management - Session 06 - File Input"></Header>

  <div id="content">
    <div id="sidebar" class="container">

      <Upload3dm @encoded3dm="update3dmData"/>
      
        
      <SliderInput 
          :title="firstSliderName"
          :min = "1"
          :max = "10"
          :step = "1"
          :val = "firstSliderValue"
          @update="updateValue"></SliderInput>

          <SliderInput 
          :title="secondSliderName"
          :min = "1"
          :max = "10"
          :step = "1"
          :val = "secondSliderValue"
          @update="updateValue"></SliderInput>

          <SliderInput 
          :title="thirdSliderName"
          :min = "1"
          :max = "10"
          :step = "1"
          :val = "thirdSliderValue"
          @update="updateValue"></SliderInput>

          <DropdownSelector 
          :title="dropdownName" 
          :options="dropdownOptions" 
          :val="dropdownIndex" 
          @update="updateValue"
          />

          <ComputeButton 
          title="Compute" 
          @click="runCompute" 
          :isDisabled="isButtonDisabled"

          />


    </div>
      
    <div id="viewer" class="container">

      <GeometryView
      :data="computeData"
      :path="path"
      :runCompute="compute"
      @updateMetadata="receiveMetedata"
      />

    </div>
  </div>

</template>

<style>


#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;

      
  background-color: rgb(59, 59, 59); 
  color: white;
}


.logo-image {
  height: 3.25rem;
  padding: 0.5rem;
}

h2 {
  font-size: 1.125rem;
  font-weight: 600;
  letter-spacing: 0.01em;
}

#content{
    
    display: flex;
    height: calc(100vh - 68px);
}

#sidebar{
    
    width:310px;
    padding: 10px;
}

#viewer{
    width: calc(100% - 310px);
}


.container{

    border-style: dotted;
    border-width: 1px;

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


</style> -->