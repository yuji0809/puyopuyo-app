<template>
    <div class="score-board">
      <h2>Score: {{ score }}</h2>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref, provide } from 'vue';
  
  export default defineComponent({
    setup() {
      const score = ref(0);
  
      const updateScore = (points: number) => {
        score.value += points;
      };
  
      provide('updateScore', updateScore);
  
      return {
        score,
      };
    },
  });
  </script>
  
  <style scoped>
  .score-board {
    font-size: 2em;
    margin: 10px;
  }
  </style>
  