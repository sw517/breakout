<template>
  <canvas
    ref="canvas"
    :width="canvas.width"
    :height="canvas.height"
    :style="{
      backgroundColor: canvas.background
    }"
  />
</template>

<script>
import { setInterval, clearInterval } from 'timers';

export default {
  name: 'Canvas',
  data() {
    return this.getInitialState();
  },
  mounted() {
    this.initGameProperties();
    document.addEventListener('keydown', this.onKeyDown);
    document.addEventListener('keyup', this.onKeyUp);
    this.draw();
  },
  methods: {
    getInitialState() {
      return {
        ctx: null,
        interval: null,
        rightPressed: false,
        leftPressed: false,
        gameActive: false,
        score: 0,
        canvas: {
          width: 250,
          height: 280,
          color: '#eeeeee',
          background: '#333333',
        },
        ball: {
          diameter: 8,
          xPos: null,
          yPos: null,
          dx: 2,
          dy: 2,
        },
        paddle: {
          width: 75,
          height: 10,
          xPos: null,
        },
        bricks: {
          columns: 5,
          rows: 3,
          width: 20,
          height: 20,
          padding: 10,
          xOffset: 30,
          yOffset: 30,
          instances: [],
        },
      };
    },
    initBallProperties() {
      this.ball.xPos = this.getCanvasWidth() / 2;
      this.ball.yPos = this.getCanvasHeight() / 2;
    },
    initPaddleProperties() {
      this.paddle.xPos = (this.getCanvasWidth() - this.paddle.width) / 2;
    },
    initBricks() {
      for (let c = 0; c < this.bricks.columns; c += 1) {
        this.bricks.instances[c] = [];
        for (let r = 0; r < this.bricks.rows; r += 1) {
          this.bricks.instances[c][r] = { x: 0, y: 0, status: 1 };
        }
      }
    },
    initGameProperties() {
      this.ctx = this.$refs.canvas.getContext('2d');
      this.canvas.width = this.getParentContainerWidth();
      this.canvas.height = this.getParentContainerHeight();
      this.initBallProperties();
      this.initPaddleProperties();
      this.initBricks();
    },
    getCanvasWidth() {
      return this.canvas.width;
    },
    getCanvasHeight() {
      return this.canvas.height;
    },
    getParentContainerWidth() {
      const styles = window.getComputedStyle(this.$el.parentElement);
      return styles.getPropertyValue('width').replace('px', '');
    },
    getParentContainerHeight() {
      const styles = window.getComputedStyle(this.$el.parentElement);
      return styles.getPropertyValue('height').replace('px', '');
    },
    initGame() {
      this.gameActive = true;
      this.interval = setInterval(this.draw, 10);
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.movePaddle();
      this.drawBall();
      this.drawPaddle();
      this.drawBricks();
      this.detectBrickCollision();
      this.drawScore();
    },
    drawBall() {
      this.ctx.beginPath();
      this.ctx.arc(this.ball.xPos, this.ball.yPos, this.ball.diameter, 0, Math.PI * 2);
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fill();
      this.ctx.closePath();
      this.ball.xPos += this.ball.dx;
      this.ball.yPos += this.ball.dy;
      this.checkBoundaries();
    },
    drawPaddle() {
      this.ctx.beginPath();
      this.ctx.rect(
        this.paddle.xPos,
        this.canvas.height - this.paddle.height,
        this.paddle.width,
        this.paddle.height,
      );
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fill();
      this.ctx.closePath();
    },
    drawBricks() {
      for (let c = 0; c < this.bricks.columns; c += 1) {
        for (let r = 0; r < this.bricks.rows; r += 1) {
          if (this.bricks.instances[c][r].status === 1) {
            const brickXPos = (c * (this.bricks.width + this.bricks.padding)) + this.bricks.xOffset;
            const brickYPos = (r * (this.bricks.height + this.bricks.padding))
              + this.bricks.yOffset;
            this.bricks.instances[c][r].x = brickXPos;
            this.bricks.instances[c][r].y = brickYPos;
            this.ctx.beginPath();
            this.ctx.rect(brickXPos, brickYPos, this.bricks.width, this.bricks.height);
            this.ctx.fillStyle = this.canvas.color;
            this.ctx.fill();
            this.ctx.closePath();
          }
        }
      }
    },
    drawScore() {
      this.ctx.font = '16px Arial';
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fillText(`Score: ${this.score}`, 8, 20);
    },
    checkBoundaries() {
      if (this.ball.xPos < this.ball.diameter
        || this.ball.xPos > this.canvas.width - this.ball.diameter) {
        this.ball.dx = -(this.ball.dx);
      }

      if (this.ball.yPos < this.ball.diameter) {
        this.ball.dy = -(this.ball.dy);
      } else if (this.ball.yPos > this.canvas.height - this.ball.diameter - this.paddle.height
        && this.ball.yPos < this.canvas.height - this.ball.diameter) {
        if (this.ball.xPos > this.paddle.xPos
          && this.ball.xPos < this.paddle.xPos + this.paddle.width) {
          this.ball.dy = -(this.ball.dy);
        }
      } else if (this.ball.yPos > this.canvas.height - this.ball.diameter) {
        this.gameOver();
      }
    },
    movePaddle() {
      if (this.rightPressed) {
        if (this.paddle.xPos + this.paddle.width < this.canvas.width) {
          this.paddle.xPos += 7;
        }
      } else if (this.leftPressed) {
        if (this.paddle.xPos >= 0) {
          this.paddle.xPos -= 7;
        }
      }
    },
    detectBrickCollision() {
      for (let c = 0; c < this.bricks.columns; c += 1) {
        for (let r = 0; r < this.bricks.rows; r += 1) {
          const brick = this.bricks.instances[c][r];
          if (brick.status === 1) {
            if (this.ball.xPos > brick.x
              && this.ball.xPos < brick.x + this.bricks.width
              && this.ball.yPos > brick.y
              && this.ball.yPos < brick.y + this.bricks.height) {
              this.ball.dy = -(this.ball.dy);
              brick.status = 0;
              this.score += 1;
              this.checkEndGame();
            }
          }
        }
      }
    },
    checkEndGame() {
      if (this.score === this.bricks.columns * this.bricks.rows) {
        this.gameOver();
      }
    },
    gameOver() {
      clearInterval(this.interval);
    },
    resetGame() {
      Object.assign(this.$data, this.initialState());
    },
    onKeyDown(e) {
      if (e.key === 'Right' || e.key === 'ArrowRight') {
        this.rightPressed = true;
        if (this.gameActive === false) {
          // this.resetGame();
          this.initGame();
        }
      }
      if (e.key === 'Left' || e.key === 'ArrowLeft') {
        this.leftPressed = true;
        if (this.gameActive === false) {
          // this.resetGame();
          this.initGame();
        }
      }
    },
    onKeyUp(e) {
      if (e.key === 'Right' || e.key === 'ArrowRight') {
        this.rightPressed = false;
      }
      if (e.key === 'Left' || e.key === 'ArrowLeft') {
        this.leftPressed = false;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
</style>
