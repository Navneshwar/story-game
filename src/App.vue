<template>
  <div id="app">
    <GameIntro v-if="showIntro" @start-game="startGame" />
    <StoryGame1 v-else />
  </div>
</template>

<script>
import GameIntro from './components/GameIntro.vue'
import StoryGame1 from './components/StoryGame1.vue'

export default {
  components: { GameIntro, StoryGame1 },
  data() {
    return {
      showIntro: true
    }
  },
  methods: {
    startGame() {
      this.showIntro = false;
    }
  }
}
</script>

<style>
/* Global styles can go here */
body {
  margin: 0;
  font-family: 'Arial', sans-serif;
  background-color: #121212;
  color: white;
}
</style>