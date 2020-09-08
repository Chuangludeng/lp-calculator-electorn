<template>
  <div id="app">
    <Main_Component/>
  </div>
</template>

<script>
import Main_Component from './components/Main_Component.vue'

export default {
  name: 'app',
  components: {
    Main_Component
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
