<template>
  <div ref="threeRef"></div>
</template>


<script setup lang="ts">
import { AmbientLight, AnimationMixer, AxesHelper, BoxGeometry, BufferAttribute, Clock, Color, InterleavedBufferAttribute, Mesh, MeshLambertMaterial, MeshNormalMaterial, PerspectiveCamera, Scene, WebGLRenderer } from 'three';
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
  const trackballControls = new TrackballControls(camera, renderer.domElement)
  const clock = new Clock()

  const ambientLight = new AmbientLight(0xffffff)
  scene.add(ambientLight)

  const cubeGeo = new BoxGeometry(2, 2, 2)
  const cubeMaterial = new MeshLambertMaterial({
    color: 0xd43335
  })
  // 存储了几何体morphTargets的细节信息
  cubeGeo.morphAttributes['position'] = []
  // 变形目标1: 尺寸为 2,20,2 的立方体
  const cubeGeo1 = new BoxGeometry(2, 20, 2)
  cubeGeo.morphAttributes['position'][0] = cubeGeo1.attributes['position'] as BufferAttribute | InterleavedBufferAttribute

  const meshCube = new Mesh(cubeGeo, cubeMaterial)
  scene.add(meshCube)

  let index = 0
  let flag = true
  function renderScene() {
    if (flag) {
      index += 0.01
      if (index > 1) {
        flag = false
      }
    }
    if (!flag) {
      index -= 0.01
      if (index <= 0) {
        flag = true
      }
    }
    meshCube.morphTargetInfluences![0] = index
    trackballControls.update()
    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }

  renderScene()

}


onMounted(() => {
  init()
})


</script>

<style scoped>

</style>
