<template>
  <div ref="statsDivRef" class="stats"></div>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { AmbientLight, AxesHelper, BoxGeometry, Color, Mesh, MeshLambertMaterial, PCFSoftShadowMap, PerspectiveCamera, PlaneGeometry, Scene, SphereGeometry, SpotLight, WebGLRenderer } from 'three';
import { ref, onMounted } from 'vue'
import Stats from 'stats.js'
import dat from 'dat.gui';

const statsDivRef = ref<HTMLDivElement>()
const threeRef = ref<HTMLElement>()
const statsRef = ref<Stats>()

function initStats() {
  statsRef.value = new Stats()
  statsRef.value?.showPanel(1)
  statsDivRef.value?.appendChild(statsRef.value.dom)
}

function init() {
  const scene = new Scene()
  const axes = new AxesHelper(20)
  scene.add(axes)

  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(new Color(0xEEEEEE))
  renderer.setSize(window.innerWidth, window.innerHeight)


  const planeGeo = new PlaneGeometry(100, 100)
  const planeMaterial = new MeshLambertMaterial({ color: 0xffffff })
  const plane = new Mesh(planeGeo, planeMaterial)
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(0, 0, 0)
  // 地面接受阴影
  plane.receiveShadow = true
  scene.add(plane)

  // 盒子
  const cubeGeo = new BoxGeometry(4, 4, 14)
  const cubeMaterial = new MeshLambertMaterial({ color: 0xff0000 })
  const cube = new Mesh(cubeGeo, cubeMaterial)
  // 开启阴影
  cube.castShadow = true
  cube.position.set(14, 2, 2)
  scene.add(cube)

  // 球
  const sphereGeo = new SphereGeometry(4, 20, 20)
  const sphereMaterial = new MeshLambertMaterial({ color: 0x7777ff })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  sphere.castShadow = true
  sphere.position.set(2, 2, 2)
  scene.add(sphere)

  // 光源
  const spotLight = new SpotLight(0xffffff)
  spotLight.position.set(-40, 60, -40)
  // 开启阴影
  spotLight.castShadow = true
  scene.add(spotLight)
  const ambientLight = new AmbientLight(0xcccccc)

  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(-50, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = PCFSoftShadowMap
  // renderer.render(scene, camera)
  let step = 0

  const controls = {
    rotationSpeed: 0.02,
    bouncingSpeed: 0.03,
    addCube: function () {
      const cubeGeo = new BoxGeometry(4, 4, 4)
      const cubeMaterial = new MeshLambertMaterial({ color: 0xff0000 })
      const cube = new Mesh(cubeGeo, cubeMaterial)
      cube.castShadow = true
      cube.position.x = -30 + Math.round(Math.random() * planeGeo.parameters.width)
      cube.position.y = Math.round(Math.random() * 5)
      cube.position.z = -20 + Math.round(Math.random() * planeGeo.parameters.height)
      cube.name = `cube-${scene.children.length}`
      scene.add(cube)
      this.numberOfObjects = scene.children.length
    },
    removeCube: function () {
      // 过滤非mesh对象
      const meshlist = scene.children.filter((o) => {
        if (o instanceof Mesh && o.name.startsWith('cube-')) {
          return true
        }
        return false
      })
      scene.remove(meshlist[meshlist.length - 1])
      this.numberOfObjects = scene.children.length
    },
    numberOfObjects: scene.children.length
  }

  const gui = new dat.GUI()
  gui.add(controls, 'rotationSpeed', 0, 0.05)
  gui.add(controls, 'bouncingSpeed', 0, 0.05)
  gui.add(controls, 'addCube')
  gui.add(controls, 'removeCube')
  // 监听
  gui.add(controls, 'numberOfObjects').listen()
  rendererScene()

  function rendererScene() {
    statsRef.value?.begin()

    scene.traverse((e) => {
      if (e instanceof Mesh && e != plane) {
        e.rotation.x += controls.rotationSpeed
        e.rotation.y += controls.rotationSpeed
        e.rotation.z += controls.rotationSpeed
      }
    })

    cube.rotation.x += 0.02
    cube.rotation.y += 0.02
    cube.rotation.z += 0.02
    step += 0.02
    sphere.position.x = 20 + 10 * Math.cos(step)
    sphere.position.y = 2 + 10 * Math.abs(Math.cos(step))

    requestAnimationFrame(rendererScene)
    renderer.render(scene, camera)
    statsRef.value?.end()
  }
}

function clear() {
  const dg = document.querySelector('.dg')
  if (dg) {
    dg.innerHTML = ''
  }
}

onMounted(() => {
  clear()
  init()
  initStats()
})

onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
