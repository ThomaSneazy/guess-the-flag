<template>
  <div ref="container" class="background-3d"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';

const container = ref(null);
let scene, camera, renderer;

onMounted(() => {
  initScene();
  animate();
  window.addEventListener('resize', onWindowResize);
});

onUnmounted(() => {
  if (renderer) {
    renderer.dispose();
  }
  window.removeEventListener('resize', onWindowResize);
});

function initScene() {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000, 0.8); 
  container.value.appendChild(renderer.domElement);

  camera.position.z = 5;
}

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
</script>

<style scoped>
.background-3d {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}
</style>