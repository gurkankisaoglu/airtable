<template>
  <div id="app">
    <Diversity></Diversity>
  </div>
</template>

<script>
  import Diversity from './components/Diversity.vue'

  export default {
    name: 'App',
    components: {
      Diversity
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
