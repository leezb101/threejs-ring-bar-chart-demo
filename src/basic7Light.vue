<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { AmbientLight, BoxGeometry, FogExp2, Mesh, MeshLambertMaterial, PCFSoftShadowMap, PerspectiveCamera, PlaneGeometry, Scene, SpotLight, WebGLRenderer, Clock } from 'three';
import { ref, onMounted } from 'vue'
import { GUI } from 'dat.gui'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';

const threeRef = ref<HTMLElement>()
const guiRef = ref<GUI>()

function init() {
  const scene = new Scene()
  scene.fog = new FogExp2(0xffffff, 0.01)
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  scene.add(camera)

  // spotlight
  const spotLight = new SpotLight(0xFFFFFF, 1.2, 150, 120)
  spotLight.position.set(-40, 60, -10)
  spotLight.castShadow = true
  scene.add(spotLight)

  // 环境光
  const ambientLight = new AmbientLight(0x3c3c3c)
  scene.add(ambientLight)

  // 平面
  const planeGeo = new PlaneGeometry(60, 40, 1, 1)
  const planeMaterial = new MeshLambertMaterial({ color: 0xffffff })
  const plane = new Mesh(planeGeo, planeMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(0, 0, 0)

  plane.receiveShadow = true
  scene.add(plane)

  const controls = {
    rotationSpeed: 0.02,
    numberOfObjects: 0,
    addCube: function () {
      const cubeSize = Math.ceil(Math.random() * 3)
      const cubeGeo = new BoxGeometry(cubeSize, cubeSize, cubeSize)
      const cubeMaterial = new MeshLambertMaterial(
        { color: Math.random() * 0xffffff }
      )
      const cube = new Mesh(cubeGeo, cubeMaterial)
      cube.castShadow = true
      cube.name = `cube-${scene.children.length}`
      cube.position.x = Math.round(Math.random() * planeGeo.parameters.width / 2)
      cube.position.y = Math.round(Math.random() * 5)
      cube.position.z = Math.round(Math.random() * planeGeo.parameters.height / 2)
      scene.add(cube)
      this.numberOfObjects = scene.children.length
    },
    removeCube: function () {
      const allChildren = scene.children
      const lastObj = allChildren[allChildren.length - 1]
      if (lastObj instanceof Mesh && lastObj !== plane) {
        scene.remove(lastObj)
        this.numberOfObjects = scene.children.length
      }
    },
    outputObjects: function () {
      console.log(scene.children)
    }
  }

  const gui = new GUI()
  gui.add(controls, 'rotationSpeed', 0, 0.5)
  gui.add(controls, 'addCube')
  gui.add(controls, 'removeCube')
  gui.add(controls, 'numberOfObjects').listen()
  gui.add(controls, 'outputObjects')
  guiRef.value = gui

  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(0x000000)
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = PCFSoftShadowMap
  threeRef.value!.appendChild(renderer.domElement)

  const trackballControls = new TrackballControls(camera, renderer.domElement)
  const clock = new Clock()

  function renderScene() {
    trackballControls.update(clock)
    // 遍历场景中的所有子对象
    scene.traverse(obj => {
      if (obj instanceof Mesh && obj !== plane) {
        obj.rotation.x += controls.rotationSpeed
        obj.rotation.y += controls.rotationSpeed
        obj.rotation.z += controls.rotationSpeed
      }
    })
    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }
  renderScene()

  const onResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
  window.addEventListener('resize', onResize, false)
}

onMounted(() => {
  init()
})


</script>

<style scoped>

</style>
