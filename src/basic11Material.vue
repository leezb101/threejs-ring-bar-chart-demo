
<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshBasicMaterial, Mesh, BoxGeometry, SphereGeometry, MeshDepthMaterial, MultiplyBlending, AmbientLight, MeshLambertMaterial, MeshNormalMaterial, SpotLight, MeshPhongMaterial, MeshStandardMaterial } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls'
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils'

const threeRef = ref<HTMLDivElement>()

function init() {
  const scene = new Scene()
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 200)
  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const planeGeo = new PlaneGeometry(200, 20)
  const planeMaterial = new MeshBasicMaterial({ color: 0xaaaaaa })
  const plane = new Mesh(planeGeo, planeMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(15, 0, 0)
  // scene.add(plane)

  // cube
  // const cubeGeo = new BoxGeometry(4, 4, 4)
  // const cubeMaterial = new MeshBasicMaterial({
  //   color: 0xff0000,
  //   wireframe: true,
  //   wireframeLinewidth: 3
  // })
  // const cube = new Mesh(cubeGeo, cubeMaterial)
  // cube.position.set(-4, 3, 0)
  // scene.add(cube)

  // const cubeGeo2 = new BoxGeometry(4, 4, 4)
  // const cubeMaterial2 = new MeshBasicMaterial({
  //   color: 0x00ff00,
  //   wireframe: true,
  //   wireframeLinewidth: 3
  // })
  // const cube2 = new Mesh(cubeGeo2, cubeMaterial2)
  // cube2.position.set(4, 3, 0)
  // scene.add(cube2)

  // const alight = new AmbientLight(0xeeeeee)
  // scene.add(alight)

  // const cubeGeo3 = new BoxGeometry(4, 4, 4)
  // const cubeMaterial3 = new MeshDepthMaterial()
  // const cube3 = new Mesh(cubeGeo3, cubeMaterial3)
  // cube3.position.set(50, 3, 0)
  // scene.add(cube3)

  const depthMaterial = new MeshDepthMaterial()
  const colorMaterial = new MeshLambertMaterial({
    color: 0xff0000,
    transparent: true,
    blending: MultiplyBlending
  })
  const cubeGeo4 = new BoxGeometry(4, 4, 4)
  const cubeGeo5 = new BoxGeometry(4, 4, 4)
  const cubeGeo6 = new BoxGeometry(4, 4, 4)

  // const cube4 = new SceneUtils.createMultiMaterialObject(cubeGeo4, [depthMaterial, colorMaterial])
  // cube4.position.set(120, 3, 0)
  // scene.add(cube4)
  // const cube5 = new SceneUtils.createMultiMaterialObject(cubeGeo5, [depthMaterial, colorMaterial])
  // cube5.position.set(20, 3, 0)
  // scene.add(cube5)
  // const cube6 = new SceneUtils.createMultiMaterialObject(cubeGeo6, [depthMaterial, colorMaterial])
  // cube6.position.set(60, 3, 0)
  // scene.add(cube6)

  const sphereGeo = new SphereGeometry(4, 20, 20)
  // const sphereMaterial = new MeshBasicMaterial({
  //   color: 0x7777ff,
  //   wireframe: true
  // })
  // const sphereMaterial = new MeshNormalMaterial({ flatShading: true })
  // const sphereMaterial = new MeshLambertMaterial({
  //   // color: 0x7777ff,
  //   emissive: 'yellow',
  //   wireframe: true
  // })
  // const sphereMaterial = new MeshPhongMaterial({ color: 0x7777ff, shininess: 15 })
  const sphereMaterial = new MeshStandardMaterial({ color: 0x7777ff, metalness: 1, roughness: 0 })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  sphere.position.set(0, 0, 0)
  scene.add(sphere)

  const spotLight = new SpotLight(0xffffff, 2)
  spotLight.position.set(-20, 10, 0)
  spotLight.lookAt(-10, 10, 0)
  scene.add(spotLight)


  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const track = new TrackballControls(camera, renderer.domElement)
  function rend() {
    track.update()
    requestAnimationFrame(rend)
    renderer.render(scene, camera)
  }
  rend()
}

onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
