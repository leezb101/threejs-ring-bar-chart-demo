
<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshBasicMaterial, Mesh, BoxGeometry, SphereGeometry, MeshDepthMaterial, MultiplyBlending, AmbientLight, MeshLambertMaterial, MeshNormalMaterial, SpotLight, MeshPhongMaterial, MeshStandardMaterial, CircleGeometry, RingGeometry, Shape, Vector2, Path, ShapeGeometry, CylinderGeometry, ConeGeometry, DoubleSide, TorusGeometry, TorusKnotGeometry, MeshMatcapMaterial, Vector3, Object3D, LatheGeometry, ExtrudeGeometry } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls'
import { ConvexGeometry } from 'three/examples/jsm/geometries/ConvexGeometry'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils'

const threeRef = ref<HTMLDivElement>()
const scene = new Scene()

function init() {

  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 200)
  const renderer = new WebGLRenderer({ antialias: true })
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  // 平面
  // const planeGeo = new PlaneGeometry(50, 30, 4, 4)
  // const planeMaterial = new MeshBasicMaterial({
  //   wireframe: true,
  //   color: 'yellow'
  // })
  // const plane = new Mesh(planeGeo, planeMaterial)
  // plane.rotation.x = 0.5 * Math.PI
  // scene.add(plane)

  // // 圆
  // const circleGeo = new CircleGeometry(20, 80, 0, Math.PI)
  // const circleMaterial = new MeshBasicMaterial({ color: 'red', wireframe: true })
  // const circle = new Mesh(circleGeo, circleMaterial)
  // // scene.add(circle)

  // // 环
  // const ringGeo = new RingGeometry(10, 20, 50, 30)
  // const ringMaterial = new MeshBasicMaterial({ color: 'green', wireframe: true })
  // const ring = new Mesh(ringGeo, ringMaterial)
  // scene.add(ring)

  // // shape自定义
  // const shape = drawShape()
  // const shapeGeo = new ShapeGeometry(shape)
  // const shapeMaterial = new MeshBasicMaterial({
  //   color: 'pink'
  // })
  // const shapeMesh = new Mesh(shapeGeo, shapeMaterial)
  // scene.add(shapeMesh)

  // addMesh('box')
  // addMesh('sphere')
  // addMesh('cylinder')
  // addMesh('cone')
  // addMesh('torus')
  // addMesh('torusKnot')
  // addMesh('convex')
  // addMesh('lathe')
  // addMesh('extrude')
  addMesh('text')


  const al = new AmbientLight(new Color(0xffffff))
  scene.add(al)

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

function addMesh(type: 'box' | 'sphere' | 'cylinder' | 'cone' | 'torus' | 'torusKnot' | 'convex' | 'lathe' | 'extrude' | 'text') {
  let mesh: Object3D | null
  if (type === 'sphere') {
    mesh = drawSphere()
  }
  if (type === 'cylinder') {
    mesh = drawCylinder()
  }
  if (type === 'cone') { mesh = drawCone() }
  if (type === 'torus') mesh = drawTorus()
  if (type === 'torusKnot') mesh = drawTorusKnot()
  if (type === 'convex') mesh = drawConvex()
  if (type === 'lathe') mesh = drawLathe()
  if (type === 'extrude') mesh = drawExtrude()
  if (type === 'text') {
    drawText()
    mesh = null
  }
  else mesh = drawBox()
  if (mesh) scene.add(mesh)
}

function drawBox() {
  const cubeGeo = new BoxGeometry(10, 10, 20, 4, 4, 4)
  const cubeMaterial = new MeshBasicMaterial({
    color: 'green',
    wireframe: true
  })
  const boxMesh = new Mesh(cubeGeo, cubeMaterial)
  return boxMesh
}

function drawSphere() {
  const sphereGeo = new SphereGeometry(20, 50, 50, -90 / 180 * Math.PI, 270 / 180 * Math.PI, 0, 0.5 * Math.PI)
  const sphereMaterial = new MeshBasicMaterial({
    color: 'red',
    wireframe: true
  })
  const sphere = new Mesh(sphereGeo, sphereMaterial)
  return sphere
}

function drawCylinder() {
  const cyGeo = new CylinderGeometry(8, 4, 15, 30, 10, false, 0, Math.PI * 2)
  const cyMaterial = new MeshBasicMaterial({
    color: 'lightBlue',
    wireframe: true
  })
  const cy = new Mesh(cyGeo, cyMaterial)
  return cy
}

function drawCone() {
  const coneGeo = new ConeGeometry(5, 10, 30, 10, false, 0, 300 / 180 * Math.PI)
  const material = new MeshBasicMaterial({
    color: 'red',
    side: DoubleSide
  })
  const mesh = new Mesh(coneGeo, material)
  return mesh
}

