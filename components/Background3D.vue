<template>
  <div ref="container" class="background-3d"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';

const container = ref(null);
let scene, camera, renderer, starField;

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
  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  container.value.appendChild(renderer.domElement);

  // Fond noir pour l'espace
  scene.background = new THREE.Color(0x000000);

  // Création du champ d'étoiles
  const starGeometry = new THREE.BufferGeometry();
  const starMaterial = new THREE.PointsMaterial({
    color: 0xFFFFFF,
    size: 0.1,
    sizeAttenuation: true
  });

  const starVertices = [];
  for (let i = 0; i < 10000; i++) {
    const x = (Math.random() - 0.5) * 2000;
    const y = (Math.random() - 0.5) * 2000;
    const z = (Math.random() - 0.5) * 2000;
    starVertices.push(x, y, z);
  }

  starGeometry.setAttribute('position', new THREE.Float32BufferAttribute(starVertices, 3));
  starField = new THREE.Points(starGeometry, starMaterial);
  scene.add(starField);

  camera.position.z = 1;
}

function animate() {
  requestAnimationFrame(animate);

  // Rotation du champ d'étoiles
  starField.rotation.y += 0.0002;
  starField.rotation.x += 0.0001;

  // Effet de "zoom" pour la profondeur
  camera.position.z = Math.sin(Date.now() * 0.0001) * 0.5 + 1.5;

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