<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, BoxGeometry, MeshBasicMaterial, Mesh, AmbientLight, MeshNormalMaterial, Object3D, AxesHelper, Vector3, BufferGeometry, Float32BufferAttribute } from 'three'
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const threeRef = ref<HTMLDivElement>()

const scene = new Scene()
const renderer = new WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)
renderer.setClearColor(0x000000)
const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)

const heightData = [10, 6, 15, 18, 3, 5]
const barsRef = ref<Object3D[]>([])

function init() {
  camera.position.set(-30, 20, 20)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const controls = new OrbitControls(camera, renderer.domElement)
  scene.add(controls)

  const alight = new AmbientLight(0xeeeeee)
  scene.add(alight)

  for (let i = 0; i < heightData.length; i++) {
    const geo = new BoxGeometry(2, heightData[i], 2)
    const material = new MeshBasicMaterial({
      color: 'white',
      wireframe: true
    })
    const normalMaterial = new MeshNormalMaterial()
    const bar = SceneUtils.createMultiMaterialObject(geo, [normalMaterial, material])
    bar.position.set(2 + 2 * i, 0, 2 + 2 * i)
    bar.scale.y = 0
    scene.add(bar)
    barsRef.value!.push(bar)
  }

  let sy = 0
  let py = 0
  function syAdd() {
    if (sy >= 1) return
    sy += 0.01
    console.log(sy, py)
  }

  function renderScene() {
    syAdd()
    barsRef.value.map((bar, index) => {
      bar.scale.y = sy
      bar.position.y = heightData[index] * sy / 2
    })
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
