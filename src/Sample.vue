<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshBasicMaterial, Mesh, BoxGeometry, SphereGeometry } from 'three'

const threeRef = ref()

function init() {
  const scene = new Scene()
  const axes = new AxesHelper(20)
  scene.add(axes)

  const renderer = new WebGLRenderer({
    antialias: true
  })
  renderer.setClearColor(new Color(0xEEEEEE))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const planeGeo = new PlaneGeometry(60, 20)
  const planeBasicMaterial = new MeshBasicMaterial({ color: 0xcccccc })
  const plane = new Mesh(planeGeo, planeBasicMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(30, 0, 10)
  scene.add(plane)

  // 盒子
  const cubeGeo = new BoxGeometry(4, 4, 14)
  const cubeMaterial = new MeshBasicMaterial({ color: 0xff0000, wireframe: true })
  const cube = new Mesh(cubeGeo, cubeMaterial)
  cube.position.set(10, 3, 10)
  scene.add(cube)

  // 球形几何体
  const sphereGeo = new SphereGeometry(4, 20, 20)
  const sphereMaterial = new MeshBasicMaterial({ color: 0x7777ff, wireframe: true })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  sphere.position.set(40, 4, 10)
  scene.add(sphere)

  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight)
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
