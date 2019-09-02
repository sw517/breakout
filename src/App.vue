<template>
  <div id="app">
    <BreakoutCanvas />
  </div>
</template>

<script>
import BreakoutCanvas from './components/BreakoutCanvas.vue';

export default {
  name: 'app',
  components: {
    BreakoutCanvas,
  },
};
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 20px;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}
</style>
