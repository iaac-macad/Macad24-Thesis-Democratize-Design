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
let renderer, camera, scene,  controls, container, plane


function init() {
  // https://threejs.org/docs/#api/en/renderers/WebGLRenderer
  // This object will render our scene
  // renderer = new THREE.WebGLRenderer()
  renderer = new THREE.WebGLRenderer( { antialias: true } );

  container = document.getElementById("threejs-container")

  // Rendered needs to know what's the size of the scene. 
  renderer.setSize(container.offsetWidth, container.offsetHeight)

  // We are taking element defined in <template> and appending our render to it. 
  container.appendChild(renderer.domElement)

  //orient object
  THREE.Object3D.DEFAULT_UP.set( 0, 0, 1 );

  // https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  camera = new THREE.PerspectiveCamera(75, container.offsetWidth / container.offsetHeight, 0.1, 1000)
  camera.position.set(15, -15, 10)

  // https://threejs.org/docs/?q=scene#api/en/scenes/Scene
  scene = new THREE.Scene()
  // scene.background = new THREE.Color("#f1e1f2");
  scene.background = new THREE.Color( "hsl(147, 31%, 63%)" );


  //from website*************
  const ambient = new THREE.HemisphereLight( 0xffffff, 0xbfd4d2, 3 );
	scene.add( ambient );

	const directionalLight = new THREE.DirectionalLight( 0xffffff, 0.3 );
	directionalLight.position.set( 1, 4, 3 ).multiplyScalar( 3 );
  directionalLight.castShadow = true;
	directionalLight.shadow.mapSize.setScalar( 2048 );
	directionalLight.shadow.bias = - 1e-4;
  directionalLight.shadow.normalBias = 1e-4;
	scene.add( directionalLight );

	// renderer
	// renderer.setPixelRatio( window.devicePixelRatio );
	// renderer.setSize( window.innerWidth, window.innerHeight );
	renderer.shadowMap.enabled = true;
	renderer.shadowMap.type = THREE.PCFSoftShadowMap;
	// document.body.appendChild( renderer.domElement );

  // add shadow plane
  plane = new THREE.Mesh(
	    new THREE.PlaneGeometry(),
	    new THREE.ShadowMaterial( {
	    color: 0xd81b60,
	    transparent: true,
      // transparent: false,
	    opacity: 0.075,
	    side: THREE.DoubleSide,
	  } ),
	);
	plane.position.z = - 0.1;
	// plane.rotation.x = - Math.PI / 2;
	plane.scale.setScalar( 5 );
	plane.receiveShadow = true;
	scene.add( plane );


  //*************************** */

  
  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement);
	controls.enableDamping = true;
	// controls.minDistance = 1;
	// controls.maxDistance = 10;
	// controls.target.set( 0, 0.35, 0 );
	controls.update();

  // add some ambient light here
  // const ambientlight = new THREE.AmbientLight(0xffffff, 1)
  // ambientlight.position.set(0, 0, 0)
  // scene.add(ambientlight)

  // const directionalLight = new THREE.DirectionalLight(0xffffff, 3)
  // directionalLight.position.set(0, 1, 0)
  // scene.add(directionalLight)


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


  // // clear objects from scene
  // scene.traverse((child) => {
  //   if (!child.isLight) {
  //     scene.remove(child)
  //   }
  //   console.log(child)
  // })


  //clear objects from scene except for lights and ground plane
  scene.traverse((child) => {
    if (!child.isLight) {
        // Check if the child is a mesh and its geometry is a PlaneGeometry
        if (child instanceof THREE.Mesh && child.geometry instanceof THREE.PlaneGeometry) {
            console.log('Child is a plane');
        } else {
            scene.remove(child);
        }
    }
});



  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    ///////////////////////////////////////////////////////////////////////
    // add object graph from rhino model to three.js scene
    object.traverse((child) => {

      // console.log(child)

      if (child.isLine) {

        console.log(child)
          
        if (child.userData.attributes.userStrings!= undefined && child.userData.attributes.userStrings.length > 0) {
            //get color from userStrings
            const colorData = child.userData.attributes.userStrings[0]
            const col = colorData[1]

            //convert color from userstring to THREE color and assign it
            const threeColor = new THREE.Color("rgb(" + col + ")")
            const mat = new THREE.LineBasicMaterial({ color: threeColor })
            child.material = mat
        }

      }

      scene.add(plane);
      child.castShadow = true;
      child.receiveShadow = true;

    })

    scene.add(object)

    // zoom to extents
    for (let child of scene.children) {
        if (child.type == "Object3D") {
          // zoom to extents
          zoomCameraToSelection(camera, controls, child.children)

        }
      }
      

    console.log("Compute done")
    // scene.add(plane);
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

/**
 * Helper function that behaves like rhino's "zoom to selection", but for three.js!
 */
 function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {

const box = new THREE.Box3();

for (const object of selection) {
  if (object.isLight) continue
  box.expandByObject(object);
}

const size = box.getSize(new THREE.Vector3());
const center = box.getCenter(new THREE.Vector3());

const maxSize = Math.max(size.x, size.y, size.z);
const fitHeightDistance = maxSize / (2 * Math.atan(Math.PI * camera.fov / 360));
const fitWidthDistance = fitHeightDistance / camera.aspect;
const distance = fitOffset * Math.max(fitHeightDistance, fitWidthDistance);

const direction = controls.target.clone()
  .sub(camera.position)
  .normalize()
  .multiplyScalar(distance);
controls.maxDistance = distance * 10;
controls.target.copy(center);

camera.near = distance / 100;
camera.far = distance * 100;
camera.updateProjectionMatrix();
camera.position.copy(controls.target).sub(direction);

controls.update();

}



// This will be run whenever this component is instantiated
onMounted(async() => {
  init()
  await loadRhino()
  // compute();
})

// //onUpdated is called when an input prop is changed
// // this runs compute whenever the input data changes
// onUpdated(() => {
//   compute();
// })



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