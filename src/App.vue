<template>
  <div id="app">
    <div class="gamepad">
      <div class="power-led" />
      <div class="screen">
        <BreakoutCanvas
          :leftBtnPressed="leftBtnPressed"
          :rightBtnPressed="rightBtnPressed"
          :startBtnPressed="startBtnPressed"
        />
      </div>
      <div class="controls">
        <div class="controls__menu">
          <button
            @mousedown="onBtnPressed('start')"
            @mouseup="onBtnPressed('start')"
            @touchstart="onBtnPressed('start')"
            @touchend="onBtnPressed('start')"
            class="controls__btn controls__start"
            :class="{ 'pressed' : startBtnPressed }"
          >
            Start
          </button>
        </div>
        <div class="controls__directions">
          <button
            @mousedown="onBtnPressed('left')"
            @mouseup="onBtnPressed('left')"
            @touchstart="onBtnPressed('left')"
            @touchend="onBtnPressed('left')"
            class="controls__btn left"
            :class="{ 'pressed' : leftBtnPressed }"
          >
            &lt;
          </button>
          <button
            @mousedown="onBtnPressed('right')"
            @mouseup="onBtnPressed('right')"
            @touchstart="onBtnPressed('right')"
            @touchend="onBtnPressed('right')"
            class="controls__btn right"
            :class="{ 'pressed' : rightBtnPressed }"
          >
            &gt;
          </button>
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
  data() {
    return {
      leftBtnPressed: false,
      rightBtnPressed: false,
      startBtnPressed: false,
    };
  },
  methods: {
    onBtnPressed(direction) {
      switch (direction) {
        case 'left':
          this.leftBtnPressed = !this.leftBtnPressed;
          break;
        case 'right':
          this.rightBtnPressed = !this.rightBtnPressed;
          break;
        default:
          this.startBtnPressed = !this.startBtnPressed;
      }
    },
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

@media
only screen and (-webkit-min-device-pixel-ratio: 2),
only screen and (   min--moz-device-pixel-ratio: 2),
only screen and (     -o-min-device-pixel-ratio: 2/1),
only screen and (        min-device-pixel-ratio: 2),
only screen and (                min-resolution: 192dpi),
only screen and (                min-resolution: 2dppx) {
  /* Retina-specific stuff here */
  .gamepad {
    max-width: 450px;
    max-height: 800px;
  }
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
  height: 50%;
  box-sizing: content-box;
}

.controls {
  display: flex;
  flex-direction: column;
  padding-top: 30px;
  flex-grow: 1;

  &__btn {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #333;
    color: #b1b1b1;
    user-select: none;

    &.pressed {
      box-shadow: inset 0 0 4px 3px #111;
    }
  }

  &__menu {
    padding-bottom: 20px;
  }

  &__start {
    width: 100px;
    height: 30px;
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
