<template>
  <div ref="threeRef"></div>
</template>


<script setup lang="ts">
import { AxesHelper, BoxGeometry, Clock, Color, Mesh, MeshNormalMaterial, PerspectiveCamera, Scene, WebGLRenderer } from 'three';
import { ref, onMounted } from 'vue'
import { Tween, Easing, update } from '@tweenjs/tween.js'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';
import { FlyControls } from 'three/examples/jsm/controls/FlyControls'

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

  const cubeGeo = new BoxGeometry(10, 10, 10)
  const material = new MeshNormalMaterial()
  const cube = new Mesh(cubeGeo, material)
  cube.position.x = cubePos.x
  scene.add(cube)



  threeRef.value!.appendChild(renderer.domElement)


  // tweenAnimate(cube)

  flyAnimate()

}

function flyAnimate() {
  const flyControl = new FlyControls(camera, renderer.domElement)
  flyControl.movementSpeed = 25
  flyControl.rollSpeed = Math.PI / 24
  flyControl.autoForward = true
  flyControl.dragToLook = true

  const clock = new Clock()
  function renderFlyScene() {
    const detail = clock.getDelta()
    flyControl.update(detail)
    update()
    requestAnimationFrame(flyAnimate)
    renderer.render(scene, camera)
  }

  renderFlyScene()

}

function tweenAnimate(mesh: Mesh) {
  const upDataFn = () => {
    mesh.position.x = cubePos.x
  }

  const tween = new Tween(cubePos)
    .to({ x: 50 }, 4000)
  tween.easing(Easing.Elastic.InOut)
  tween.onUpdate(upDataFn)

  // 衔接返回动画
  const tweenback = new Tween(cubePos)
    .to({ x: 0 }, 4000)
  tweenback.easing(Easing.Elastic.InOut)
  tweenback.onUpdate(upDataFn)
  tween.chain(tweenback)
  tweenback.chain(tween)
  tween.start()

  const track = new TrackballControls(camera, renderer.domElement)
  function renderTrackScene() {
    track.update()
    update()
    requestAnimationFrame(renderTrackScene)
    renderer.render(scene, camera)
  }

  renderTrackScene()
}

onMounted(() => {
  init()
})


</script>

<style scoped>

</style>
