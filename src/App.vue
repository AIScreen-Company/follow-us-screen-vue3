<template>
  <div id="app">
    <Screen />
  </div>
</template>

<script>
import Screen from './components/Screen.vue'

export default {
  name: 'App',
  components: {
    Screen,
  },
}
</script>

<style lang="scss">
@import './assets/Fonts/stylesheet.css';
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Satoshi !important;
}
#app {
  height: 100vh;
  width: 100vw;
}
</style>
