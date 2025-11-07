<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <TimerComponent />
  </div>
</template>

<script>
import TimerComponent from './components/Timer.vue'

export default {
  name: 'App',
  components: {
    TimerComponent
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