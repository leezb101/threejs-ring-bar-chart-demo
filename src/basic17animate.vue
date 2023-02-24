<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { AxesHelper, BoxGeometry, Color, Mesh, MeshNormalMaterial, PerspectiveCamera, Raycaster, Scene, Vector3, WebGLRenderer } from 'three'
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';

const threeRef = ref<HTMLElement>()
const scene = new Scene()
const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 500)

function init() {

  const renderer = new WebGLRenderer({})
  renderer.setClearColor(new Color(0x000000))
  renderer.setSize(window.innerWidth, window.innerHeight)

  const axes = new AxesHelper(20)
  scene.add(axes)

  const cube = createCube()
  scene.add(cube)
  clickControl()

  function clickControl() {
    document.addEventListener('click', (eve) => {
      let vector = new Vector3(eve.clientX / window.innerWidth * 2 - 1, -(eve.clientY / innerHeight) * 2 + 1, 0.5)
      // 屏幕坐标转换为three场景坐标（鼠标点击坐标转三维坐标）
      vector = vector.unproject(camera)

      // 摄像机位置向场景发射“光线”
      const raycaster = new Raycaster(camera.position, vector.sub(camera.position).normalize())
      // 判断指定的对象中哪些被该“光线”照射到了,此处需要判断的只有一个cube
      const intersects = raycaster.intersectObjects([cube])

      if (intersects.length) {
        // 旋转网格
        console.log(intersects[0])
        intersects[0].object.rotation.x += 10 * Math.PI / 180
        intersects[0].object.rotation.y += 10 * Math.PI / 180
        intersects[0].object.rotation.z += 10 * Math.PI / 180
      }
    })
  }

  camera.position.set(-30, 40, 30)
  camera.lookAt(scene.position)
  threeRef.value!.appendChild(renderer.domElement)

  const track = new TrackballControls(camera, renderer.domElement)
  function renderScene() {
    track.update()
    requestAnimationFrame(renderScene)
    renderer.render(scene, camera)
  }
  renderScene()
}

function createCube() {
  const cubeMaterial = new MeshNormalMaterial()
  const cubeGeo = new BoxGeometry(10, 10, 10)
  const cubeMesh = new Mesh(cubeGeo, cubeMaterial)
  return cubeMesh
}

onMounted(() => {
  init()
})


</script>

<style scoped>

</style>
