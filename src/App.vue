<template>
  <div id="app">
    <Empresarios />
  </div>
</template>

<script>
import Empresarios from './components/Empresarios.vue'

export default {
  name: 'App',
  components: {
    Empresarios
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
}
</style>
