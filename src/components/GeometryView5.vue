<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>
  </div>
</template>

<script setup>
// Imports
import { onMounted, onUpdated, watch, ref } from 'vue'
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js"
import { runCompute } from "@/scripts/compute.js"
import { loadRhino } from "@/scripts/compute.js";

// Initialize the Rhino loader
const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')

const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["updateMetadata"]);

// Watch for changes in runCompute prop
watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
});

// Three.js objects
let renderer, camera, scene, controls, container, plane;
let cameraPosition, cameraTarget;

// Flag to track the first run
let isFirstRun = true;

// Function to handle custom logs and toggle spinner
function handleCustomLog(message) {
  console.log(message); // Call the original console.log

  if (message === "Running compute...") {
    window.dispatchEvent(new Event("compute-start")); // Emit custom event for compute start
  } else if (message === "Compute done.") {
    window.dispatchEvent(new Event("compute-end")); // Emit custom event for compute end
  }
}

function init() {
  // Initialize Three.js scene, renderer, etc.
  renderer = new THREE.WebGLRenderer({ antialias: true });
  container = document.getElementById("threejs-container");
  renderer.setSize(container.offsetWidth, container.offsetHeight);
  container.appendChild(renderer.domElement);
  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);

  camera = new THREE.PerspectiveCamera(75, container.offsetWidth / container.offsetHeight, 0.1, 1000);
  camera.position.set(15, -15, 10);
  scene = new THREE.Scene();
  scene.background = new THREE.Color("#add8e6");

  const ambient = new THREE.HemisphereLight(0xffffff, 0xbfd4d2, 5);
  scene.add(ambient);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 5);
  directionalLight.position.set(1, 4, 3).multiplyScalar(3);
  directionalLight.castShadow = true;
  directionalLight.shadow.mapSize.setScalar(2048);
  directionalLight.shadow.bias = -1e-4;
  directionalLight.shadow.normalBias = 1e-4;
  scene.add(directionalLight);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.update();

  plane = new THREE.Mesh(
    new THREE.PlaneGeometry(5000, 5000),
    new THREE.MeshBasicMaterial({
      color: 0xd3d3d3,
      side: THREE.DoubleSide,
    })
  );
  plane.position.z = -0.1;
  plane.receiveShadow = true;
  scene.add(plane);

  animate();
}

// Function to run computation
async function compute() {
  handleCustomLog("Running compute..."); // Show spinner
  console.log("Running compute... \ndata sent: ", props.data);

  const doc = await runCompute(props.data, props.path);

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }

  // Preserve camera state
  cameraPosition = camera.position.clone();
  cameraTarget = controls.target.clone();

  // Clear objects from scene except for lights and ground plane
  scene.traverse((child) => {
    if (!child.isLight) {
      if (child instanceof THREE.Mesh && child.geometry instanceof THREE.PlaneGeometry) {
        console.log('Child is a plane');
      } else {
        scene.remove(child);
      }
    }
  });

  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    object.traverse((child) => {
      if (child.isLine) {
        const mat = new THREE.LineBasicMaterial({
          color: new THREE.Color(0x000000),
          transparent: true,
          opacity: 0.1,
          linewidth: 1
        });
        child.material = mat;
      }
      scene.add(plane);
      child.castShadow = true;
      child.receiveShadow = true;
    });

    scene.add(object);

    // Restore camera state
    camera.position.copy(cameraPosition);
    controls.target.copy(cameraTarget);
    controls.update();

    // Zoom to extents only on the first run
    if (isFirstRun) {
      for (let child of scene.children) {
        if (child.type == "Object3D") {
          zoomCameraToSelection(camera, controls, child.children);
        }
      }
      isFirstRun = false; // Set the flag to false after the first run
    }

    handleCustomLog("Compute done."); // Hide spinner
    console.log("Compute done");
  });
}

// For controls update
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

// This will be run whenever the window is resized
window.addEventListener("resize", onWindowResize);
function onWindowResize() {
  const height = container.offsetHeight;
  const width = container.offsetWidth;

  camera.aspect = width / height;
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);
}

// Helper function for zooming the camera
function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {
  const box = new THREE.Box3();
  for (const object of selection) {
    if (object.isLight) continue;
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
onMounted(async () => {
  init();
  await loadRhino();
});

// Run compute as soon as any input changes
onUpdated(() => {
  compute();
});
</script>

<style scoped>
#viewport {
  height: 100vh;
  width: 100vw;
  min-width: 200px;
  min-height: 200px;
  position: inherit;
}

#threejs-container {
  height: 100vh;
  width: 100vw;
  min-width: 200px;
  min-height: 200px;
  position: inherit;
}
</style>
