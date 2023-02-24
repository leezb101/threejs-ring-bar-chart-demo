<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, AxesHelper, Color, SpriteMaterial, Sprite, BufferGeometry, PointsMaterial, Float32BufferAttribute, Points, Texture, Vector2, ColorRepresentation, Vector3 } from 'three'
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

  // createSprites()
  // createPoints()
  createCanvasSprites()

  camera.position.set(-60, 80, 60)
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

function createSprites() {
  for (let x = -15; x < 15; x++) {
    for (let y = -15; y < 15; y++) {
      const material = new SpriteMaterial({
        color: Math.random() * 0xffffff
      })
      const sprite = new Sprite(material)
      sprite.position.set(x * 4, y * 4, 0)
      scene.add(sprite)
    }
  }
}

// 高性能版的sprites
function createPoints() {
  const gem = new BufferGeometry()
  const material = new PointsMaterial({
    size: 2,
    vertexColors: true,
    color: 0xffffff,
  })
  let veticsFloat32Array = []
  let veticsColors = []
  for (let x = -15; x < 15; x++) {
    for (let y = -15; y < 15; y++) {
      veticsFloat32Array.push(x * 4, y * 4)
      const randomColor = new Color(`#${(Math.random() * 0xffffff << 0).toString(16)}`)
      veticsColors.push(randomColor.r, randomColor.g, randomColor.b)
    }
  }

  const vertices = new Float32BufferAttribute(veticsFloat32Array, 3)
  const colors = new Float32BufferAttribute(veticsColors, 3)
  gem.attributes['position'] = vertices
  gem.attributes['color'] = colors

  const cloud = new Points(gem, material)
  scene.add(cloud)
}

function createCanvasSprites() {
  const createGhostTexture = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')
    if (!ctx) return
    ctx.translate(-81, -84)

    // body
    ctx.fillStyle = 'orange'
    ctx.beginPath()
    ctx.moveTo(83, 116)
    ctx.lineTo(83, 102)
    ctx.bezierCurveTo(83, 94, 89, 88, 97, 88)
    ctx.bezierCurveTo(105, 88, 111, 94, 111, 102)
    ctx.lineTo(111, 116)
    ctx.lineTo(106.333, 111.333)
    ctx.lineTo(101.666, 116)
    ctx.lineTo(97, 111.333)
    ctx.lineTo(92.333, 116)
    ctx.lineTo(87.666, 111.333)
    ctx.lineTo(83, 116)
    ctx.fill()

    // eye
    ctx.fillStyle = 'white'
    ctx.beginPath()
    ctx.moveTo(91, 96)
    ctx.bezierCurveTo(88, 96, 87, 99, 87, 101)
    ctx.bezierCurveTo(87, 103, 88, 106, 91, 106)
    ctx.bezierCurveTo(94, 106, 95, 103, 95, 101)
    ctx.bezierCurveTo(95, 99, 94, 96, 91, 96)
    ctx.moveTo(103, 96)
    ctx.bezierCurveTo(100, 96, 99, 99, 99, 101)
    ctx.bezierCurveTo(99, 103, 100, 106, 103, 106)
    ctx.bezierCurveTo(106, 106, 107, 103, 107, 101)
    ctx.bezierCurveTo(107, 99, 106, 96, 103, 96)
    ctx.fill()

    // the pupils
    ctx.fillStyle = 'blue'
    ctx.beginPath()
    ctx.arc(101, 102, 2, 0, Math.PI * 2, true)
    ctx.fill()
    ctx.beginPath()
    ctx.arc(89, 102, 2, 0, Math.PI * 2, true)
    ctx.fill()

    const texture = new Texture(canvas)
    texture.needsUpdate = true
    return texture
  }

  const createCloud = (size: number, transparent: boolean, opacity: number, sizeAttenuation: boolean, color: ColorRepresentation) => {
    const geom = new BufferGeometry()
    const material = new PointsMaterial({
      size,
      transparent,
      opacity,
      sizeAttenuation,
      color,
      map: createGhostTexture()
    })
    let veticsFloat32Array = []
    const range = 500
    for (let i = 0; i < 1000; i++) {
      const paticle = new Vector3(Math.random() * range - range / 2, Math.random() * range - range / 2, Math.random() * range - range / 2)
      veticsFloat32Array.push(paticle.x, paticle.y, paticle.z)
    }
    const vertices = new Float32BufferAttribute(veticsFloat32Array, 3)
    geom.attributes['position'] = vertices
    const cloud = new Points(geom, material)
    scene.add(cloud)
  }
  createCloud(20, true, 0.6, true, 0xffffff)
}

onMounted(() => {
  init()
})

</script>

<style scoped>

</style>
