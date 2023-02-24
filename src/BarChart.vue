<template>
  <div ref="threeRef">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { Scene, PerspectiveCamera, WebGLRenderer, Color, AxesHelper, PlaneGeometry, MeshLambertMaterial, Mesh, BoxGeometry, SphereGeometry, Vector3, Object3D, SpotLight, AmbientLight, PCFSoftShadowMap } from 'three'

const threeRef = ref()

const dataList = [15, 22, 12, 8, 5]

function init() {
  const scene = new Scene()
  const axes = new AxesHelper(20)
  scene.add(axes)

  const renderer = new WebGLRenderer({
    antialias: true
  })
  renderer.setClearColor(new Color(0xEEEEEE))
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = PCFSoftShadowMap

  const planeGeo = new PlaneGeometry(60, 20)
  const planeBasicMaterial = new MeshLambertMaterial({ color: 0xcccccc })
  const plane = new Mesh(planeGeo, planeBasicMaterial)
  plane.receiveShadow = true
  plane.rotation.x = -0.5 * Math.PI
  plane.position.set(30, 0, 10)
  scene.add(plane)

  const light = new SpotLight(new Color(0xeeeeee))
  light.position.set(0, 30, 0)
  light.lookAt(0, 0, 30)
  light.castShadow = true
  scene.add(light)

  const alight = new AmbientLight(0xeeeeee)
  scene.add(alight)


  const barMaterial = new MeshLambertMaterial({ color: 0xd43335 })

  const barsRef = ref<Object3D[]>([])

  for (let i = 0; i < dataList.length; i++) {
    const barGeo = new BoxGeometry(3, 0, 3)
    const bar = new Mesh(barGeo, barMaterial)
    bar.position.x = (5 + 3) * (i + 1)
    bar.position.y = dataList[i] / 2
    bar.position.z = 1.5
    bar.castShadow = true
    bar.name = `bar-${i}`
    scene.add(bar)
    barsRef.value.push(bar)
  }


  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight)
  camera.position.set(-30, 50, 30)
  camera.lookAt(new Vector3(20, 0, 0))
  threeRef.value!.appendChild(renderer.domElement)
  let scaleY = [0, 0, 0, 0, 0]
  function animate() {

    for (let i = 0; i < barsRef.value.length; i++) {
      let b = barsRef.value[i]
      scaleY[i] += 0.05
      const threeRef = ref()
      function init() {
        const scene = new Scene()
        const axes = new AxesHelper(20)
        scene.add(axes)

        const renderer = new WebGLRenderer({
          antialias: true
        })
        renderer.setClearColor(new Color(0xEEEEEE))
        renderer.setSize(window.innerWidth, window.innerHeight)

        const planeGeo = new PlaneGeometry(60, 20)
        const planeBasicMaterial = new MeshBasicMaterial({ color: 0xcccccc })
        const plane = new Mesh(planeGeo, planeBasicMaterial)
        plane.rotation.x = -0.5 * Math.PI
        plane.position.set(30, 0, 10)
        scene.add(plane)

        // 盒子
        const cubeGeo = new BoxGeometry(4, 4, 14)
        const cubeMaterial = new MeshBasicMaterial({ color: 0xff0000, wireframe: true })
        const cube = new Mesh(cubeGeo, cubeMaterial)
        cube.position.set(10, 3, 10)
        scene.add(cube)

        // 球形几何体
        const sphereGeo = new SphereGeometry(4, 20, 20)
        const sphereMaterial = new MeshBasicMaterial({ color: 0x7777ff, wireframe: true })
        const sphere = new Mesh(sphereGeo, sphereMaterial)
        sphere.position.set(40, 4, 10)
        scene.add(sphere)

        const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight)
        camera.position.set(-30, 40, 30)
        camera.lookAt(scene.position)
        threeRef.value!.appendChild(renderer.domElement)
        renderer.render(scene, camera)
      }

      onMounted(() => {
        init()
      })





      b.scale.set(1, scaleY[i] / dataList[i], 1)
    }
    console.log(scaleY)
    renderer.render(scene, camera)
    requestAnimationFrame(animate)
  }
  animate()
}



onMounted(() => {
  init()
})



</script>

<style scoped>

</style>
