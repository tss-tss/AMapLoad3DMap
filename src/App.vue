<template>
  <div id="app">
    <AMap3D />
    <EchatsMap />
  </div>
</template>

<script>
import AMap3D from './components/AMap3D.vue'
import EchatsMap from './components/EchatsMap.vue'
export default {
  name: 'App',
  components: {
    AMap3D,
    EchatsMap
  }
}
</script>

<style lang="scss">
html, body {
  margin: 0;
  height: 100%;
  width: 100%;
}
#app {
  width: 100vw;
  height: 100vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
