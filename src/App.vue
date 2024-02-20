<!-- // Script is in some way equivalent to script.js. This is place
to define variables, methods and imports of other Vue compoennts. -->
<script setup>
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
import Upload3dm from "./components/Upload3dm.vue"



import def from './assets/rhino_input.gh' //import Grasshopper definition for assets

let firstSliderName = ref("Radius") //must match the Input name in your GH definition!
let firstSliderValue = ref(10) //default slider value

let secondSliderName = ref("Count") //must match the Input name in your GH definition!
let secondSliderValue = ref(10) //default slider value

const encodedFile = ref(null);

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
  
  console.log( parameterName + " : " + newValue)
}

function update3dmData(newData) {
  encodedFile.value = newData
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
    [secondSliderName.value]: Number(secondSliderValue.value)
  };

  return data
})



</script>

<!-- Template is a HTML-based syntax that allows you to bind the rendered DOM elements
with data, objects, functions etc. -->
<template>
  <Header title="Digital Tools for Cloud-based Data Management - Session 04 - Compute"></Header>

  <div id="content">
    <div id="sidebar" class="container">

      <Upload3dm @encoded3dm="update3dmData"/>
      
        
      <SliderInput 
          :title="firstSliderName"
          :min = "1"
          :max = "10"
          :step = "1"
          :val = "5"
          @update="updateValue"></SliderInput>

          <SliderInput 
          :title="secondSliderName"
          :min = "1"
          :max = "10"
          :step = "1"
          :val = "5"
          @update="updateValue"></SliderInput>


          <ComputeButton title="Compute" @click="runCompute" />


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

<!-- Style is for CSS styling -->
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


</style>

