<template>
  <div>
    <div ref="container" style="width: 100vw; height: 100vh"></div>
    <div
      style="
        position: absolute;
        top: 10px;
        left: 10px;
        background: rgba(255, 255, 255, 0.8);
        padding: 10px;
      "
    >
      <label>
        Ширина двери:
        <input type="range" v-model="doorWidth" min="1" max="5" step="0.1" />
      </label>
      <label>
        Высота двери:
        <input type="range" v-model="doorHeight" min="2" max="6" step="0.1" />
      </label>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import woodTexture from "../assets/wood_texture.jpg";

export default defineComponent({
  setup() {
    const container = ref<HTMLDivElement | null>(null);
    const doorWidth = ref(2);
    const doorHeight = ref(4);

    let scene: THREE.Scene,
      camera: THREE.PerspectiveCamera,
      renderer: THREE.WebGLRenderer;
    let door: THREE.Mesh;
    let controls: OrbitControls;
    let texture: THREE.Texture;

    const initScene = () => {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0xaaaaaa);

      camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        100
      );
      camera.position.set(5, 5, 10);

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      container.value?.appendChild(renderer.domElement);

      controls = new OrbitControls(camera, renderer.domElement);

      // Свет
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(5, 10, 5);
      directionalLight.castShadow = true;
      scene.add(directionalLight);

      // Дополнительный свет
      const pointLight = new THREE.PointLight(0xffffff, 1, 50);
      pointLight.position.set(0, 5, 5);
      scene.add(pointLight);

      // Плоскость (пол)
      const planeGeometry = new THREE.PlaneGeometry(20, 20);
      const planeMaterial = new THREE.MeshStandardMaterial({ color: 0x808080 });
      const plane = new THREE.Mesh(planeGeometry, planeMaterial);
      plane.rotation.x = -Math.PI / 2;
      plane.receiveShadow = true;
      scene.add(plane);

      // Дверь
      const textureLoader = new THREE.TextureLoader();
      texture = textureLoader.load(
        woodTexture,
        (tex) => {
          tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
          tex.repeat.set(doorWidth.value, doorHeight.value);
        },
        undefined,
        (err) => {
          console.error("Ошибка загрузки текстуры:", err);
        }
      );

      const woodMaterial = new THREE.MeshStandardMaterial({
        map: texture,
        emissive: 0x333333,
        emissiveIntensity: 0.5,
      });

      door = new THREE.Mesh(
        new THREE.BoxGeometry(doorWidth.value, doorHeight.value, 0.2),
        woodMaterial
      );
      door.position.set(0, doorHeight.value / 2, -2);
      door.castShadow = true;
      scene.add(door);

      // Куб
      const cubeGeometry = new THREE.BoxGeometry(2, 2, 2);
      const cubeMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });
      const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
      cube.position.set(-3, 1, 0);
      cube.castShadow = true;
      scene.add(cube);

      // Сфера
      const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
      const sphereMaterial = new THREE.MeshStandardMaterial({
        color: 0x0000ff,
      });
      const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
      sphere.position.set(3, 1, 0);
      sphere.castShadow = true;
      scene.add(sphere);

      animate();
    };

    const animate = () => {
      requestAnimationFrame(animate);
      controls.update();
      renderer.render(scene, camera);
    };

    watch([doorWidth, doorHeight], () => {
      door.geometry.dispose();
      door.geometry = new THREE.BoxGeometry(
        doorWidth.value,
        doorHeight.value,
        0.2
      );
      door.position.y = doorHeight.value / 2;

      // Обновляем текстурные повторения
      texture.repeat.set(doorWidth.value, doorHeight.value);
    });

    onMounted(initScene);

    return { container, doorWidth, doorHeight };
  },
});
</script>

<style>
/* Стили для обеспечения полного отображения сцены */
</style>
