<template>
  <div :style="{ width: '100%', height: '100%' }" ref="chartRef">
    <div
      class="tooltip"
      :style="{
        left: `${tooltipX + 10}px`,
        top: `${tooltipY}px`,
      }"
      v-show="showTooltip"
    >
      <span :style="{ background: ringColors[tooltipCurrentIndex] }"></span
      >{{ datas[tooltipCurrentIndex].label }}:{{
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
  Raycaster,
  Vector2,
  AxesHelper,
  AmbientLight,
  PointLight,
  Color,
  Group,
  Shape,
  ExtrudeGeometryOptions,
  ExtrudeGeometry,
  MeshPhongMaterial,
  DoubleSide,
  Mesh,
  Object3D,
  MaterialLoader,
} from 'three';
import * as SceneUtils from 'three/examples/jsm/utils/SceneUtils';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass';
import { FXAAShader } from 'three/examples/jsm/shaders/FXAAShader';
import basic5DatVue from './basic5Dat.vue';

const chartRef = ref<HTMLDivElement>();
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

const ringsRef = ref<Mesh[]>([]);

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

const ringColors = computed(() => {
  if (props.colors) return props.colors;
  return defaultColors;
});

const ringHighlightColos = computed(() => {
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
  if (!chartRef.value) return;
  // dom渲染后初始化size和aspect
  renderer.setSize(chartRef.value.clientWidth, chartRef.value.clientHeight);
  camera.aspect = chartRef.value.clientWidth / chartRef.value.clientHeight;
  camera.position.set(-35, 20, 30);
  camera.lookAt(scene.position);
  chartRef.value.append(renderer.domElement);

  // 初始化镜头控制器
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableZoom = false;
  controls.enablePan = false;

  // 初始化两种光源
  const alight = new AmbientLight(0xffffff, 0.5);
  scene.add(alight);

  const pointLight = new PointLight(0xeeeeee, 0.8);
  pointLight.position.set(-20, 30, 40);
  scene.add(pointLight);

  composer = new EffectComposer(renderer);
  const renderPass = new RenderPass(scene, camera);
  composer.addPass(renderPass);

  addRings();
  addGround();
  constructComposer();

  document.addEventListener('mousemove', onMouseMove);

  // 逐帧动画
  let sy = 0;
  function syAdd() {
    if (sy >= 1) {
      return;
    }
    sy += 0.02;
  }

  function renderScene() {
    syAdd();
    ringsRef.value.map((r) => {
      r.scale.z = sy;
    });

    camera.updateMatrixWorld();
    requestAnimationFrame(renderScene);
    composer.render();
  }

  renderScene();
};

const constructComposer = () => {
  if (!chartRef.value) return;
  outlinePass = new OutlinePass(
    new Vector2(chartRef.value.clientWidth, chartRef.value.clientHeight),
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
    1 / chartRef.value.clientWidth,
    1 / chartRef.value.clientHeight
  );
  composer.addPass(effectFXAA);
};

const addRings = () => {
  const innerR = 8;
  const outerR = 12;
  const total = props.datas.reduce((pre, cur) => cur.value + pre, 0);
  const group = new Group();

  // 定义需要累加的环状块z旋转变量
  let deltaRotate = 0;

  let copyData = [...props.datas];
  const sortedRingData = copyData.sort((a, b) => a.value - b.value);

  for (let i = 0; i < sortedRingData.length; i++) {
    const partShape = new Shape();
    const percent = sortedRingData[i].value / total;
    // 绘制环状平面图（x,y）坐标系
    partShape.moveTo(innerR, 0);
    partShape.absarc(0, 0, innerR, 0, percent * 2 * Math.PI, false);
    partShape.lineTo(
      outerR * Math.cos(percent * 2 * Math.PI),
      outerR * Math.sin(percent * 2 * Math.PI)
    );
    partShape.absarc(0, 0, outerR, percent * 2 * Math.PI, 0, true);
    partShape.lineTo(innerR, 0);

    // 设置xy坐标系挤出的配置项
    const extrudeOptions: ExtrudeGeometryOptions = {
      depth: 14 * (sortedRingData[i].value / total),
      curveSegments: 30,
      steps: 5,
      bevelEnabled: false,
      bevelSegments: 0,
      bevelOffset: 0,
      bevelSize: 0,
      bevelThickness: 0,
    };
    const partGeo = new ExtrudeGeometry(partShape, extrudeOptions);
    const material = new MeshPhongMaterial({
      color: ringColors.value[i],
      opacity: 1,
      transparent: true,
      shininess: 1,
      side: DoubleSide,
      emissive: 'white',
      emissiveIntensity: 0.2,
    });

    const mesh = new Mesh(partGeo, material);
    mesh.name = `ring-${i}`;
    mesh.rotation.z = Math.PI * 2 * deltaRotate;
    deltaRotate += sortedRingData[i].value / total;
    ringsRef.value.push(mesh);
    group.add(mesh);
  }
  // 将整个环状图从xy坐标平面沿x轴旋转至xz平面
  group.rotation.x = -Math.PI / 2;
  scene.add(group);
};

const addGround = () => {
  const innerR = 8;
  const outerR = 12;
  const arcShape = new Shape();
  arcShape.moveTo(outerR, 0);
  arcShape.lineTo(innerR, 0);
  arcShape.absarc(0, 0, innerR + 2, 0, Math.PI * 2, false);
  arcShape.absarc(0, 0, outerR + 1, 0, Math.PI * 2, true);

  const extrudeOptions: ExtrudeGeometryOptions = {
    curveSegments: 40,
    depth: 0.5,
    bevelEnabled: false,
    bevelSegments: 0,
    steps: 1,
    bevelSize: 0,
    bevelThickness: 0,
  };

  const groundGeo = new ExtrudeGeometry(arcShape, extrudeOptions);
  const material = new MeshPhongMaterial({
    color: 0xdde1d2,
    opacity: 0.6,
    transparent: true,
  });
  const ground = new Mesh(groundGeo, material);
  ground.name = 'ground';
  scene.add(ground);
  ground.rotation.x = -Math.PI / 2;
};

const onMouseMove = (event: MouseEvent) => {
  if (!chartRef.value) return;
  let px = renderer.domElement.getBoundingClientRect().left;
  let py = renderer.domElement.getBoundingClientRect().top;
  mousePoint.x = ((event.clientX - px) / chartRef.value.clientWidth) * 2 - 1;
  mousePoint.y = -((event.clientY - py) / chartRef.value.clientHeight) * 2 + 1;
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
  ringsRef.value.forEach((item, i) => {
    if (item.name && item.name == `ring-${idx}`) {
      const material = new MeshPhongMaterial({
        color: ringHighlightColos.value[idx],
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
        color: ringColors.value[i],
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
  init();
});
</script>

<style scoped lang="scss">
.tooltip {
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
</style>
