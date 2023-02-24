
<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshBasicMaterial, Mesh, BoxGeometry, SphereGeometry } from 'three'

const threeRef = ref<HTMLDivElement>()

function init() {
  const scene = new Scene()
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const planeGeo = new PlaneGeometry(60, 20)
  const planeMaterial = new MeshBasicMaterial({ color: 0xaaaaaa })
  const plane = new Mesh(planeGeo, planeMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(15, 0, 0)
  scene.add(plane)

  // cube
  const cubeGeo = new BoxGeometry(4, 4, 4)
  const cubeMaterial = new MeshBasicMaterial({
    color: 0xff0000,
    wireframe: true
  })
  const cube = new Mesh(cubeGeo, cubeMaterial)
  cube.position.set(-4, 3, 0)
  scene.add(cube)

  const sphereGeo = new SphereGeometry(4, 20, 20)
  const sphereMaterial = new MeshBasicMaterial({
    color: 0x7777ff,
    wireframe: true
  })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  sphere.position.set(20, 4, 2)
  scene.add(sphere)

  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)
  renderer.render(scene, camera)

}

onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
