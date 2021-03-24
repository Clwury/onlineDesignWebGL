<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <custom-canvas></custom-canvas>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import customCanvas from './components/canvas'

export default {
  name: 'App',
  components: {
    // HelloWorld
    customCanvas
  }
}
</script>

<style>
/* #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
body {
  margin: 0;
  padding: 0;
}
#app {
  height: 100vh;
  display: flex;
  flex-flow: column;
}
</style>
