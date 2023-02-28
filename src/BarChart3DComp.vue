<template>
  <div class="barchart-container" ref="barChartRef">
    <div class="bar-tooltip" :style="{
      left: `${tooltipX + 10}px`,
      top: `${tooltipY}px`,
    }" v-show="showTooltip">
      <span :style="{ background: barColors[tooltipCurrentIndex] }"></span>{{ datas[tooltipCurrentIndex].label }}:{{
        datas[tooltipCurrentIndex].value
      }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  BoxGeometry,
  MeshPhongMaterial,
  AmbientLight,
  Raycaster,
  Vector2,
  Color,
  PointLight,
  Mesh,
  DoubleSide,
  Vector3,
  PlaneGeometry,
  Object3D,
} from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader';

const barChartRef = ref<HTMLDivElement>();
const tooltipX = ref<number>(0);
const tooltipY = ref<number>(0);
const showTooltip = ref<boolean>(false);
const tooltipCurrentIndex = ref<number>(0);


// 初始化scene
const scene = new Scene();
// 初始化renderer
const renderer = new WebGLRenderer({
  antialias: true,
});
// renderer.setClearColor(0xffffffff);
renderer.setClearAlpha(0);

const camera = new PerspectiveCamera(45, 1, 0.1, 1000);

interface DataItem {
  label: string;
  value: number;
}

const props = defineProps<{
  datas: DataItem[];
  colors?: string[] | undefined;
  highlightColors?: string[] | undefined;
}>();
const DATA_SCALE_FACTOR = 100 // 用来将所有数据换算为100为最大值的百分比

const barsRef = ref<Mesh[]>([]);

const defaultColors = [
  '#5B8FF9',
  '#E5679A',
  '#44D7B6',
  '#F7B500',
  '#8F84E0',
  '#FF8362',
  '#6FD8D6',
  '#E55050',
];
const defaultHighlightColors = [
  '#7EA8FF',
  '#FF96C1',
  '#80E5CE',
  '#FFD338',
  '#B1A7FD',
  '#FFAB94',
  '#99F7F5',
  '#FF6E6E',
];

const barColors = computed(() => {
  if (props.colors) return props.colors;
  return defaultColors;
});

const barHighlightColors = computed(() => {
  if (props.highlightColors) return props.highlightColors;
  return defaultHighlightColors;
});

// 通过射线获取鼠标悬停的相关配置
let INTERSECTED: any;
const raycaster = new Raycaster(); // 鼠标穿透选择用的垂直z轴的射线
const mousePoint = new Vector2(); // 鼠标所在xy平面坐标

// 用于处理选中环图高亮部分的配置，效果选择器
let composer: EffectComposer, outlinePass: OutlinePass, effectFXAA: ShaderPass;
let selectedObjects = [] as unknown[];


const init = () => {
  if (!barChartRef.value) return
  renderer.setSize(barChartRef.value.clientWidth, barChartRef.value.clientHeight);
  camera.aspect = barChartRef.value.clientWidth / barChartRef.value.clientHeight;
  camera.position.set(20, 20, 80);

  barChartRef.value.append(renderer.domElement);

  // 初始化镜头控制器
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableZoom = false;
  controls.enablePan = false;
  controls.enableRotate = false



  // 初始化两种光源
  const alight = new AmbientLight(0xffffff, 0.5);
  scene.add(alight);

  const pointLight = new PointLight(0xeeeeee, 0.8);
  pointLight.position.set(-20, 30, 40);
  scene.add(pointLight);

  addBars()
  addGround()
  constructComposer();

  // todo: 增加mousemove事件
  barChartRef.value.addEventListener('mousemove', onMouseMove)

  let sy = 0
  function syAdd() {
    if (sy >= 1) return
    sy += 0.01
  }
  function renderScene() {
    syAdd()

    const total = props.datas.reduce((pre, cur) => cur.value + pre, 0)
    const scaledDatas = props.datas.map((item, idx) => {
      const res = item.value / total * DATA_SCALE_FACTOR
      return {
        label: item.label,
        value: res,
        index: idx
      }
    })

    barsRef.value.map((bar, index) => {
      bar.scale.y = sy
      bar.position.y = scaledDatas[index].value * sy / 2
    })
    camera.lookAt(new Vector3(20, 20, 0));
    requestAnimationFrame(renderScene)
    // renderer.render(scene, camera)
    composer.render()
  }

  renderScene()
}

const addBars = () => {

  const total = props.datas.reduce((pre, cur) => cur.value + pre, 0)
  const scaledDatas = props.datas.map((item, idx) => {
    const res = item.value / total * DATA_SCALE_FACTOR
    return {
      label: item.label,
      value: res,
      index: idx
    }
  })

  for (let i = 0; i < props.datas.length; i++) {
    const geo = new BoxGeometry(2, scaledDatas[i].value, 2)
    const material = new MeshPhongMaterial({
      color: barColors.value[i],
      opacity: 1,
      transparent: true,
      shininess: 1,
      side: DoubleSide,
      emissive: 'white',
      emissiveIntensity: 0.2,
    })
    const bar = new Mesh(geo, material)
    bar.position.set(4 + (4 + 2) * i, 0, 1) // bar宽度厚度为4，间距为4，最高不超过100
    bar.scale.y = 0
    bar.name = `bar-${scaledDatas[i].index}`
    scene.add(bar)
    barsRef.value.push(bar)
  }
}

