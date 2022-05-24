<template>
<div id="app">
<!--    <VMyChart title="Every Sec update chart" />-->
<!--    <MyChart   title="Chart with horizantalBreak" />-->
    <DoubleChart   title="Modified Chart" />
  </div>
</template>

<script>
/* eslint-disable */
import VMyChart from './components/VmyChart.vue'
import MyChart from "@/components/MyChart";
import DoubleChart from "@/components/DoubleMyChart";

export default {
  name: 'App',
  components: {
    MyChart,
    VMyChart,
    DoubleChart
  }
}
</script>

<style>
#app {
}
</style>
