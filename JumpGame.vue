<template>
  <div class="game-container" @keydown="handleKeyDown" @keyup="handleKeyUp" tabindex="0">
    <div class="player" :style="{ bottom: playerY + 'px' }"></div>
    <div class="obstacle" :style="{ left: obstacleX + 'px' }"></div>
    <div class="score">Score: {{ score }}</div>
    <div v-if="gameOver" class="game-over">
      Game Over! Final Score: {{ score }}
      <button @click="restartGame">Restart</button>
    </div>
    <!-- Controls for jumping and increasing level -->
    <div class="controls">
      <button @click="jump">Jump (Button)</button>
      <button @click="levelUpManually">Increase Level</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      score: 0,
      gameOver: false,
      obstacleX: 600,
      playerY: 0,
      jumpVelocity: 0,
      isJumping: false,
      jumpCharge: 0,
      maxJumpCharge: 15,
      gameLoopId: null,
      level: 1,
      obstacleSpeed: 4,
      levelThreshold: 10,
      chargeInterval: null
    }
  },
  methods: {
    gameLoop() {
      if (this.gameOver) return;

      // Move obstacle left
      this.obstacleX -= this.obstacleSpeed; 
      if (this.obstacleX < -50) {
        this.obstacleX = 600; // reset position
        this.incrementScore();
        if (this.score % this.levelThreshold === 0) {
          this.levelUp();
        }
      }

      // Handle jump physics
      if (this.isJumping) {
        this.playerY += this.jumpVelocity;
        this.jumpVelocity -= 0.5;

        if (this.playerY <= 0) {
          this.playerY = 0;
          this.isJumping = false;
        }
      }

      // Check collision
      if (this.checkCollision()) {
        this.endGame();
        return;
      }

      this.gameLoopId = requestAnimationFrame(this.gameLoop);
    },
    checkCollision() {
      const playerLeft = 50;
      const playerRight = 100;
      const playerBottom = this.playerY;
      const playerTop = this.playerY + 50;

      const obstacleLeft = this.obstacleX;
      const obstacleRight = this.obstacleX + 50;
      const obstacleBottom = 0;
      const obstacleTop = 50;

      const horizontalOverlap = 
        playerRight > obstacleLeft && playerLeft < obstacleRight;
      const verticalOverlap =
        playerBottom < obstacleTop && playerTop > obstacleBottom;

      return horizontalOverlap && verticalOverlap;
    },
    endGame() {
      this.gameOver = true;
      cancelAnimationFrame(this.gameLoopId);
    },
    incrementScore() {
      this.score += 1;
    },
    handleKeyDown(e) {
      if (e.code === 'Space') {
        if (!this.isJumping && !this.gameOver) {
          this.jumpCharge = 0;
          this.chargeInterval = setInterval(() => {
            if (this.jumpCharge < this.maxJumpCharge) {
              this.jumpCharge++;
            }
          }, 50);
        }
      }
    },
    handleKeyUp(e) {
      if (e.code === 'Space') {
        if (!this.isJumping && !this.gameOver) {
          clearInterval(this.chargeInterval);
          this.jump();
        }
      }
    },
    jump() {
      this.jumpVelocity = 8 + this.jumpCharge * 0.5;
      this.isJumping = true;
      this.jumpCharge = 0;
    },
    levelUp() {
      this.level += 1;
      this.obstacleSpeed += 1;
    },
    restartGame() {
      this.score = 0;
      this.gameOver = false;
      this.obstacleX = 600;
      this.playerY = 0;
      this.level = 1;
      this.obstacleSpeed = 4;
      this.gameLoop();
    },
    jumpViaButton() {
      if (!this.isJumping && !this.gameOver) {
        this.jump();
      }
    },
    levelUpManually() {
      this.levelUp();
    }
  },
  mounted() {
    this.$el.focus();
    this.gameLoop();
  }
}
</script>

<style scoped>
.game-container {
  position: relative;
  width: 600px;
  height: 200px;
  border: 2px solid #000;
  overflow: hidden;
  background: linear-gradient(to right, #87ceeb, #f0e68c);
  margin: 50px auto;
  outline: none;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0,0,0,0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-family: Arial, sans-serif;
}

.player {
  position: absolute;
  left: 50px;
  width: 50px;
  height: 50px;
  background-color: #3498db;
  border-radius: 10px;
  bottom: 0;
  transition: bottom 0.1s;
}

.obstacle {
  position: absolute;
  bottom: 0;
  width: 50px;
  height: 50px;
  background-color: #e74c3c;
  border-radius: 8px;
  transition: left 0.1s;
}

.score {
  position: absolute;
  top: 10px;
  left: 20px;
  font-size: 20px;
  font-weight: bold;
  color: #2c3e50;
}

.game-over {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(255,255,255,0.9);
  padding: 20px;
  border-radius: 15px;
  text-align: center;
  font-size: 24px;
  box-shadow: 0 0 20px rgba(0,0,0,0.3);
}

/* Control buttons styling */
.controls {
  margin-top: 10px;
  display: flex;
  gap: 10px;
}

button {
  padding: 8px 12px;
  font-size: 14px;
  background-color: #2ecc71;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #27ae60;
}
</style>