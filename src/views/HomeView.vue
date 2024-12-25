<script setup>
import * as THREE from 'three'
import { ref, onMounted, computed, watch } from 'vue'
import { useWindowSize } from '@vueuse/core';
import { useTweakPane } from '../composables/useTweakPane.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

let renderer, camera, controls;

const canvas = ref(null)
const scene = new THREE.Scene()
const { width, height } = useWindowSize()

const aspectRatio = computed(() => width.value / height.value)
const updateRenderer = () => {
  renderer.setSize(width.value, height.value)
  renderer.setPixelRatio(window.devicePixelRatio)
}

const updateCamera = () => {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

watch(aspectRatio, updateRenderer)
watch(aspectRatio, updateCamera)

camera = new THREE.PerspectiveCamera(
  45, 
  aspectRatio.value, 
  0.1, 
  1000  
)

camera.updateProjectionMatrix()

camera.position.x = 0
camera.position.z = 5

scene.add(camera)

const cube = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1, 32, 32),
  new THREE.MeshBasicMaterial({
    color: 0x008080
  })
)
scene.add(cube)

const { pane, fpsGraph } = useTweakPane()
const loop = () => {
  fpsGraph.begin()
  renderer.render(scene, camera)
  controls.update()
  fpsGraph.end()

  requestAnimationFrame(loop)
}
onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    antialias: true
  })
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  updateRenderer()
  updateCamera()

  loop()
})
</script>

<template>
  <div>
    <canvas ref="canvas"/>
  </div>
</template>

<style>

</style>