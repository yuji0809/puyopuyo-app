<template>
  <div id="app">
    <ScoreBoard />
    <GameBoard />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import GameBoard from './components/GameBoard.vue';
import ScoreBoard from './components/ScoreBoard.vue';

export default defineComponent({
  components: {
    GameBoard,
    ScoreBoard,
  },
});
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
