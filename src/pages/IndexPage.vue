<template>
  <div ref="container"></div>
  <CreditsArtists />
</template>
<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { ref, onMounted, onUnmounted } from 'vue';
import SunCalc from 'suncalc';
import CreditsArtists from 'src/components/creditsArtists.vue';

let camera, scene, renderer, controls, earth, sun, moon, earthOrbit;

// const scale = 1000;
const container = ref(null);
const loader = new GLTFLoader();

onMounted(() => {
  // Cria a câmera
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.set(50, 50, 50);
  camera.lookAt(0, 0, 0);

  // Cria a cena
  scene = new THREE.Scene();

  // Cria o renderizador
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;

  // Cria os controles de órbita
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.rotateSpeed = 0.5;
  controls.maxDistance = 400;

  loader.load('/images/sun/sun.gltf', (gltf) => {
    sun = gltf.scene;
    scene.add(sun);

    const sunLight = new THREE.PointLight(0xffffff, 1, 1000);
    sunLight.position.set(0, 0, 0);
    sun.add(sunLight);

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    loader.load('/images/earth/earth.gltf', (gltf) => {
      earth = gltf.scene;
      earth.scale.setScalar(0.01);
      earth.position.set(150, 0, 0);
      sun.add(earth);

      // Cria uma mesh que representa a órbita da Terra
      earthOrbit = new THREE.Line(
        new THREE.CircleGeometry(),
        new THREE.LineBasicMaterial({ color: 0xffffff })
      );
      earthOrbit.position.set(0, 0, 0);
      earthOrbit.rotation.x = -Math.PI / 2;
      scene.add(earthOrbit);

      loader.load('/images/moon/moon.gltf', (gltf) => {
        moon = gltf.scene;
        moon.scale.setScalar(0.1);
        moon.position.set(130, 0, 0);
        if (earth) {
          earth.add(moon);
        } else {
          console.error('Erro: modelo da Terra não foi carregado');
          return;
        }
        console.log(moon);
      });
    });

    const sphereGeometry = new THREE.SphereGeometry(550, 60, 40);
    const texture = new THREE.TextureLoader().load('/images/bk2.jpg');
    const sphereMaterial = new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.DoubleSide,
    });
    const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
    scene.add(sphere);
  });

  container.value.appendChild(renderer.domElement);

  animate();
});

onUnmounted(() => {
  renderer.dispose();
});

function animate() {
  requestAnimationFrame(animate);

  updateEarthPosition();

  controls.update();
  renderer.render(scene, camera);
}
function updateEarthPosition() {
  if (!sun) return;
  if (!earth) return;

  const date = new Date();
  const { azimuth, altitude } = SunCalc.getPosition(date, 0, 0);
  const distance = 150 * Math.cos(altitude);

  earth.position.setFromSphericalCoords(
    distance,
    azimuth - Math.PI / 2,
    altitude
  );

  earthOrbit.position.copy(sun.position);
  earthOrbit.lookAt(earth.position);

  if (earth.children[0]) {
    const moonDistance = 30;

    const date = new Date();
    const { azimuth, altitude } = SunCalc.getPosition(
      date,
      earth.position.x,
      earth.position.y
    );
    const moonPosition = new THREE.Vector3().setFromSphericalCoords(
      moonDistance,
      azimuth - Math.PI / 2,
      altitude
    );

    earth.children[0].position.copy(moonPosition);

    console.log('sun', sun.position);
    console.log('moon', moon.position);
    console.log('earth', earth.position);
  }
}
</script>

<style></style>
