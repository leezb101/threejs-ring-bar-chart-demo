
<template>
  <div ref="statsDivRef"></div>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshLambertMaterial, Mesh, BoxGeometry, SphereGeometry, SpotLight, Vector2, PCFSoftShadowMap, Clock } from 'three'
import Stats from 'stats.js'
import { GUI } from 'dat.gui'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls'

const threeRef = ref<HTMLDivElement>()
const statsDivRef = ref<HTMLDivElement>()

function init() {
  const scene = new Scene()
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const planeGeo = new PlaneGeometry(60, 20)
  const planeMaterial = new MeshLambertMaterial({ color: 0xaaaaaa })
  const plane = new Mesh(planeGeo, planeMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(15, 0, 0)
  plane.receiveShadow = true
  scene.add(plane)

  const spotlight = new SpotLight(0xffffff)
  spotlight.position.set(-40, 40, -15)
  spotlight.castShadow = true
  spotlight.shadow.mapSize = new Vector2(1024, 1024)
  spotlight.shadow.camera.far = 130
  spotlight.shadow.camera.near = 40
  scene.add(spotlight)

  // cube
  const cubeGeo = new BoxGeometry(4, 4, 4)
  const cubeMaterial = new MeshLambertMaterial({
    color: 0xff0000,
  })
  const cube = new Mesh(cubeGeo, cubeMaterial)
  cube.position.set(-4, 3, 0)
  cube.receiveShadow = true
  cube.castShadow = true
  scene.add(cube)

  const sphereGeo = new SphereGeometry(4, 20, 20)
  const sphereMaterial = new MeshLambertMaterial({
    color: 0x7777ff,
  })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  sphere.position.set(20, 4, 2)
  sphere.receiveShadow = true
  sphere.castShadow = true
  scene.add(sphere)

  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = PCFSoftShadowMap

  const trackballControls = new TrackballControls(camera, renderer.domElement)
  const clock = new Clock()

  // dat.gui可以操控的状态值的state
  let controls = {
    rotationSpeed: 0.02,
    bouncingSpeed: 0.03
  }

  const gui = new GUI()
  gui.add(controls, 'rotationSpeed', 0, 0.5)
  gui.add(controls, 'bouncingSpeed', 0, 0.5)
  gui.show()
  const stats = initStats()



  // 帧率动画
  let step = 0
  function renderScene() {
    stats.update()
    trackballControls.update(clock)
    // cube animate
    cube.rotation.x += controls.rotationSpeed
    cube.rotation.y += controls.rotationSpeed
    cube.rotation.z += controls.rotationSpeed

    // sphere animate
    step += controls.bouncingSpeed
    sphere.position.x = 20 + 10 * Math.cos(step)
    sphere.position.y = 2 + 10 * Math.abs(Math.sin(step))

    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }
  renderScene()

  const onResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
  window.addEventListener('resize', onresize, false)

}

// type: 帧率统计 0 fps 1 ms 2 mb 3 custom
function initStats(type?: 0 | 1 | 2) {
  let panelType = typeof type !== 'undefined' && type && !isNaN(type) ? type : 0
  const stats = new Stats()
  stats.showPanel(panelType)
  statsDivRef.value?.appendChild(stats.dom)
  return stats
}



onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
