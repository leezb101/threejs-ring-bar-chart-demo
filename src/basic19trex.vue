<template>
  <div ref="threeRef"></div>
</template>


<script setup lang="ts">
import { AnimationMixer, AxesHelper, BoxGeometry, Clock, Color, Mesh, MeshNormalMaterial, PerspectiveCamera, Scene, WebGLRenderer } from 'three';
import { ref, onMounted } from 'vue'
import { Tween, Easing, update } from '@tweenjs/tween.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';

const threeRef = ref<HTMLDivElement>()
const scene = new Scene()
const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 500)
const renderer = new WebGLRenderer({})
renderer.setClearColor(new Color(0x000000))
renderer.setSize(window.innerWidth, window.innerHeight)

const cubePos = { x: 0 }

function init() {
  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)

  const axes = new AxesHelper(20)
  scene.add(axes)

  threeRef.value!.appendChild(renderer.domElement)

  let mixer: AnimationMixer | null = null
  const loader = new GLTFLoader()
  const trexPath = new URL('./assets/trex.glb', import.meta.url).href
  loader.load(trexPath, (trex) => {
    mixer = new AnimationMixer(trex.scene.children[0])
    debugger
    const test = trex.animations[0]
    mixer.clipAction(test).setDuration(10).play()
    scene.add(trex.scene)
  })
  const trackballControls = new TrackballControls(camera, renderer.domElement)
  const clock = new Clock()

  function renderScene() {
    const delta = clock.getDelta()

    if (mixer) {
      mixer.update(delta)
    }
    trackballControls.update()
    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }

}


onMounted(() => {
  init()
})


</script>

<style scoped>

</style>
