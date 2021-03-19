<template>
  <div id="app">
    <HousePlanMapper />
  </div>
</template>

<script>
import HousePlanMapper from './components/HousePlanMapper.vue'

export default {
  name: 'App',
  components: {
    HousePlanMapper
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
