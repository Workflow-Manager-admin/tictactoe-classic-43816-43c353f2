<template>
  <div class="ttt-container">
    <div class="ttt-turn-indicator">
      <!-- Shows winner, draw or whose turn it is -->
      <span v-if="winner === null && !isDraw">Turn: <strong :style="{ color: getColor(currentPlayer) }">{{ currentPlayer }}</strong></span>
      <span v-if="winner !== null">🎉 Winner: <strong :style="{ color: getColor(winner) }">{{ winner }}</strong>!</span>
      <span v-if="winner === null && isDraw">It's a draw!</span>
    </div>
    <div class="ttt-board">
      <div v-for="row in 3" :key="row" class="ttt-row">
        <button
          v-for="col in 3"
          :key="col"
          class="ttt-cell"
          :disabled="board[(row-1)*3+(col-1)] !== null || winner !== null"
          @click="makeMove((row-1)*3+(col-1))"
          :aria-label="'Cell ' + row + ',' + col"
        >
          <span :class="['ttt-cell-symbol', board[(row-1)*3+(col-1)] === 'X' ? 'ttt-x' : 'ttt-o']">
            {{ board[(row-1)*3+(col-1)] }}
          </span>
        </button>
      </div>
    </div>
    <div class="ttt-reset-btn">
      <button @click="resetGame" class="reset-btn" :disabled="isInitial">
        Reset Game
      </button>
    </div>
  </div>
</template>

<script setup>
// Minimalist, accessible TicTacToe logic (2-player, local only)
import { ref, computed } from 'vue'

const PLAYER_X = 'X'
const PLAYER_O = 'O'

// The 3x3 board is a flat array [0...8], 'X', 'O' or null
const board = ref(Array(9).fill(null))
const currentPlayer = ref(PLAYER_X)
const winner = ref(null) // X, O, or null
const isDraw = ref(false)

const isInitial = computed(() => board.value.every(cell => cell === null))

// PUBLIC_INTERFACE
function makeMove(index) {
  /** Handles a player's move and updates board/winner/draw state. */
  if (board.value[index] !== null || winner.value !== null) return

  // Reactively update cell
  board.value[index] = currentPlayer.value
  if (detectWinner()) {
    winner.value = currentPlayer.value
    isDraw.value = false
  } else if (board.value.every(cell => cell !== null)) {
    winner.value = null
    isDraw.value = true
  } else {
    // Switch turn
    currentPlayer.value = currentPlayer.value === PLAYER_X ? PLAYER_O : PLAYER_X
  }
}

// PUBLIC_INTERFACE
function resetGame() {
  /** Resets game state for a fresh start. */
  board.value = Array(9).fill(null)
  currentPlayer.value = PLAYER_X
  winner.value = null
  isDraw.value = false
}

// PUBLIC_INTERFACE
function getColor(player) {
  // X: primary, O: accent
  if (player === PLAYER_X) return '#6b0000'
  if (player === PLAYER_O) return '#2196f3'
  return '#000'
}

function detectWinner() {
  /** Returns true if the current player has won. */
  const winPatterns = [
    // rows
    [0, 1, 2],[3, 4, 5],[6, 7, 8],
    // columns
    [0, 3, 6],[1, 4, 7],[2, 5, 8],
    // diagonals
    [0, 4, 8],[2, 4, 6]
  ]
  return winPatterns.some(pattern =>
    pattern.every(idx => board.value[idx] === currentPlayer.value)
  )
}
</script>

<style scoped>
.ttt-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #fff;
  font-family: system-ui, sans-serif;
  padding: 0 12px;
}

.ttt-turn-indicator {
  font-size: 1.3rem;
  font-weight: 500;
  margin-bottom: 2rem;
  text-align: center;
  color: #000;
}

.ttt-board {
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 12px;
  background: #f8f8f8;
  box-shadow: 0 2px 8px rgba(0,0,0,0.03);
  padding: 16px 14px;
}

.ttt-row {
  display: flex;
}

.ttt-cell {
  background: #fff;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  width: 64px;
  height: 64px;
  margin: 0.17rem;
  font-size: 2.3rem;
  cursor: pointer;
  transition: border-color 0.16s, box-shadow 0.18s;
  outline: none;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #000;
  font-weight: bold;
  box-shadow: 0 1px 2px rgba(33,33,33,.06);
}
.ttt-cell:active {
  box-shadow: none;
  opacity: 0.88;
}
.ttt-cell:disabled {
  opacity: 0.68;
  cursor: default;
  background: #f4f4f4;
}

.ttt-cell-symbol {
  display: inline-block;
  width: 100%;
  text-align: center;
}

.ttt-x {
  color: #6b0000;
}

.ttt-o {
  color: #2196f3;
}

.ttt-reset-btn {
  margin-top: 2rem;
  text-align: center;
}
.reset-btn {
  background: #fff;
  border: 2px solid #2196f3;
  color: #2196f3;
  border-radius: 10px;
  min-width: 110px;
  padding: 0.7em 1.1em;
  font-size: 1.09rem;
  font-weight: 500;
  cursor: pointer;
  outline: none;
  transition: background 0.13s, color 0.14s, border-color 0.15s;
}
.reset-btn:disabled {
  opacity: 0.66;
  cursor: not-allowed;
}
.reset-btn:not(:disabled):hover, .reset-btn:focus-visible {
  background: #2196f3;
  color: #fff;
  border-color: #2196f3;
}
</style>
