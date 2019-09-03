<template>
  <div id="app">
    <div class="gamepad">
      <div class="power-led" />
      <div class="screen">
        <BreakoutCanvas />
      </div>
      <div class="controls">
        <div class="controls__menu">
          <div class="controls__btn controls__start">
            Start
          </div>
        </div>
        <div class="controls__directions">
          <div class="controls__btn left" />
          <div class="controls__btn right" />
        </div>
      </div>
    </div>
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
html,body {
  width: 100%;
  height: 100%;
  margin: 0;
  box-sizing: border-box;

  * {
    box-sizing: inherit;
  }
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: #2b2b2b;
}

.gamepad {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  padding: 40px;
  max-width: 320px;
  max-height: 600px;
  background-color: #3faf81;
  border-radius: 8px;
  box-shadow: 0 0 20px 2px #000;
}

.power-led {
  position: absolute;
  top: 20px;
  right: 20px;
  transform: translate(-50%, -50%);
  width: 10px;
  height: 10px;
  background-color: #a70606;
  border-radius: 100%;
  box-shadow: 0 0 2px 1px #f5b0b0;
}

.screen {
  border-radius: 4px;
  padding: 10px;
  background-color: #000;
  height: 56%;
  box-sizing: content-box;
}

.controls {
  display: flex;
  flex-direction: column;
  padding-top: 30px;
  flex-grow: 1;

  &__btn {
    background-color: #333;
  }

  &__menu {
    padding-bottom: 20px;
  }

  &__start {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100px;
    height: 30px;
    color: #b1b1b1;
  }

  &__directions {
    display: flex;
    justify-content: space-between;
    flex-grow: 1;

    .controls__btn {
      width: 45%;
      border-radius: 10px;
    }
  }
}
</style>
