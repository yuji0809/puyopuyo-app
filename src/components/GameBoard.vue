<template>
  <div>
    <!-- スタートボタン -->
    <div v-if="!gameStarted" class="start-button">
      <button @click="startGame">Start Game</button>
    </div>
    <div v-else>
      <button @click="endGame">End Game</button>
    </div>
    <!-- ゲームボード -->
    <div class="game-board" v-if="gameStarted && !gameOver">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
        <Puyo v-for="(cell, cellIndex) in row" :key="cellIndex" :type="getPuyoType(rowIndex, cellIndex)" />
      </div>
    </div>
    <!-- ゲームオーバーメッセージ -->
    <div v-if="gameOver" class="game-over-message">
      <h1>Game Over</h1>
      <p>Press Space to Restart</p>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, onBeforeUnmount } from 'vue';
import Puyo from './Puyo.vue';

interface Cell {
  id: string;
  type: string | null;
}

export default defineComponent({
  components: {
    Puyo,
  },
  setup() {
    const rows = 12;
    const cols = 6;
    const board = ref<Cell[][]>([]);
    const activePuyo = ref<{ x: number; y: number; type: string }>({ x: 0, y: 0, type: 'red' });
    const gameOver = ref(false);
    const gameStarted = ref(false);
    const score = ref(0);
    const dropInterval = 1000; // 1秒ごとにぷよを落とす
    let gameInterval: number | null = null;

    const initBoard = () => {
      board.value = [];
      for (let r = 0; r < rows; r++) {
        const row: Cell[] = [];
        for (let c = 0; c < cols; c++) {
          row.push({ id: `${r}-${c}`, type: null });
        }
        board.value.push(row);
      }
    };

    const startGame = () => {
      gameOver.value = false;
      gameStarted.value = true;
      score.value = 0;
      initBoard();
      spawnNewPuyo();
      if (gameInterval) {
        clearInterval(gameInterval);
      }
      gameInterval = setInterval(dropPuyo, dropInterval); // 1秒ごとにぷよを落とす
    };

    const endGame = () => {
      gameOver.value = true;
      gameStarted.value = false;
      if (gameInterval) {
        clearInterval(gameInterval);
      }
    };

    const spawnNewPuyo = () => {
      activePuyo.value = { x: Math.floor(cols / 2), y: 0, type: getRandomPuyoType() };
      if (board.value[activePuyo.value.y][activePuyo.value.x].type !== null) {
        gameOver.value = true;
        gameStarted.value = false;
        if (gameInterval) {
          clearInterval(gameInterval);
        }
      }
    };

    const getRandomPuyoType = () => {
      const puyoTypes = ['red', 'green', 'blue', 'yellow', 'purple'];
      return puyoTypes[Math.floor(Math.random() * puyoTypes.length)];
    };

    const dropPuyo = () => {
      if (activePuyo.value.y < rows - 1 && board.value[activePuyo.value.y + 1][activePuyo.value.x].type === null) {
        activePuyo.value.y += 1;
      } else {
        board.value[activePuyo.value.y][activePuyo.value.x].type = activePuyo.value.type;
        checkForMatches();
        spawnNewPuyo();
      }
    };

    const movePuyoLeft = () => {
      if (activePuyo.value.x > 0 && board.value[activePuyo.value.y][activePuyo.value.x - 1].type === null) {
        activePuyo.value.x -= 1;
      }
    };

    const movePuyoRight = () => {
      if (activePuyo.value.x < cols - 1 && board.value[activePuyo.value.y][activePuyo.value.x + 1].type === null) {
        activePuyo.value.x += 1;
      }
    };

    const rotatePuyo = () => {
      // 単体ぷよの回転は意味がないため未実装
    };

    const checkForMatches = () => {
      const matchedPuyos: { x: number, y: number }[] = [];
      const directions = [
        { x: 0, y: 1 },
        { x: 1, y: 0 },
        { x: 0, y: -1 },
        { x: -1, y: 0 },
      ];

      const isMatch = (x: number, y: number, type: string | null) => {
        if (type === null) return false;
        const visited: { [key: string]: boolean } = {};
        const queue: { x: number, y: number }[] = [{ x, y }];
        const matchGroup: { x: number, y: number }[] = [];

        while (queue.length > 0) {
          const { x, y } = queue.shift()!;
          const key = `${x}-${y}`;
          if (visited[key] || board.value[y][x].type !== type) continue;
          visited[key] = true;
          matchGroup.push({ x, y });

          for (const dir of directions) {
            const newX = x + dir.x;
            const newY = y + dir.y;
            if (newX >= 0 && newY >= 0 && newX < cols && newY < rows) {
              queue.push({ x: newX, y: newY });
            }
          }
        }

        if (matchGroup.length >= 4) {
          matchedPuyos.push(...matchGroup);
          return true;
        }

        return false;
      };

      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          if (board.value[y][x].type !== null) {
            isMatch(x, y, board.value[y][x].type);
          }
        }
      }

      matchedPuyos.forEach(puyo => {
        board.value[puyo.y][puyo.x].type = null;
      });

      if (matchedPuyos.length > 0) {
        score.value += matchedPuyos.length * 10;
      }
    };

    const handleKeydown = (event: KeyboardEvent) => {
      if (!gameStarted.value) return;
      switch (event.key) {
        case 'ArrowLeft':
          movePuyoLeft();
          break;
        case 'ArrowRight':
          movePuyoRight();
          break;
        case 'ArrowDown':
          dropPuyo();
          break;
        case 'ArrowUp':
          rotatePuyo();
          break;
        case ' ':
          if (gameOver.value) {
            startGame();
          }
          break;
      }
    };

    const getPuyoType = (rowIndex: number, cellIndex: number): string | null => {
      if (rowIndex === activePuyo.value.y && cellIndex === activePuyo.value.x) {
        return activePuyo.value.type;
      }
      return board.value[rowIndex][cellIndex].type;
    };

    onMounted(() => {
      window.addEventListener('keydown', handleKeydown);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('keydown', handleKeydown);
      if (gameInterval) {
        clearInterval(gameInterval);
      }
    });

    return {
      board,
      activePuyo,
      gameOver,
      gameStarted,
      score,
      startGame,
      endGame,
      getPuyoType,
    };
  },
});
</script>

<style scoped>
.game-board {
  display: grid;
  grid-template-rows: repeat(12, 40px);
  grid-template-columns: repeat(6, 40px);
  gap: 1px;
  background-color: #000;
  padding: 10px;
  border: 2px solid #ccc;
  position: relative;
}

.row {
  display: contents;
}

.puyo {
  width: 40px;
  height: 40px;
}

.start-button {
  text-align: center;
  margin-top: 50px;
}

.game-over-message {
  text-align: center;
  margin-top: 50px;
}
</style>
