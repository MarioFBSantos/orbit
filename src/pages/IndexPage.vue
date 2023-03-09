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

let camera, scene, renderer, controls, earth, sun, moon, earthOrbit, moonOrbit;

const container = ref(null);
const loader = new GLTFLoader();

onMounted(() => {
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.set(50, 50, 50);
  camera.lookAt(0, 0, 0);

  scene = new THREE.Scene();

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;

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
        moon.position.set(30, 0, 0);
        moonOrbit = new THREE.Object3D();
        moonOrbit.position.set(0, 0, 0);
        moonOrbit.rotation.x = -Math.PI / 2;
        moonOrbit.add(moon);
        earth.add(moonOrbit);
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
  if (!moon) return;

  const date = new Date();

  const pos = SunCalc.getPosition(date, 0, 0);
  const dist = 150;
  const phi = ((90 - (pos.altitude * 180) / Math.PI) * Math.PI) / 180;
  const theta = (((pos.azimuth * 180) / Math.PI - 90) * Math.PI) / 180;
  const x = dist * Math.sin(phi) * Math.cos(theta);
  const y = dist * Math.sin(phi) * Math.sin(theta);
  const z = dist * Math.cos(phi);

  earth.position.set(x, y, z);

  earthOrbit.position.copy(sun.position);
  earthOrbit.lookAt(earth.position);

  const moonPos = SunCalc.getMoonPosition(
    date,
    earth.position.x,
    earth.position.y,
    earth.position.z
  );
  const moonDist = 80;
  const moonPhi = ((90 - (moonPos.altitude * 180) / Math.PI) * Math.PI) / 180;
  const moonTheta = (((moonPos.azimuth * 180) / Math.PI - 90) * Math.PI) / 180;
  const moonX = moonDist * Math.sin(moonPhi) * Math.cos(moonTheta);
  const moonY = moonDist * Math.sin(moonPhi) * Math.sin(moonTheta);
  const moonZ = moonDist * Math.cos(moonPhi);

  moon.position.set(moonX, moonY, moonZ);

  moonOrbit.position.copy(earth.position);
  moonOrbit.rotation.x = -Math.PI / 2;
  earth.add(moonOrbit);

  moonOrbit.position.copy(earth.position);
  moonOrbit.lookAt(moon.position);
}
</script>

<style></style>
