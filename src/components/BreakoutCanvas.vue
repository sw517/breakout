<template>
  <canvas ref="canvas" :width="canvas.width" :height="canvas.height" />
</template>

<script>
import { setInterval, clearInterval } from 'timers';

export default {
  name: 'Canvas',
  data() {
    return {
      ctx: null,
      interval: null,
      rightPressed: false,
      leftPressed: false,
      canvas: {
        width: 250,
        height: 300,
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
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d');
    this.initBallProperties();
    this.initPaddleProperties();
    this.initBricks();
    document.addEventListener('keydown', this.onKeyDown);
    document.addEventListener('keyup', this.onKeyUp);
    this.interval = setInterval(this.draw, 10);
  },
  methods: {
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
    getCanvasWidth() {
      return this.canvas.width;
    },
    getCanvasHeight() {
      return this.canvas.height;
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.movePaddle();
      this.drawBall();
      this.drawPaddle();
      this.drawBricks();
      this.detectBrickCollision();
    },
    drawBall() {
      this.ctx.beginPath();
      this.ctx.arc(this.ball.xPos, this.ball.yPos, this.ball.diameter, 0, Math.PI * 2);
      this.ctx.fillStyle = '#0095DD';
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
      this.ctx.fillStyle = '#0095DD';
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
            this.ctx.fillStyle = '#0095DD';
            this.ctx.fill();
            this.ctx.closePath();
          }
        }
      }
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
        clearInterval(this.interval);
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
            }
          }
        }
      }
    },
    onKeyDown(e) {
      if (e.key === 'Right' || e.key === 'ArrowRight') {
        this.rightPressed = true;
      }
      if (e.key === 'Left' || e.key === 'ArrowLeft') {
        this.leftPressed = true;
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
canvas {
  background-color: #eee;
}
</style>
