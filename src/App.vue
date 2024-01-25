<template>
  <div id="app">
    <Map />
    <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Map from './components/map.vue'

export default {
  name: 'app',
  components: {
    HelloWorld,
    Map
  }
}
</script>

<style lang="less">
#app {
  height: 100%;
}
</style>
