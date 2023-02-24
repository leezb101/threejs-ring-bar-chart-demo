<template>
  <div ref="threeRef">
    <div class="legend">
      <div v-for="(item, i) in ringData" class="legend-item">
        <span></span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, BoxGeometry, MeshBasicMaterial, Mesh, AmbientLight, MeshNormalMaterial, Object3D, AxesHelper, Vector3, BufferGeometry, Float32BufferAttribute, Shape, ExtrudeGeometryOptions, ExtrudeGeometry, MeshPhongMaterial, Group, PointLight, MeshLambertMaterial, DoubleSide, Raycaster, Vector2, Color, Material } from 'three'
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass'
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass'
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader'


const threeRef = ref<HTMLDivElement>()

const scene = new Scene()
const renderer = new WebGLRenderer({ antialias: true })
renderer.setSize(window.innerWidth, window.innerHeight)
renderer.setClearColor(0x121927)
const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)

const ringData = [10, 30, 20, 25, 15]
const ringsRef = ref<Object3D[]>([])

const colors = ['#5B8FF9', '#E5679A', '#44D7B6', '#F7B500', '#8F84E0', '#FF8362', '#6FD8D6', '#E55050']
const highLightcolors = ['#7EA8FF', '#FF96C1', '#80E5CE', '#FFD338', '#B1A7FD', '#FFAB94', '#99F7F5', '#FF6E6E']

let INTERSECTED: any
const raycaster = new Raycaster()
const mousePoint = new Vector2()

let composer: EffectComposer, outlinePass: OutlinePass, effectFXAA: ShaderPass, shaderPass: ShaderPass
let selectedObjects = [] as any[]

