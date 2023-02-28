<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue';
import Sample from './Sample.vue';
import SampleB from './SampleB.vue';
import BarChart from './BarChart.vue';
// import basic1 from './basic1.vue';
// import basic2 from './basic2.vue'
// import basic3 from './basic3.vue';
// import basic5Dat from './basic5Dat.vue';
// import basic7Light from './basic7Light.vue';
// import basic8mesh from './basic8mesh.vue';
// import basic10Light from './basic10Light.vue';
// import Basic11Material from './basic11Material.vue';
// import basic12Geo from './basic12Geo.vue';
// import basic14sprite from './basic14sprite.vue';
// import basic15rain from './basic15rain.vue';
// import basic16group from './basic16group.vue';
// import basic17animate from './basic17animate.vue';
// import basic18tween from './basic18tween.vue';
// import Basic19trex from './basic19trex.vue';
// import basic20trans from './basic20trans.vue';
import transbar from './transbar.vue';
import ringchart from './ringchart.vue';
import RingChartComp3D from './RingChartComp3D.vue';
import BarChart3DComp from './BarChart3DComp.vue';

const ringData = [
  { label: '公司1', value: 10 },
  { label: '公司2', value: 30 },
  { label: '公司3', value: 20 },
  { label: '公司4', value: 25 },
  { label: '公司5', value: 15 },
];
</script>

<template>
  <!-- <Sample /> -->
  <!-- <SampleB /> -->
  <!-- <basic3 /> -->
  <!-- <basic5Dat /> -->
  <!-- <basic7Light /> -->
  <!-- <basic8mesh /> -->
  <!-- <basic10Light /> -->
  <!-- <Basic11Material /> -->
  <!-- <basic12Geo /> -->
  <!-- <basic14sprite /> -->
  <!-- <basic15rain /> -->
  <!-- <basic16group /> -->
  <!-- <basic17animate /> -->
  <!-- <basic18tween /> -->
  <!-- <Basic19trex /> -->
  <!-- <basic20trans /> -->
  <!-- <transbar /> -->
  <!-- <ringchart /> -->
  <div class="container">
    <div class="grid-1"></div>
    <div class="grid-2"></div>
    <div class="grid-3">
      <RingChartComp3D :datas="ringData" />
    </div>
    <div class="grid-4">
      <BarChart3DComp :datas="ringData" />
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.container {
  height: 100vh;
  display: grid;
  grid-template-columns: repeat(2, 49%);
  grid-template-rows: repeat(2, 49%);
  grid-gap: 1%;
}

.grid-1 {
  background-color: cyan;
}

.grid-2 {
  background: yellowgreen;
}

.grid-3 {
  background-color: pink;
  padding: 12px;
}

.grid-4 {
  background: orange;
}
</style>
