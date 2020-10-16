<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <MineField/>
</template>

<script>
import MineField from './components/mineField.vue'

export default {
  name: 'App',
  components: {
    MineField
  }
}
</script>

<style>
html,
body {
    box-sizing: border-box;
    background-color: whitesmoke;
}
#app {
  font-family: Avenir, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: whitesmoke;
  /* margin-top: 60px; */
  background-color: black;
  width: 100%;
  height: 100vh;
}
</style>
