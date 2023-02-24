<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, AxesHelper, Color, SpriteMaterial, Sprite, BufferGeometry, PointsMaterial, Float32BufferAttribute, Points, Texture, Vector2, ColorRepresentation, Vector3, TextureLoader, AdditiveBlending } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls'


const threeRef = ref<HTMLElement>()
const scene = new Scene()

function init() {
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 500)
  const renderer = new WebGLRenderer({})
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const rainUrl = new URL('./assets/raindrop.png', import.meta.url).href
  const texture = new TextureLoader().load(rainUrl)
  const material = new PointsMaterial({
    size: 20,
    transparent: true,
    opacity: 0.5,
    map: texture,
    sizeAttenuation: true,
    color: 0xffffff,
    blending: AdditiveBlending,
    depthTest: false,
    depthWrite: false
  })

  const geom = new BufferGeometry()
  let veticsFloat32Array = []
  const range = 500
  for (let i = 0; i < 1000; i++) {
    const particle = new Vector3(
      Math.random() * range - range / 2,
      Math.random() * range - range / 2,
      Math.random() * range - range / 2
    )
    veticsFloat32Array.push(...particle.toArray())
  }

  function initRain(vetics: any) {
    const vertices = new Float32BufferAttribute(vetics, 3)
    geom.attributes['position'] = vertices
    const cloud = new Points(geom, material)
    scene.add(cloud)
    return cloud
  }

  const speed = 1
  let cloud: Points | null = null
  cloud = initRain(veticsFloat32Array)




  camera.position.set(-60, 80, 60)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const track = new TrackballControls(camera, renderer.domElement)

  function renderScene() {
    track.update()
    requestAnimationFrame(renderScene)
    if (cloud) {
      scene.remove(cloud)
    }
    let rainPositionArray = Array.from(cloud?.geometry.attributes['position'].array)
    for (let i = 0; i < rainPositionArray.length; i += 3) {
      (rainPositionArray[i + 1] as number) -= speed
      if ((rainPositionArray[i + 1] as number) < 0) {
        rainPositionArray[i + 1] = Math.random() * range - range / 2
      }
    }
    cloud = initRain(rainPositionArray)
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
