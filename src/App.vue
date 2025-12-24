<template>
  <div class="game-container">
    <TankGame />
  </div>
</template>

<script setup lang="ts">
import TankGame from './components/TankGame.vue'
</script>

<style scoped>
.game-container {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
