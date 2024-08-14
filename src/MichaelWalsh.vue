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

//Import and define Store
import {useComputeStore} from "@/stores/computeStore.js"
const computeStore = useComputeStore()

import def from "./assets/Genius.gh" 

let firstSliderName = ref("Green_Space") //must match the Input name in your GH definition!
let firstSliderValue = ref(5) //default slider value

let secondSliderName = ref("Roads") //must match the Input name in your GH definition!
let secondSliderValue = ref(5) //default slider value

let thirdSliderName = ref("Amenities") //must match the Input name in your GH definition!
let thirdSliderValue = ref(5) //default slider value

let fourthSliderName = ref("Public_Transport") //must match the Input name in your GH definition!
let fourthSliderValue = ref(5) //default slider value

let fifthSliderName = ref("Internet_Speed") //must match the Input name in your GH definition!
let fifthSliderValue = ref(5) //default slider value

let sixthSliderName = ref("Education") //must match the Input name in your GH definition!
let sixthSliderValue = ref(5) //default slider value


///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


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

  else if (parameterName === fourthSliderName.value) {
    fourthSliderValueSliderValue.value = newValue
  }

  else if (parameterName === fifthSliderName.value) {
    fifthSliderValue.value = newValue
  }

  else if (parameterName === sixthSliderName.value) {
    sixthSliderValue.value = newValue
  }
}

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
    [fourthSliderName.value]: Number(fourthSliderValue.value),
    [fifthSliderName.value]: Number(fifthSliderValue.value),
    [sixthSliderName.value]: Number(sixthSliderValue.value),
  };

  return data
})

onBeforeMount( () => {
})





</script>

<!-- Template is a HTML-based syntax that allows you to bind the rendered DOM elements
with data, objects, functions etc. -->
<template>
    <div id="sidebar" class="container" position = 'relative' >
      <div>
      <img src="./assets/Logo.png" alt = "logo" style = "width:30px ; height: 30px ; float: right; margin-right: 100px;">
      <h3  style = " font-size: x-large; ">Genius Loci </h3> 
      </div>
      <div style = "margin-top: 20px;">  <p>
        
        
        
        Hello and welcome to Genius Loci!
          The app to help you find your ideal home in Manchester.
          Please input your preference below (0 being low and 10 being high) and the Map will display the top performing postcodes suited to your choices. <br>
          Please allow some time for calculations.
      </p>
      </div>
     
  
      <SliderInput :title="firstSliderName" @update="updateValue"/> 

      <SliderInput :title="secondSliderName" @update="updateValue"/> 

      <SliderInput :title = "thirdSliderName" @update="updateValue"/> 

      <SliderInput :title = "fourthSliderName" @update="updateValue"/>

      <SliderInput :title = "fifthSliderName" @update="updateValue"/>

      <SliderInput :title = "sixthSliderName" @update="updateValue"/>
      
      <p>
        Key:
        </p>

      <img src="./assets/Key_1.png" alt = "Key" style = "width:140px ; height: 180px ; float: right; margin-right: 100px;">

    </div>

      
    <div id="viewer" class="container">

    <GeometryView :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
    </div>
  
     

        <div id="bottombar" v-if="metadata && metadata.length > 0" class="container" style="display: flex; color: white; justify-content: space-around;"> 
        <div style="font-size: x-small; line-height: 1.8;">Rank <br>
          First <br>
          Second <br>
          Third <br>
          Fourth <br>
          Fifth</div>
        <div style="font-size: x-small; line-height: 1.4; color: white;  ">Postcode  
          <p style = "font-size: xx-small; line-height: 0.9;">{{ metadata[20].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[21].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[22].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[23].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[24].value }}</p>   </div>
        <div style="font-size: x-small; line-height: 1.4;">Average Price [£] 
          <p style = "font-size: xx-small; line-height: 0.9;">{{ metadata[0].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[1].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[2].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[3].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[4].value }}</p> </div>
        <div style="font-size: x-small; line-height: 1.4;">Average Area [Sqm] 
          <p style = "font-size: xx-small; line-height: 0.9;">{{ metadata[5].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[6].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[7].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[8].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[9].value }}</p> </div>
        <div style="font-size: x-small; line-height: 1.4;">Approx. Price/Sqm [£/Sqm] 
          <p style = "font-size: xx-small; line-height: 0.9;">{{ metadata[10].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[11].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[12].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[13].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[14].value }}</p> </div>
        <div style="font-size: x-small; line-height: 1.4;">Most Common Typology 
          <p style = "font-size: xx-small; line-height: 0.9;">{{ metadata[15].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[16].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[17].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[18].value }}</p>  
          <p style = "font-size: xx-small;  line-height: 0.9;">{{ metadata[19].value }}</p>  </div>
      </div> 

      <div id="footer" class="container" > 
        <p style="margin-left: 20px ; margin-right: 20px;">Postcode district derived boundaries reproduced under the Open Government Licence v3.0. Postal Boundaries © GeoLytix copyright and database right 2012. Contains Ordnance Survey data © Crown copyright and database right 2012. Contains Royal Mail data © Royal Mail copyright and database right 2012. Contains National Statistics data © Crown copyright and database right 2012. Price Paid data produced by HM Land Registry © Crown copyright 2017.
       </p>
      </div> 


      

  </template>
    
   
    
  <style scoped>
  
  .container{
    background-color: rgb(0, 0, 0);
  border-style: dotted;
  border-width: 1px;
  border-radius: 10px;
  font-family: 'Courier New', Courier, monospace;
  font-size: x-small;
  }

  
  
  #content {
  
    display: flex;
    height: calc(100vh - 68px);
  }
  
  #sidebar {
  
    width: 310px;
    padding: 10px;
    margin-top: 20px;
    margin-bottom: 15px;
  }
  
  #viewer {
    width: calc(100% - 340px);
    margin-bottom: 150px;
    margin-left: 10px;
    margin-top: 20px;
  }

  #bottombar {
  
  position: absolute; 
  bottom: 0;
  height: 100px;
  width: calc(100% - 360px);
  padding: 10px;
  margin-left: 340px;
}

#footer {
 position: absolute; 
 bottom: -45px;
 color:#535253;
 width: calc(100%);
 margin-top: auto;
 font-size: xx-small;
 
}
  
.modern-range {
  -webkit-appearance: none;
  width: 100%;
  background: linear-gradient(90deg, hsl(0, 0%, 100%), #535253);
  height: 17px;
  border-radius: 15px;
  margin: 10px 0px;
  font-family: 'Courier New', Courier, monospace;
}

/* Apply font size to the container */
.range-container {
  font-size: small;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 15px;
  background-color: rgb(59, 59, 59);
  cursor: pointer;
}
</style>