const addGround = () => {
  const geo = new PlaneGeometry(1000, 4)
  const material = new MeshPhongMaterial({
    color: 'gray',
    emissive: 'white',
    emissiveIntensity: 0.4,
    side: DoubleSide,
  })
  const ground = new Mesh(geo, material)
  ground.name = 'ground'
  ground.rotation.x = -Math.PI / 2
  ground.position.set(0, 0, 1)
  scene.add(ground)
}

const constructComposer = () => {
  if (!barChartRef.value) return;
  composer = new EffectComposer(renderer);
  const renderPass = new RenderPass(scene, camera);
  composer.addPass(renderPass);
  outlinePass = new OutlinePass(

    new Vector2(barChartRef.value.clientWidth, barChartRef.value.clientHeight),
    scene,
    camera
  );
  outlinePass.visibleEdgeColor = new Color(0xffaa22);
  outlinePass.edgeStrength = 10;
  outlinePass.edgeGlow = 0.5;
  outlinePass.edgeThickness = 1;
  outlinePass.pulsePeriod = 1;
  composer.addPass(outlinePass);

  effectFXAA = new ShaderPass(FXAAShader);
  effectFXAA.uniforms['resolution'].value.set(
    1 / barChartRef.value.clientWidth,
    1 / barChartRef.value.clientHeight
  );
  composer.addPass(effectFXAA);
};

const onMouseMove = (event: MouseEvent) => {
  if (!barChartRef.value) return;
  let px = renderer.domElement.getBoundingClientRect().left;
  let py = renderer.domElement.getBoundingClientRect().top;
  mousePoint.x = ((event.clientX - px) / barChartRef.value.clientWidth) * 2 - 1;
  mousePoint.y = -((event.clientY - py) / barChartRef.value.clientHeight) * 2 + 1;
  checkIntersection(event);
};

const checkIntersection = (event: MouseEvent) => {
  raycaster.setFromCamera(mousePoint, camera);
  const intersects = raycaster.intersectObjects(scene.children, true);
  if (intersects.length) {
    if (
      INTERSECTED != intersects[0].object &&
      intersects[0].object instanceof Mesh
    ) {
      // 跳过底座的选中效果
      if (intersects[0].object.name === 'ground') return;
      INTERSECTED = intersects[0].object;
      const idx = Number((INTERSECTED.name as string).split('-')[1]);
      highlightMesh(idx);
      showTips(idx, event);
      addSelection(intersects[0].object);
      outlinePass.selectedObjects = selectedObjects as any[];
    }
  } else {
    showTooltip.value = false;
    INTERSECTED = null;
    addSelection(null);
    outlinePass.selectedObjects = [];
  }
};

const highlightMesh = (idx: number) => {
  barsRef.value.forEach((item, i) => {
    if (item.name && item.name == `bar-${idx}`) {
      const material = new MeshPhongMaterial({
        color: barHighlightColors.value[idx],
        opacity: 1,
        transparent: true,
        shininess: 1,
        side: DoubleSide,
        emissive: 'white',
        emissiveIntensity: 0.2,
      });
      item.material = material;
    } else {
      const material = new MeshPhongMaterial({
        color: barColors.value[i],
        opacity: 1,
        transparent: true,
        shininess: 1,
        side: DoubleSide,
        emissive: 'white',
        emissiveIntensity: 0.2,
      });
      item.material = material;
    }
  });
};

const showTips = (idx: number, event: MouseEvent) => {
  // let px = renderer.domElement.getBoundingClientRect().left;
  // let py = renderer.domElement.getBoundingClientRect().top;
  // console.log(px, py);
  // console.log(event);
  showTooltip.value = true;
  tooltipCurrentIndex.value = idx;
  tooltipX.value = event.clientX;
  tooltipY.value = event.clientY;
};

const addSelection = (obj: Object3D | Mesh | null) => {
  if (obj) {
    selectedObjects = [];
    selectedObjects.push(obj);
  } else {
    selectedObjects = [];
  }
};

onMounted(() => {
  init()
})

</script>

<style scoped lang="scss">
.barchart-container {
  height: 100%;
  width: 100%;

  .bar-tooltip {
    position: absolute;
    padding: 10px;
    background: rgba(230, 300, 300, 0.6);
    border-radius: 4px;
    transition: all 0.3s;

    div {
      display: flex;
      align-items: center;
      color: #fff;

      span {
        border-radius: 50%;
        margin-right: 5px;
        width: 10px;
        height: 10px;
      }
    }
  }
}
</style>
