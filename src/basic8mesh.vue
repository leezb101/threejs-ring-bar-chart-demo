<template>
  <div ref="statsRef"></div>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { AxesHelper, BoxGeometry, Clock, Mesh, MeshLambertMaterial, PerspectiveCamera, PlaneGeometry, Scene, SpotLight, Vector2, WebGLRenderer } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';
import { GUI } from 'dat.gui';
import Stats from 'stats.js';

const threeRef = ref<HTMLElement>()
const statsRef = ref<HTMLDivElement>()

function init() {
  let controls = {
    positionX: 0,
    positionY: 3,
    positionZ: 0,
    rotationX: 0,
    rotationY: 0,
    rotationZ: 0
  }

  // 帧率统计 0 fps 1 ms 2 mb 3 custom
  function initStats(type?: 0 | 1 | 2 | 3) {
    var panelType = typeof type !== 'undefined' && type && !isNaN(type) ? type : 0
    var stats = new Stats()
    stats.showPanel(panelType)
    document.body.appendChild(stats.dom)
    return stats
  }

  const stats = initStats()
  // 定义场景
  const scene = new Scene()
  // 定义摄像机
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  // 定义渲染器
  const renderer = new WebGLRenderer()
  // 定义场景的颜色为黑色
  renderer.setClearColor(0x000000)
  // 定义场景大小为整个窗口
  renderer.setSize(window.innerWidth, window.innerHeight)
  // 启动渲染阴影效果
  renderer.shadowMap.enabled = true
  // 创建一个粗细为20的坐标轴
  const axes = new AxesHelper(20)
  scene.add(axes)

  // 添加光源
  const spotLight = new SpotLight(0xFFFFFF)
  spotLight.position.set(-40, 40, -15)
  spotLight.castShadow = true
  spotLight.shadow.mapSize = new Vector2(1024, 1024)
  spotLight.shadow.camera.far = 130
  spotLight.shadow.camera.near = 40
  scene.add(spotLight)

  // 创建一个宽为60，高为20的的平面。
  const planeGeometry = new PlaneGeometry(60, 40)
  // 设置平面的材质
  const planeMaterial = new MeshLambertMaterial({
    'color': 0xffffff
  })
  // 赋值到Mesh
  const plane = new Mesh(planeGeometry, planeMaterial)
  // 设置平面位置和旋转
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(15, 0, 0)
  // 设置地面为投影面
  plane.receiveShadow = true
  // 将平面添加到场景中
  scene.add(plane)

  // 创建一个立方体
  const cubeGeometry = new BoxGeometry(4, 4, 4)
  const cubeMaterial = new MeshLambertMaterial({
    'color': 0xFF0000
  })
  const cube = new Mesh(cubeGeometry, cubeMaterial)
  cube.position.set(0, 3, 0)
  // 开启投影
  cube.castShadow = true
  scene.add(cube)

  // GUI
  const gui = new GUI()
  const guiPosition = gui.addFolder('position')
  const contX = guiPosition.add(controls, 'positionX', -10, 10)
  const contY = guiPosition.add(controls, 'positionY', -4, 20)
  const contZ = guiPosition.add(controls, 'positionZ', -10, 10)
  contX.listen()
  contX.onChange(function () {
    cube.position.x = controls.positionX
  })
  contY.listen()
  contY.onChange(function () {
    cube.position.y = controls.positionY
  })
  contZ.listen()
  contZ.onChange(function () {
    cube.position.z = controls.positionZ
  })
  const guiRotation = gui.addFolder('rotation')
  guiRotation.add(controls, 'rotationX', -4, 4)
  guiRotation.add(controls, 'rotationY', -4, 4)
  guiRotation.add(controls, 'rotationZ', -4, 4)

  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const trackballControls = new TrackballControls(camera, renderer.domElement)
  const clock = new Clock()

  function renderScene() {
    stats.update()
    trackballControls.update()
    cube.rotation.x = controls.rotationX
    cube.rotation.y = controls.rotationY
    cube.rotation.z = controls.rotationZ
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
