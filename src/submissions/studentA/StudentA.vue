<script setup>
import { ref, computed } from "vue"

// Import components
import SliderInput from "../../components/SliderInput.vue"
import GeometryView from "../../components/GeometryView.vue"
import def from './assets/Spikey_hops.gh'


// Define the tittle of your project
const title = ref("Project title")

// Define Grasshopper input names
const firstSliderName = ref("Radius") //must match the Input name in your GH definition!
const firstSlider = ref(11) //default slider value

const secondSliderName = ref("Count") //must match the Input name in your GH definition!
const secondSlider = ref(27) //default slider value

const thirdSliderName = ref("Length") //must match the Input name in your GH definition!
const thirdSlider = ref(9) //default slider value


///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])


function updateValue(newValue, parameterName) {
  console.log(parameterName)

  if (parameterName === firstSliderName.value) {
    firstSlider.value = newValue;

  } else if (parameterName === secondSliderName.value) {
    secondSlider.value = newValue;

  } else if (parameterName === thirdSliderName.value) {
    thirdSlider.value = newValue;
  }
}

function receiveMetadata(newValue) {
  console.log(newValue);
  metadata.value = newValue;
}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    [firstSliderName.value]: Number(firstSlider.value),
    [secondSliderName.value]: Number(secondSlider.value),
    [thirdSliderName.value]: Number(thirdSlider.value)
  };
  return data;
});
</script>


<template>
    <div class="gh-inputs-container">
      <h2 class="submission-title">{{ title }}</h2>
      <p>Description of the project </p>
      
      <SliderInput v-bind:title="firstSliderName"
        v-bind:min="11"
        v-bind:max="15"
        v-bind:step="1"
        v-bind:val="firstSlider"
        v-on:update="updateValue"
      />
  
      <SliderInput  v-bind:title="secondSliderName"
        v-bind:min="25"
        v-bind:max="30"
        v-bind:step="1"
        v-bind:val="secondSlider"
        v-on:update="updateValue"
      />
  
      <SliderInput v-bind:title="thirdSliderName"
        v-bind:min="3"
        v-bind:max="12"
        v-bind:step="1"
        v-bind:val="thirdSlider"
        v-on:update="updateValue"
      />
      

  
      <div v-if="metadata.length > 0">
        <h2 class="submission-title">Received values:</h2>
        <ol>
          <li v-for="item in metadata" :key="item.name">
            {{ item.name }}: {{ item.value }}
          </li>
        </ol>
      </div>
    </div>
    
    <div class="geometry-view">
      <GeometryView
        v-bind:data="computeData"
        v-bind:path="path"
        v-on:updateMetadata="receiveMetadata"
      />
    </div>

</template>
  
 
  
  <style scoped>
  .gh-inputs-container {
    display: flex;
    flex-direction: column;
    height: 73%;
    align-items: flex-start;
    padding: 15px;
    margin: 20px;
  }

  .geometry-view {
    display: flex;
    margin: 12px;
    border: none;
    min-width: 200px;
    position: inherit;
  }
  
  .submission-title {
    text-align: left;
    padding: 1px;
  }
  
  ol li {
    padding: 1px;
    margin-left: 30px;
    font-family: "Roboto Mono", monospace;
  }
  
  ol {
    border-radius: 10px;
    padding: 5px 1px;
    margin-left: 1px;
    width: 330px;
    list-style-type: circle;
  }
  
  .legend-image {
    margin-top:20px;
    height:600px;
  }
  </style>