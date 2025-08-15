<template>
  <div class="level-intro" :style="introStyle">
    <div class="intro-background"></div>
    <div class="content-wrapper">
      <div class="level-banner" :style="bannerStyle">LEVEL {{ level }}</div>
      <div class="level-title" :style="titleStyle">{{ titles[level] }}</div>
      <div class="subtitle" v-if="subtitles[level]">{{ subtitles[level] }}</div>
      <button @click="startLevel" class="continue-btn">
        <span class="btn-text">{{ buttonText }}</span>
        <span class="btn-arrow">→</span>
      </button>
    </div>
    <div class="particles" v-if="showParticles"></div>
  </div>
</template>

<script>
export default {
  props: {
    level: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      showParticles: false,
      buttonText: 'BEGIN',
      levelColors: ['#ff5555', '#4CAF50', '#2196F3', '#9C27B0', '#FF9800'],
      titles: {
    1: "The Otherworld Gate",
    2: "Survive in the New Society", 
    3: "Solve the Glowing Rock Mystery",
    4: "Yet To Be Revealed",
    5: "Final Reckoning"
    },
    subtitles: {
    1: "A vortex swallows you whole - welcome to another realm...",
    2: "Strange rules govern this twisted civilization...",
    3: "The luminous stone hums with forbidden knowledge...",
    4: "Something ancient stirs in the shadows...",
    5: "Your choices culminate in this decisive hour..."
    },
      bgImages: [
        "linear-gradient(135deg, #1a1a2e 0%, #16213e 100%)",
        "linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%)",
        "linear-gradient(135deg, #200122 0%, #6f0000 100%)",
        "linear-gradient(135deg, #3a1c71 0%, #d76d77 50%, #ffaf7b 100%)",
        "linear-gradient(135deg, #000000 0%, #434343 100%)"
      ]
    }
  },
  computed: {
    introStyle() {
      return {
        '--level-color': this.levelColors[this.level - 1],
        '--level-bg': this.bgImages[this.level - 1]
      }
    },
    bannerStyle() {
      return {
        'text-shadow': `0 0 15px ${this.levelColors[this.level - 1]}, 
                       0 0 30px ${this.levelColors[this.level - 1]}`
      }
    },
    titleStyle() {
      return {
        'background': `linear-gradient(90deg, 
                      ${this.levelColors[this.level - 1]}, 
                      #ffffff)`,
        '-webkit-background-clip': 'text',
        '-webkit-text-fill-color': 'transparent'
      }
    }
  },
  methods: {
    startLevel() {
      this.buttonText = 'LOADING...';
      setTimeout(() => {
        this.$emit('continue');
      }, 500);
    },
    animateParticles() {
      this.showParticles = true;
    }
  },
  mounted() {
    setTimeout(this.animateParticles, 300);
  }
}
</script>

<style scoped>
.level-intro {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: var(--level-bg);
  display: grid;
  place-items: center;
  z-index: 1000;
  overflow: hidden;
}

.intro-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(
    ellipse at center,
    rgba(255, 255, 255, 0.1) 0%,
    rgba(0, 0, 0, 0.8) 100%
  );
  z-index: 1;
}

.content-wrapper {
  position: relative;
  z-index: 2;
  text-align: center;
  max-width: 90%;
}

.level-banner {
  font-size: clamp(4rem, 15vw, 8rem);
  font-weight: 900;
  color: var(--level-color);
  margin-bottom: 1rem;
  animation: pulse 2s infinite, float 6s ease-in-out infinite;
  font-family: 'Audiowide', cursive;
}

.level-title {
  font-size: clamp(1.5rem, 5vw, 3rem);
  font-weight: 700;
  margin-bottom: 2rem;
  animation: fadeIn 1.5s ease-out;
  font-family: 'Orbitron', sans-serif;
}

.subtitle {
  font-size: clamp(1rem, 3vw, 1.5rem);
  margin-bottom: 3rem;
  opacity: 0.8;
  animation: fadeIn 2s ease-out;
  font-style: italic;
}

.continue-btn {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: 2px solid var(--level-color);
  padding: 1rem 2.5rem;
  border-radius: 50px;
  cursor: pointer;
  font-size: 1.2rem;
  font-weight: bold;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin: 0 auto;
  animation: fadeIn 2.5s ease-out;
  backdrop-filter: blur(5px);
}

.continue-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.btn-arrow {
  transition: transform 0.3s ease;
}

.continue-btn:hover .btn-arrow {
  transform: translateX(5px);
}

.particles {
  position: absolute;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.1) 1px,
    transparent 1px
  );
  background-size: 20px 20px;
  animation: particlesMove 20s linear infinite;
  z-index: 1;
}

/* Animations */
@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes particlesMove {
  from { background-position: 0 0; }
  to { background-position: 100px 100px; }
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .level-banner {
    font-size: 4rem;
  }
  
  .level-title {
    font-size: 1.5rem;
  }
  
  .subtitle {
    font-size: 1rem;
  }
  
  .continue-btn {
    padding: 0.8rem 2rem;
    font-size: 1rem;
  }
}
</style>