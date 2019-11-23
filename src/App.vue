<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <Lobby/>
  </div>
</template>

<script>
import Lobby from './components/Lobby.vue'

export default {
  name: 'app',
  components: {
    Lobby
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
  margin-top: 60px;
}
</style>
