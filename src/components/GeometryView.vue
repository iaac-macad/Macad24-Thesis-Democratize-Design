<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>

</div>
</template>

<script setup>
// Imports;
import { onMounted, onUpdated, watch } from 'vue'
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js"
import { runCompute } from "@/scripts/compute.js"
import { loadRhino } from "@/scripts/compute.js";


const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')


const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["updateMetadata"]);  


watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
})


// Three js objects
let renderer, camera, scene,  controls, container


function init() {
  // https://threejs.org/docs/#api/en/renderers/WebGLRenderer
  // This object will render our scene
  renderer = new THREE.WebGLRenderer()

  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);

  container = document.getElementById("threejs-container")

  // Rendered needs to know what's the size of the scene. 
  renderer.setSize(container.offsetWidth, container.offsetHeight)

  // We are taking element defined in <template> and appending our render to it. 
    container.appendChild(renderer.domElement)

  

// https://threejs.org/docs/#api/en/cameras/OrthographicCamera
camera = new THREE.OrthographicCamera(
  container.offsetWidth / -0.08,   // left
  container.offsetWidth / 0.08,    // right
  container.offsetHeight / 0.08,   // top
  container.offsetHeight / -0.08,  // bottom
  1,
  200000000
);

// Set the position to the opposite corner
camera.position.set(100000, -100000, 100000);

// Set the target to the center
camera.up.set(0, 0, 1); // Set the up direction to be along the Z-axis
camera.lookAt(new THREE.Vector3(0, 0, 0));

// Remove the perspective camera settings
// camera = new THREE.PerspectiveCamera(10, 2, 1, 200000000);
// camera.position.set(-100000, 0, 0);
  // https://threejs.org/docs/?q=scene#api/en/scenes/Scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color("#f5f6fa")

  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement)
  controls.target.set(0, 0, 0);  // Set the target to the center of your scene
  controls.enableDamping = true;  // an animation loop is required when either damping or auto-rotation are enabled
  controls.dampingFactor = 0.25; 

  



  // add some ambient light here
  const directionalLight = new THREE.DirectionalLight(0xffffff);
  directionalLight.intensity = 2;
  scene.add(directionalLight);


  // add fun shape
  animate()
}


// this function runs Compute
async function compute() {
  console.log("Runnning compute... \ndata sent: ", props.data)
  const doc = await runCompute(props.data, props.path)

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }
  
  
  // clear objects from scene
  scene.traverse((child) => {
    if (!child.isLight) {
      scene.remove(child)
    }
  })


  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    ///////////////////////////////////////////////////////////////////////
    // add object graph from rhino model to three.js scene
    object.traverse((child) => {

      console.log(child)

      if (child.isLine) {


      }
    })



    scene.add(object);
    console.log("Compute done")
  });
}



// for controls update
function animate() {
  // https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame
  // native Javascript function that tells your browser you are animating!
  requestAnimationFrame(animate);
  controls.update();


  renderer.render(scene, camera);
}

// This will be run whenever the window is resized
window.addEventListener("resize", onWindowResize);
function onWindowResize() {

  const height = container.offsetHeight
  const width = container.offsetWidth

  camera.aspect = width/ height
  camera.updateProjectionMatrix();


  renderer.setSize(width, height)

}


// This will be run whenever this component is instantiated
onMounted(async() => {
  init()
  await loadRhino()
  compute();
})

//onUpdated is called when an input prop is changed
onUpdated(() => {
  compute();
})



</script>

<style scoped>


#viewport {

  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

#threejs-container {
  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

</style>