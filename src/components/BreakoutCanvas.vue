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
import { setInterval, clearInterval, setTimeout } from 'timers';

export default {
  name: 'Canvas',
  data() {
    return this.getInitialState();
  },
  mounted() {
    this.initGameProperties();
    document.addEventListener('keydown', this.onKeyDown);
    document.addEventListener('keyup', this.onKeyUp);
    this.$nextTick(() => {
      this.draw();
    });
  },
  methods: {
    /**
     * Return the initial state of the component.
     */
    getInitialState() {
      return {
        ctx: null,
        interval: null,
        rightPressed: false,
        leftPressed: false,
        gameActive: false,
        instructionsVisible: true,
        countdown: 3,
        countdownVisible: false,
        gameEnded: false,
        score: 0,
        canvas: {
          width: 250,
          height: 280,
          color: '#ccc',
          background: '#333333',
        },
        ball: {
          diameter: 5,
          xPos: null,
          yPos: null,
          dx: 0,
          dy: 0,
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
    /**
     * Set the X and Y position of the ball to the center
     * position in the canvas.
     */
    initBallProperties() {
      this.ball.xPos = this.getCanvasWidth() / 2;
      this.ball.yPos = this.getCanvasHeight() / 2;
    },
    /**
     * Set the X position of the paddle to be in the center
     * of the canvas.
     */
    initPaddleProperties() {
      this.paddle.xPos = (this.getCanvasWidth() - this.paddle.width) / 2;
    },
    /**
     * Create an object for each brick shown in the game. The object
     * contains key for the X and Y position of the block and the status.
     * The status is 1 if the block is visible, 0 if not.
     */
    initBricks() {
      for (let c = 0; c < this.bricks.columns; c += 1) {
        this.bricks.instances[c] = [];
        for (let r = 0; r < this.bricks.rows; r += 1) {
          this.bricks.instances[c][r] = { x: 0, y: 0, status: 1 };
        }
      }
    },
    /**
     * Set the context of the canvas and get the width and height
     * of the parent container surrounding the canvas so these values
     * can be set on the canvas itself.
     * Call the methods to initiate the properties of the ball, paddle
     * and bricks.
     */
    initGameProperties() {
      this.ctx = this.$refs.canvas.getContext('2d');
      this.canvas.width = this.getParentContainerWidth();
      this.canvas.height = this.getParentContainerHeight();
      this.initBallProperties();
      this.initPaddleProperties();
      this.initBricks();
    },
    /**
     * Move ball by 'speed' value in pixels.
     */
    changeBallSpeed(speed) {
      this.ball.dx = this.ball.dx < 0 ? -speed : speed;
      this.ball.dy = this.ball.dy < 0 ? -speed : speed;
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
      this.changeBallSpeed(2);
      this.interval = setInterval(this.draw, 10);
    },
    draw() {
      this.ctx.clearRect(0, 0, this.getCanvasWidth(), this.getCanvasHeight());
      this.movePaddle();
      this.drawBall();
      this.drawPaddle();
      this.drawBricks();
      this.detectBrickCollision();
      this.drawScore();
      if (this.instructionsVisible) {
        this.drawIntructions();
      }
      if (this.countdownVisible) {
        this.drawCountdown();
      }
      if (this.gameEnded) {
        this.drawGameOver();
      }
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
      this.ctx.font = '12px Courier';
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fillText(`Score: ${this.score}`, 8, 20);
    },
    drawIntructions() {
      this.ctx.beginPath();
      this.ctx.rect(0, 0, this.getCanvasWidth(), 58);
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fill();
      this.ctx.lineWidth = 3;
      this.ctx.strokeStyle = this.canvas.background;
      this.ctx.stroke();
      this.ctx.closePath();
      this.ctx.font = '12px Courier';
      this.ctx.fillStyle = this.canvas.background;
      this.ctx.fillText('Spacebar to start', 8, 20);
      this.ctx.fillText('Left/Right arrows to move', 8, 40);
    },
    drawCountdown() {
      this.ctx.font = '18px Courier';
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fillText(
        this.countdown,
        (this.getCanvasWidth() / 2) - 5,
        (this.getCanvasHeight() / 2) + 30,
      );
    },
    drawGameOver() {
      this.ctx.font = '18px Courier';
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fillText(
        'Game Over',
        (this.getCanvasWidth() / 2) - 50,
        (this.getCanvasHeight() / 2) + 30,
      );

      this.ctx.font = '10px Courier';
      this.ctx.fillStyle = this.canvas.color;
      this.ctx.fillText(
        '(Space to restart)',
        (this.getCanvasWidth() / 2) - 56,
        (this.getCanvasHeight() / 2) + 60,
      );
    },
    /**
     * Check to see if the ball has collided with the sides
     * of the canvas.
     */
    checkBoundaries() {
      // Check if ball hits left or right side of canvas.
      if (this.ball.xPos < this.ball.diameter
        || this.ball.xPos > this.canvas.width - this.ball.diameter) {
        this.ball.dx = -(this.ball.dx);
      }
      // Check if ball hits top of canvas.
      if (this.ball.yPos < this.ball.diameter) {
        this.ball.dy = -(this.ball.dy);
        // Check if ball hits paddle.
      } else if (this.ball.yPos > this.canvas.height - this.ball.diameter - this.paddle.height
        && this.ball.yPos < this.canvas.height - this.ball.diameter) {
        // Check if ball is between the sides of the paddle.
        if (this.ball.xPos > this.paddle.xPos
          && this.ball.xPos < this.paddle.xPos + this.paddle.width) {
          this.ball.dy = -(this.ball.dy);
        }
        // Check if ball hits bottom of canvas.
      } else if (this.ball.yPos > this.canvas.height - this.ball.diameter) {
        this.gameOver();
      }
    },
    /**
     * Update the position of the paddle.
     */
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
    /**
     * Check if the ball has collided with any of the bricks
     * visible on the canvas.
     */
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
    startCountdown() {
      this.countdownVisible = true;
      const countdownInterval = setInterval(() => {
        this.draw();
        this.countdown -= 1;
        if (this.countdown === 0) {
          this.countdown = 'GO';
        }
      }, 2000 / 3);
      setTimeout(() => {
        this.instructionsVisible = false;
        this.countdownVisible = false;
        clearInterval(countdownInterval);
        this.initGame();
      }, 3000);
    },
    /**
     * Check if all the bricks have been hit and are no longer visible.
     */
    checkEndGame() {
      if (this.score === this.bricks.columns * this.bricks.rows) {
        this.gameOver();
      }
    },
    /**
     * Stop the game.
     */
    gameOver() {
      this.gameEnded = true;
      clearInterval(this.interval);
      this.gameActive = false;
    },
    /**
     * Reasign the initial data properties to the component
     * and update certain values after resetting.
     */
    resetGame() {
      Object.assign(this.$data, this.getInitialState());
      this.initGameProperties();
    },
    onSpacePressed() {
      if (!this.instructionsVisible) {
        this.resetGame();
        this.$nextTick(() => {
          this.draw();
        });
      } else {
        this.startCountdown();
      }
    },
    onKeyDown(e) {
      if (e.key === 'Right' || e.key === 'ArrowRight') {
        this.rightPressed = true;
      }
      if (e.key === 'Left' || e.key === 'ArrowLeft') {
        this.leftPressed = true;
      }
      // Spacebar
      if (e.keyCode === 32) {
        if (this.gameActive === false) {
          this.onSpacePressed();
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
