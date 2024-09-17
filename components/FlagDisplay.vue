<template>
  <div ref="container" class="flag-display"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import * as THREE from 'three';
import { gsap } from 'gsap';

const props = defineProps({
  countryCode: {
    type: String,
    required: true
  }
});

const container = ref(null);
let scene, camera, renderer, particleSystem, raycaster, mouse;
let originalPositions, targetPositions;

onMounted(() => {
  initScene();
  animate();
  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('resize', onWindowResize);
});

onUnmounted(() => {
  if (renderer) {
    renderer.dispose();
  }
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('resize', onWindowResize);
});

watch(() => props.countryCode, updateFlagTexture);

function initScene() {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  container.value.appendChild(renderer.domElement);

  raycaster = new THREE.Raycaster();
  mouse = new THREE.Vector2();

  camera.position.z = 3;

  createParticleSystem();
}

function createParticleSystem() {
  const textureLoader = new THREE.TextureLoader();
  const particleTexture = createCircleTexture();

  textureLoader.load(`https://flagcdn.com/w1280/${props.countryCode.toLowerCase()}.png`, (texture) => {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    canvas.width = texture.image.width;
    canvas.height = texture.image.height;
    ctx.drawImage(texture.image, 0, 0);

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;

    const geometry = new THREE.BufferGeometry();
    const particles = 100000;
    const positions = new Float32Array(particles * 3);
    const colors = new Float32Array(particles * 3);

    for (let i = 0; i < particles; i++) {
      const x = (Math.random() - 0.5) * 2.8;
      const y = (Math.random() - 0.5) * 2;
      const z = (Math.random() - 0.5) * 0.05;

      positions[i * 3] = x;
      positions[i * 3 + 1] = y;
      positions[i * 3 + 2] = z;

      const pixelX = Math.floor((x + 1.5) / 3 * canvas.width);
      const pixelY = Math.floor((1 - (y + 1) / 2) * canvas.height);
      const pixelIndex = (pixelY * canvas.width + pixelX) * 4;

      colors[i * 3] = data[pixelIndex] / 255;
      colors[i * 3 + 1] = data[pixelIndex + 1] / 255;
      colors[i * 3 + 2] = data[pixelIndex + 2] / 255;
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

    const material = new THREE.PointsMaterial({
      size: 0.025, // Augmenté de 0.015 à 0.025
      map: particleTexture,
      vertexColors: true,
      sizeAttenuation: true, 
      transparent: true,
      alphaTest: 0.5,
      blending: THREE.AdditiveBlending, // Ajout du blending additif
      depthWrite: false, // Désactivation de l'écriture de profondeur
    });

    if (particleSystem) {
      const oldPositions = particleSystem.geometry.attributes.position.array;
      const oldColors = particleSystem.geometry.attributes.color.array;

      gsap.to(oldPositions, {
        duration: 1,
        ease: "power2.inOut",
        ...positions,
        onUpdate: () => {
          particleSystem.geometry.attributes.position.needsUpdate = true;
        }
      });

      gsap.to(oldColors, {
        duration: 1,
        ease: "power2.inOut",
        ...colors,
        onUpdate: () => {
          particleSystem.geometry.attributes.color.needsUpdate = true;
        }
      });
    } else {
      particleSystem = new THREE.Points(geometry, material);
      scene.add(particleSystem);
    }

    originalPositions = positions.slice();
    targetPositions = positions.slice();
  });
}

function createCircleTexture() {
  const canvas = document.createElement('canvas');
  canvas.width = 128; // Augmenté de 64 à 128
  canvas.height = 128; // Augmenté de 64 à 128
  const ctx = canvas.getContext('2d');
  
  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const radius = canvas.width / 2;
  
  const gradient = ctx.createRadialGradient(centerX, centerY, 0, centerX, centerY, radius);
  gradient.addColorStop(0, 'rgba(255, 255, 255, 1)');
  gradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.5)');
  gradient.addColorStop(1, 'rgba(255, 255, 255, 0)');
  
  ctx.beginPath();
  ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI, false);
  ctx.fillStyle = gradient;
  ctx.fill();
  
  return new THREE.CanvasTexture(canvas);
}

function animate() {
  requestAnimationFrame(animate);
  if (particleSystem) {
    const positions = particleSystem.geometry.attributes.position.array;
    for (let i = 0; i < positions.length; i += 3) {
      positions[i] += (targetPositions[i] - positions[i]) * 0.1;
      positions[i + 1] += (targetPositions[i + 1] - positions[i + 1]) * 0.1;
      positions[i + 2] += (targetPositions[i + 2] - positions[i + 2]) * 0.1;
    }
    particleSystem.geometry.attributes.position.needsUpdate = true;
  }
  renderer.render(scene, camera);
}

function updateFlagTexture() {
  createParticleSystem();
}

function onMouseMove(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(particleSystem);

  if (intersects.length > 0) {
    const positions = particleSystem.geometry.attributes.position.array;
    const intersection = intersects[0];
    const repelStrength = 1.5;
    const repelRadius = 1;

    for (let i = 0; i < positions.length; i += 3) {
      const dx = positions[i] - intersection.point.x;
      const dy = positions[i + 1] - intersection.point.y;
      const dz = positions[i + 2] - intersection.point.z;
      const distance = Math.sqrt(dx * dx + dy * dy + dz * dz);

      if (distance < repelRadius) {
        const force = (repelRadius - distance) / repelRadius;
        targetPositions[i] = originalPositions[i] + dx * force * repelStrength;
        targetPositions[i + 10] = originalPositions[i + 10] + dy * force * repelStrength;
        targetPositions[i + 20] = originalPositions[i + 20] + dz * force * repelStrength;
      } else {
        targetPositions[i] = originalPositions[i];
        targetPositions[i + 10] = originalPositions[i + 10];
        targetPositions[i + 20] = originalPositions[i + 20];
      }
    }
  }
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
</script>

<style scoped>
.flag-display {
  width: 100%;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
}
</style>

