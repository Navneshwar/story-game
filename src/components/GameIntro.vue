<template>
  <div class="intro-container" :class="{ 'fade-out': animationComplete }">
    <div class="intro-content">
      <h1 class="title" ref="title">Oozh - adventure to </h1>
      
      <div class="level-container">
        <div 
          v-for="level in levels" 
          :key="level.number"
          class="level-card"
          :style="{ 'transition-delay': `${level.number * 0.3}s` }"
        >
          <div class="level-number">LEVEL {{ level.number }}</div>
          <div class="level-difficulty">Difficulty: {{ level.difficulty }}</div>
          <div class="level-desc">{{ level.description }}</div>
        </div>
      </div>

      <button 
        class="start-button" 
        @click="startGame"
        :class="{ pulse: showStartButton }"
      >
        BEGIN JOURNEY
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      animationComplete: false,
      showStartButton: false,
      levels: [
        {
          number: 1,
          difficulty: "Easy",
          description: "Find your way out of the mystical forest"
        },
        {
          number: 2,
          difficulty: "Medium",
          description: "Survive in the new society"
        },
        {
          number: 3,
          difficulty: "Hard",
          description: "Solve the glowing rock mystery"
        },
        {
          number: 4,
          difficulty: "Very Hard",
          description: "Yet To Be revealed"
        },
        {
          number: 5,
          difficulty: "Nightmare",
          description: "Face the final challenge"
        }
      ]
    }
  },
  mounted() {
    this.animateIntro();
  },
  methods: {
    animateIntro() {
      // Title animation
      setTimeout(() => {
        this.$refs.title.classList.add('animate');
        
        // Show start button after all animations
        setTimeout(() => {
          this.showStartButton = true;
        }, 2500);
      }, 500);
    },
    startGame() {
      this.animationComplete = true;
      setTimeout(() => {
        this.$emit('start-game');
      }, 500);
    }
  }
}
</script>

<style scoped>
.intro-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  transition: opacity 0.5s ease;
}

.intro-content {
  max-width: 800px;
  text-align: center;
  padding: 2rem;
}

.title {
  font-size: 3.5rem;
  margin-bottom: 3rem;
  opacity: 0;
  transform: translateY(30px);
  transition: all 1s ease;
}

.title.animate {
  opacity: 1;
  transform: translateY(0);
  text-shadow: 0 0 10px #00ffff, 0 0 20px #0088ff;
}

.level-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1.5rem;
  margin-bottom: 3rem;
}

.level-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 1.5rem;
  width: 200px;
  opacity: 0;
  transform: scale(0.8);
  transition: all 0.5s ease;
}

.level-card:nth-child(1) { transition-delay: 0.3s; }
.level-card:nth-child(2) { transition-delay: 0.6s; }
.level-card:nth-child(3) { transition-delay: 0.9s; }
.level-card:nth-child(4) { transition-delay: 1.2s; }
.level-card:nth-child(5) { transition-delay: 1.5s; }

.title.animate ~ .level-container .level-card {
  opacity: 1;
  transform: scale(1);
}

.level-number {
  font-weight: bold;
  font-size: 1.3rem;
  color: #00ffff;
  margin-bottom: 0.5rem;
}

.level-difficulty {
  font-style: italic;
  margin-bottom: 0.5rem;
}

.start-button {
  background: #ff5555;
  color: white;
  border: none;
  padding: 1rem 2rem;
  font-size: 1.2rem;
  border-radius: 50px;
  cursor: pointer;
  opacity: 0;
  transform: scale(0.9);
  transition: all 0.5s ease;
}

.start-button.pulse {
  opacity: 1;
  transform: scale(1);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 85, 85, 0.7); }
  70% { box-shadow: 0 0 0 15px rgba(255, 85, 85, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 85, 85, 0); }
}

.fade-out {
  opacity: 0;
  pointer-events: none;
}
</style>