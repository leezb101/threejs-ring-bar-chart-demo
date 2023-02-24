<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, AxesHelper, Color, SpriteMaterial, Sprite, BufferGeometry, PointsMaterial, Float32BufferAttribute, Points, Texture, Vector2, ColorRepresentation, Vector3, TextureLoader, AdditiveBlending, BoxGeometry, MeshNormalMaterial, DoubleSide, MeshBasicMaterial, SphereGeometry, Group, Material, Mesh } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls'
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils'
import { mergeBufferGeometries } from 'three/examples/jsm/utils/BufferGeometryUtils'


const threeRef = ref<HTMLElement>()
const scene = new Scene()

function createMesh(geom: BufferGeometry) {
  const meshMaterial = new MeshNormalMaterial()
  meshMaterial.side = DoubleSide
  const wireFrameMaterial = new MeshBasicMaterial({
    transparent: true,
    opacity: 0.5,
    wireframeLinewidth: 0.5
  })
  wireFrameMaterial.wireframe = true
  const mesh = SceneUtils.createMultiMaterialObject(geom, [meshMaterial, wireFrameMaterial])
  return mesh
}

function createGroup() {
  const sphereGeometry = new SphereGeometry(5, 10, 10)
  const cubeGeometry = new BoxGeometry(6, 6, 6)
  const sphere = createMesh(sphereGeometry)
  sphere.position.x = -20
  const cube = createMesh(cubeGeometry)
  const group = new Group()
  group.add(sphere)
  group.add(cube)
  return group
}

function createMergeGeo() {
  const meshMaterial = new MeshNormalMaterial()
  const cubeGeo1 = new BoxGeometry(6, 6, 6)
  const sphereGeo = new SphereGeometry(4, 20, 20)
  let geoArray = [cubeGeo1, sphereGeo]
  let materialArray = [meshMaterial, meshMaterial]
  let merged = mergeBufferGeometries(geoArray, true)
  let mergeMesh = new Mesh(merged, materialArray)
  return mergeMesh
}

function init() {
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 500)
  const renderer = new WebGLRenderer({})
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  // const group = createGroup()
  // scene.add(group)
  const mergeMesh = createMergeGeo()
  scene.add(mergeMesh)

  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const track = new TrackballControls(camera, renderer.domElement)



  function renderScene() {
    track.update()
    requestAnimationFrame(renderScene)
    // group.rotation.y += 0.04
    mergeMesh.rotation.y += 0.04
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