function init() {

  camera.position.set(-35, 20, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableZoom = false
  controls.enablePan = false

  const alight = new AmbientLight(0xffffff, 0.5)
  scene.add(alight)

  const plight = new PointLight(0xeeeeee, 0.8)
  plight.position.set(-20, 30, 40)
  scene.add(plight)

  addRingPart()
  addGround()

  composer = new EffectComposer(renderer)
  const renderPass = new RenderPass(scene, camera)
  composer.addPass(renderPass)

  outlinePass = new OutlinePass(new Vector2(window.innerWidth, window.innerHeight), scene, camera)
  composer.addPass(outlinePass)
  outlinePass.visibleEdgeColor = new Color(0xffaa22)
  outlinePass.edgeStrength = 10;//边缘强度
  outlinePass.edgeGlow = 0.5;//缓缓接近
  outlinePass.edgeThickness = 1;//边缘厚度
  outlinePass.pulsePeriod = 1; //脉冲周期

  effectFXAA = new ShaderPass(FXAAShader)
  effectFXAA.uniforms['resolution'].value.set(1 / window.innerWidth, 1 / window.innerHeight)
  composer.addPass(effectFXAA)


  document.addEventListener('mousemove', onMouseMove)

  let sy = 0
  function syAdd() {
    if (sy >= 1) return
    sy += 0.02
  }

  function renderScene() {
    syAdd()
    ringsRef.value.map((r, idx) => {
      r.scale.z = sy
    })

    camera.updateMatrixWorld()
    requestAnimationFrame(renderScene)
    // renderer.render(scene, camera)
    composer.render()
  }

  renderScene()
}

function addRingPart() {
  const innerR = 5
  const outR = 8
  const total = ringData.reduce((pre, cur) => cur + pre, 0)
  const group = new Group()

  let deltaRotate = 0

  const sortedRingData = ringData.sort((a, b) => a - b)

  for (let i = 0; i < sortedRingData.length; i++) {
    const partShape = new Shape()
    const percent = sortedRingData[i] / total
    partShape.moveTo(innerR, 0)
    partShape.absarc(0, 0, innerR, 0, percent * 2 * Math.PI, false)
    partShape.lineTo(outR * Math.cos(percent * 2 * Math.PI), outR * Math.sin(percent * 2 * Math.PI))
    partShape.absarc(0, 0, outR, percent * 2 * Math.PI, 0, true)
    partShape.lineTo(innerR, 0)
    const extrudeOptions: ExtrudeGeometryOptions = {
      depth: 10 * (sortedRingData[i] / total),
      curveSegments: 30,
      steps: 4,
      bevelEnabled: false,
      bevelSegments: 0,
      bevelOffset: 0,
      bevelSize: 0,
      bevelThickness: 0
    }
    const partGeo = new ExtrudeGeometry(partShape, extrudeOptions)
    const material = new MeshPhongMaterial({
      color: colors[i],
      opacity: 1,
      transparent: true,
      shininess: 1,
      side: DoubleSide,
      emissive: 'white',
      emissiveIntensity: .2
    })

    const mesh = new Mesh(partGeo, material)
    mesh.name = `ring-${i}`
    mesh.rotation.z = Math.PI * 2 * deltaRotate
    deltaRotate += sortedRingData[i] / total
    ringsRef.value.push(mesh)
    group.add(mesh)
  }

  group.rotation.x = - Math.PI / 2
  scene.add(group)
}

function addGround() {
  const innerR = 5
  const outR = 8
  const arcShape = new Shape()
  arcShape.moveTo(outR, 0)
  arcShape.lineTo(innerR, 0)
  arcShape.absarc(0, 0, innerR + 2, 0, Math.PI * 2, false)
  arcShape.absarc(0, 0, outR + 1, 0, Math.PI * 2, true)

  const extrudeOptions: ExtrudeGeometryOptions = {
    curveSegments: 40, depth: 0.5, bevelEnabled: false, bevelSegments: 0, steps: 1, bevelSize: 0, bevelThickness: 0
  }
  const groundGeo = new ExtrudeGeometry(arcShape, extrudeOptions)
  const material = new MeshPhongMaterial({
    color: 0xDDE1D2,
    opacity: 0.6,
    transparent: true
  })
  const ground = new Mesh(groundGeo, material)
  ground.name = 'ground'
  scene.add(ground)
  ground.rotation.x = - Math.PI / 2
}

const onMouseMove = (event: MouseEvent) => {
  mousePoint.x = (event.clientX / window.innerWidth) * 2 - 1
  mousePoint.y = -(event.clientY / window.innerHeight) * 2 + 1
  checkIntersection()
}

function checkIntersection() {
  raycaster.setFromCamera(mousePoint, camera)
  const intersects = raycaster.intersectObjects(scene.children, true)
  if (intersects.length) {
    if (INTERSECTED != intersects[0].object && intersects[0].object instanceof Mesh) {
      if (INTERSECTED) {
        INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex)
      }
      if (intersects[0].object.name === 'ground') return
      INTERSECTED = intersects[0].object
      const idx = Number((INTERSECTED.name as string).split('-')[1])
      debugger
      highlightMesh(idx)
      addSelection(intersects[0].object)
      outlinePass.selectedObjects = selectedObjects
    }
  } else {
    if (INTERSECTED) INTERSECTED.material.emissive.setHex(INTERSECTED.currentHex)
    INTERSECTED = null
    addSelection(null)
    outlinePass.selectedObjects = []
  }

}

function highlightMesh(idx: number) {
  // obj.currentHex = obj.material.emissive.getHex()
  // obj.material.emissive.setHex(0x999999)
  ringsRef.value.forEach((item, i) => {
    if (item.name && item.name == `ring-${idx}`) {
      const material = new MeshPhongMaterial({
        color: highLightcolors[idx],
        opacity: 1,
        transparent: true,
        shininess: 1,
        side: DoubleSide,
        emissive: 'white',
        emissiveIntensity: .2
      })
      item.material = material
    } else {
      const material = new MeshPhongMaterial({
        color: colors[i],
        opacity: 1,
        transparent: true,
        shininess: 1,
        side: DoubleSide,
        emissive: 'white',
        emissiveIntensity: .2
      })
      item.material = material
    }
  })


  // const material = new MeshPhongMaterial({
  //   color: 0xffffff * Math.random(),
  //   opacity: 1,
  //   transparent: true
  // })
  // obj.material = material
}

function addSelection(obj: Object3D | Mesh | null) {
  if (obj) {
    selectedObjects = []
    selectedObjects.push(obj)
  } else {
    selectedObjects = []
  }
}

onMounted(() => {
  init()
})

</script>

<style scoped>
</style>
