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
  },
  animate: {
    type: String,
    default: 'none' // 'none', 'in', 'out'
  }
});

const container = ref(null);
let scene, camera, renderer, flag, raycaster, mouse;

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
watch(() => props.animate, handleAnimation);

function initScene() {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
  renderer.setSize(container.value.clientWidth, container.value.clientHeight);
  container.value.appendChild(renderer.domElement);

  raycaster = new THREE.Raycaster();
  mouse = new THREE.Vector2();

  const geometry = new THREE.PlaneGeometry(3, 2, 100, 100); // Augmenté à 100x100 pour plus de fluidité
  const material = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      flagTexture: { value: new THREE.TextureLoader().load(`https://flagcdn.com/w1280/${props.countryCode.toLowerCase()}.png`) },
      mousePos: { value: new THREE.Vector2(0.5, 0.5) },
      hoverAmplitude: { value: 0.15 }, // Augmenté de 0.1 à 0.15
      waveAmplitude: { value: 0.08 }, // Augmenté de 0.05 à 0.08
    },
    vertexShader: `
      uniform float time;
      uniform vec2 mousePos;
      uniform float hoverAmplitude;
      uniform float waveAmplitude;
      varying vec2 vUv;
      
      void main() {
        vUv = uv;
        vec3 pos = position;
        float dist = distance(uv, mousePos);
        
        // Ondulation générale plus fluide
        float wave = sin(uv.x * 8.0 + time * 0.8) * waveAmplitude * (1.0 - uv.x);
        wave += cos(uv.y * 6.0 + time * 0.6) * waveAmplitude * 0.7;
        
        // Effet de hover plus prononcé
        float hover = smoothstep(0.4, 0.0, dist) * hoverAmplitude;
        
        pos.z += wave + hover * sin(dist * 8.0 - time * 1.5);
        pos.y += hover * cos(dist * 6.0 - time * 1.2) * 0.3;
        
        gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
      }
    `,
    fragmentShader: `
      uniform sampler2D flagTexture;
      uniform float time;
      varying vec2 vUv;
      
      void main() {
        vec4 texColor = texture2D(flagTexture, vUv);
        
        // Ajout d'un léger effet de grain
        float noise = fract(sin(dot(vUv, vec2(12.9898, 78.233)) * time) * 43758.5453);
        texColor.rgb += (noise - 0.5) * 0.05;
        
        gl_FragColor = texColor;
      }
    `,
    side: THREE.DoubleSide,
    transparent: true,
  });

  flag = new THREE.Mesh(geometry, material);
  flag.scale.set(0.001, 0.001, 0.001); // Commencer avec une échelle très petite
  flag.position.z = -1; // Positionner le drapeau en arrière
  scene.add(flag);

  camera.position.z = 2.5; // Ajusté pour une meilleure vue
  
  // Appeler onWindowResize une fois pour initialiser correctement la taille
  onWindowResize();
}

function animate() {
  requestAnimationFrame(animate);
  if (flag) {
    flag.material.uniforms.time.value += 0.05;
  }
  renderer.render(scene, camera);
}

function updateFlagTexture() {
  if (flag) {
    new THREE.TextureLoader().load(
      `https://flagcdn.com/w1280/${props.countryCode.toLowerCase()}.png`,
      (texture) => {
        flag.material.uniforms.flagTexture.value = texture;
      },
      undefined,
      (err) => {
        console.error('Erreur de chargement de la texture:', err);
      }
    );
  }
}

function onMouseMove(event) {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObject(flag);

  if (intersects.length > 0) {
    flag.material.uniforms.mousePos.value.copy(intersects[0].uv);
  } else {
    flag.material.uniforms.mousePos.value.set(0.5, 0.5);
  }
}

function onWindowResize() {
  const width = container.value.clientWidth;
  const height = container.value.clientHeight;
  
  camera.aspect = width / height;
  camera.updateProjectionMatrix();
  renderer.setSize(width, height);

  // Ajuster la taille du drapeau en fonction de l'aspect ratio
  const aspect = width / height;
  const scale = Math.min(1, aspect / 1.5);
  flag.scale.set(scale, scale, scale);
}

function handleAnimation(newValue) {
  if (newValue === 'in') {
    animateIn();
  } else if (newValue === 'out') {
    animateOut();
  }
}

function animateIn() {
  gsap.to(flag.scale, {
    x: 1,
    y: 1,
    z: 1,
    duration: 1.5,
    ease: "elastic.out(1, 0.6)",
    onComplete: () => {
      flag.material.uniforms.hoverAmplitude.value = 0.1;
    }
  });
  gsap.to(flag.position, {
    z: 0,
    duration: 1.5,
    ease: "power2.out"
  });
}

function animateOut() {
  gsap.to(flag.scale, {
    x: 0.001,
    y: 0.001,
    z: 0.001,
    duration: 1,
    ease: "power2.in"
  });
  gsap.to(flag.position, {
    z: 1,
    duration: 1,
    ease: "power2.in",
    onComplete: () => {
      flag.material.uniforms.hoverAmplitude.value = 0;
    }
  });
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
  overflow: hidden;
}
</style>

