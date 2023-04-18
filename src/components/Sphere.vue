<template>
    <div class="w-full flex flex-col justify-between relative font-mono">
        <canvas ref="canvas" class="w-full h-250px absolute -z-10" />
        <header class="w-full fixed p-10 text-white">
            <nav class=" flex justify-between lg:px-20 sm:gpx-0 md:px-5">
                <NuxtLink to="#">Esfera</NuxtLink>
                <div>
                    <NuxtLink to="#" class="px-10">Projetos</NuxtLink>
                    <NuxtLink to="#">Sobre mim</NuxtLink>
                </div>
            </nav>
        </header>
        <main>
            <h1 class="text-bold text-white text-5xl text-center pt-32">Movimente</h1>
            <div class="w-full flex justify-between px-24">

                <button class="text-purple-300 border-2 p-2 border-solid rounded hidden lg:block md:block"
                    @click="controls.enableZoom = true">Habilitar
                    Zoom (scroll)</button>
                <button class="text-purple-300 border-2 p-2 border-solid rounded hidden lg:block md:block"
                    @click="controls.enablePan = true">Habilitar
                    movimento (botão direito)</button>
            </div>
        </main>
    </div>
</template>

<script setup>
import * as THREE from 'three';
import { useWindowSize, useMouseInElement } from '@vueuse/core'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { gsap } from 'gsap';


//cena
const scene = new THREE.Scene();

//forma
const geometry = new THREE.SphereGeometry();
const material = new THREE.MeshStandardMaterial({
    color: '#00ff83',
    roughness: 0.5
})
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

//luz
const light = new THREE.PointLight(0xffffff, 1, 100);
light.position.set(0, 10, 10);
light.intensity = 1.25;
scene.add(light);


//tamanhos
const { width, height } = useWindowSize();

//camera
const aspectRatio = computed(() => width.value / height.value);
const camera = new THREE.PerspectiveCamera(45, aspectRatio, 0.2, 100);
camera.position.z = 7.5
scene.add(camera);

//render
const canvas = ref(null);
let renderer = new THREE.WebGLRenderer();


//controls
let controls = new OrbitControls(camera, renderer.domElement)

//movimentos
let rgb = [12, 23, 255];
const mouse = reactive(useMouseInElement(canvas));
window.addEventListener('mousemove', (e) => {

    rgb = [
        Math.round((mouse.elementX / width.value) * 255), Math.round((mouse.elementY / height.value) * 255), 150]
    //animação
    let newColor = new THREE.Color(`rgb(${rgb.join(',')})`)

    gsap.to(mesh.material.color, { r: newColor.r, g: newColor.g, b: newColor.b })

})



const loop = () => {
    renderer.render(scene, camera);
    window.requestAnimationFrame(loop);
    controls.update();
};

const updateCamera = () => {
    camera.aspect = aspectRatio.value;
    camera.updateProjectionMatrix();
}

const updateRenderer = () => {
    renderer.setSize(width.value, height.value);
    renderer.render(scene, camera);
}

const setRenderer = () => {
    if (canvas.value) {
        renderer = new THREE.WebGLRenderer({ canvas: canvas.value });
        renderer.setSize(width.value, height.value);
        renderer.setPixelRatio(2);
        controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.enablePan = false;
        controls.enableZoom = false;
        controls.autoRotate = true;
        controls.autoRotateSpeed = 5;
        updateRenderer();
    }
}

watch(aspectRatio, () => {
    updateCamera();
    updateRenderer();
})

onMounted(() => {
    updateCamera();
    setRenderer();
    loop();

    //controles

})

</script>