const drawTorus = (): Mesh => {
  const geo = new TorusGeometry(10, 3, 40, 20, 225 / 180 * Math.PI)
  const mt = new MeshBasicMaterial({
    color: 'red',
    wireframe: true
  })
  const mesh = new Mesh(geo, mt)
  return mesh
}

const drawTorusKnot = (): Mesh => {
  const geo = new TorusKnotGeometry(10, 2, 200, 30, 4, 5)
  const material = new MeshMatcapMaterial({
    color: 'orange'
  })
  const mesh = new Mesh(geo, material)
  return mesh
}

// 高级几何
const drawConvex = (): Mesh => {
  function generatePoints() {
    const points = []
    for (let i = 0; i < 20; i++) {
      const x = -15 + Math.round(Math.random() * 30)
      const y = -15 + Math.round(Math.random() * 30)
      const z = -15 + Math.round(Math.random() * 30)
      points.push(new Vector3(x, y, z))
    }
    const spGroup = new Object3D()
    const material = new MeshBasicMaterial({
      color: 'orange'
    })
    points.forEach(p => {
      const spGeom = new SphereGeometry(0.2)
      const spMesh = new Mesh(spGeom, material)
      spMesh.position.copy(p)
      spGroup.add(spMesh)
    })
    scene.add(spGroup)
    return points
  }
  const points = generatePoints()
  const convexMaterial = new MeshLambertMaterial({
    color: 'red',
    wireframe: true
  })
  const convexGeo = new ConvexGeometry(points)
  const convexMesh = new Mesh(convexGeo, convexMaterial)
  return convexMesh

}

const drawLathe = (): Object3D => {
  function generatePoints() {
    const points = []
    const count = 60
    const height = 5
    for (let i = 0; i < count; i++) {
      points.push(new Vector2((Math.sin(i * 0.2) + Math.cos(i * 0.3)) * height + 12, i - count + count / 2))
    }
    const spGroup = new Object3D()
    const material = new MeshBasicMaterial({
      color: 'red'
    })
    points.forEach(point => {
      const spGeo = new SphereGeometry(0.2)
      const spMesh = new Mesh(spGeo, material)
      spMesh.position.set(point.x, point.y, 0)
      spGroup.add(spMesh)
    })
    scene.add(spGroup)
    return points
  }

  const points = generatePoints()
  const latheGeo = new LatheGeometry(points, 30)
  const meshMaterial = new MeshNormalMaterial()
  meshMaterial.side = DoubleSide
  const wireFrameMat = new MeshBasicMaterial()
  wireFrameMat.wireframe = true

  const mesh = SceneUtils.createMultiMaterialObject(latheGeo, [meshMaterial, wireFrameMat])
  return mesh
}


function drawShape() {
  const shape = new Shape()
  shape.moveTo(10, 10)
  shape.lineTo(10, 40)
  shape.bezierCurveTo(15, 25, 25, 25, 30, 40)
  shape.splineThru([
    new Vector2(32, 30),
    new Vector2(28, 20),
    new Vector2(30, 10)
  ])
  shape.quadraticCurveTo(20, 15, 10, 10)

  const hole1 = new Path()
  hole1.absellipse(16, 24, 2, 3, 0, Math.PI * 2, true, 0)
  shape.holes.push(hole1)

  const hole2 = new Path()
  hole2.absellipse(23, 24, 2, 3, 0, Math.PI * 2, true, 0)
  shape.holes.push(hole2)

  const hole3 = new Path()
  hole3.absarc(20, 16, 2, 0, Math.PI, true)
  shape.holes.push(hole3)

  return shape
}

const drawExtrude = (): Object3D => {
  const geo = new ExtrudeGeometry(drawShape(), {
    depth: 10
  })
  const meshMaterial = new MeshNormalMaterial({
    flatShading: true,
    side: DoubleSide
  })
  const mesh = new Mesh(geo, meshMaterial)
  return mesh
}

const drawText = (): void => {
  const loader = new FontLoader()
  const path = new URL('./assets/YEFONTYanShanTinXinKai_Regular.json', import.meta.url).href
  loader.load(path, (loadedFont) => {
    const textGeo = new TextGeometry('国资国企监管平台', {
      font: loadedFont,
      size: 6,
      height: 3
    })

    const material = new MeshNormalMaterial({
      // flatShading: true,
      side: DoubleSide
    })
    // const wireframeMat = new MeshBasicMaterial({
    //   wireframe: true
    // })

    const mesh = new Mesh(textGeo, material)
    // const mesh = SceneUtils.createMultiMaterialObject(textGeo, [material, wireframeMat])
    mesh.position.x = -30
    scene.add(mesh)
  })
}

onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
