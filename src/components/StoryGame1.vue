<template>
  <div class="story-container">
    <!-- Level Intro Component -->
    <LevelIntro 
      v-if="showLevelIntro"
      :level="currentLevel"
      @continue="showLevelIntro = false"
    />

    <!-- Loading state -->
    <div v-if="loading" class="loading-overlay">
      <div class="loading-content">
        <div class="spinner"></div>
        Loading story...
      </div>
    </div>

    <!-- Error state -->
    <div v-if="error" class="error-overlay">
      <div class="error-content">
        Failed to load story data. Please refresh the page.
        <button @click="loadStoryData">Retry</button>
      </div>
    </div>

    <!-- Transition wrapper for animations -->
    <transition :name="transitionName" mode="out-in">
      <!-- Main content -->
      <div v-if="!loading && !error && !showLevelIntro" :key="currentPid">
        <!-- Preloaded background image -->
        <div 
          class="story-background" 
          :style="{ backgroundImage: currentBgImage ? `url(${currentBgImage})` : 'none' }"
        ></div>
        
        <!-- Translucent text overlay with transition -->
        <transition name="fade">
          <div class="text-overlay">
            <h1 v-if="shouldShowName">{{ currentPassage.name }}</h1>
            <p class="story-text">{{ currentPassage.text }}</p>
            <div class="choices-container">
              <transition-group name="choice" tag="div">
                <button 
                  v-for="(choice, index) in currentPassage.choices" 
                  :key="choice.next"
                  @click="goToPassage(choice.next)"
                  class="choice-button"
                  :style="{ transitionDelay: `${index * 0.1}s` }"
                >
                  {{ choice.text }}
                </button>
              </transition-group>
            </div>
            <div v-if="!currentPassage.choices || currentPassage.choices.length === 0" class="end-container">
              <p class="end-text">Level {{ currentLevel }} ends here...</p>
              <button 
                @click="handleLevelEnd" 
                class="level-complete-button"
                :class="{ 'level1-button': currentLevel === 1, 'restart-button': currentLevel !== 1 }"
              >
                {{ currentLevel === 1 ? 'Continue to Level 2' : 'Play Again' }}
              </button>
            </div>
          </div>
        </transition>
      </div>
    </transition>
  </div>
</template>

<script>
import LevelIntro from './LevelIntro.vue';

export default {
  components: {
    LevelIntro
  },
  data() {
    return {
      storyData: null,
      currentPid: null,
      loading: true,
      error: false,
      currentLevel: 1,
      showLevelIntro: true,
      currentBgImage: null,
      nextBgImage: null,
      transitionName: 'fade',
      imageCache: new Map()
    };
  },
  computed: {
    currentPassage() {
      if (!this.storyData?.passages) return {};
      return this.storyData.passages.find(p => p.pid === this.currentPid) || {};
    },
    shouldShowName() {
      return this.currentPid !== "1" && this.currentPid !== "14";
    }
  },
  methods: {
    async goToPassage(nextPid) {
      // Set transition direction based on PID (for slide effects)
      this.transitionName = parseInt(nextPid) > parseInt(this.currentPid) ? 'slide-left' : 'slide-right';
      
      // Preload next image
      const nextPassage = this.storyData.passages.find(p => p.pid === nextPid);
      if (nextPassage?.image) {
        await this.preloadImage(nextPassage.image);
      }
      
      this.currentPid = nextPid;
      this.checkLevelCompletion();
    },
    async preloadImage(imagePath) {
      if (this.imageCache.has(imagePath)) {
        this.currentBgImage = this.imageCache.get(imagePath);
        return;
      }

      return new Promise((resolve) => {
        const img = new Image();
        img.src = `/images/${imagePath}`;
        img.onload = () => {
          this.imageCache.set(imagePath, img.src);
          this.currentBgImage = img.src;
          resolve();
        };
        img.onerror = () => {
          console.error(`Failed to load image: ${imagePath}`);
          this.currentBgImage = null;
          resolve();
        };
      });
    },
    async loadAllImages() {
      if (!this.storyData?.passages) return;
      
      const imageUrls = this.storyData.passages
        .filter(p => p.image)
        .map(p => `/images/${p.image}`);
      
      await Promise.all(imageUrls.map(url => {
        return new Promise((resolve) => {
          const img = new Image();
          img.src = url;
          img.onload = () => {
            this.imageCache.set(url, url);
            resolve();
          };
          img.onerror = () => resolve();
        });
      }));
    },
    restartGame() {
      this.currentPid = this.storyData.startnode;
      this.currentLevel = 1;
      this.showLevelIntro = true;
    },
    async loadStoryData() {
      try {
        this.loading = true;
        this.error = false;
        const response = await fetch("/story1.json");
        this.storyData = await response.json();
        this.currentPid = this.storyData.startnode;
        
        // Preload first image
        const firstPassage = this.currentPassage;
        if (firstPassage?.image) {
          await this.preloadImage(firstPassage.image);
        }
        
        // Start background loading of all images
        this.loadAllImages();
      } catch (err) {
        console.error("Failed to load story data:", err);
        this.error = true;
      } finally {
        this.loading = false;
      }
    },
    handleLevelComplete() {
      this.currentLevel++;
      this.showLevelIntro = true;
    },
    checkLevelCompletion() {
      const levelEndNodes = {
        1: "13", // PID 13 marks end of Level 1
        // Add more level end markers as needed
      };

      if (this.currentPid === levelEndNodes[this.currentLevel]) {
        this.handleLevelComplete();
      }
    },
    handleLevelEnd() {
      if (this.currentLevel === 1) {
        this.$emit('level-complete', 2); // Emit event to parent to switch to Level 2
      } else {
        this.restartGame();
      }
    }
  },
  created() {
    this.loadStoryData();
  }
};
</script>

<style scoped>
/* Main container */
.story-container {
  position: relative;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Background image with fade effect */
.story-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  z-index: 1;
  filter: brightness(0.8) contrast(1.1);
  transition: opacity 0.7s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: opacity;
}

/* Text overlay with glass morphism effect */
.text-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(34, 33, 33, 0.85);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  padding: 2rem;
  color: white;
  z-index: 2;
  max-height: 55vh;
  overflow-y: auto;
  border-top: 2px solid #4CAF50;
  box-shadow: 0 -10px 30px rgba(0, 0, 0, 0.3);
}

/* Story text styling */
.story-text {
  font-size: 1.2rem;
  line-height: 1.7;
  margin-bottom: 1.8rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

/* Choices container */
.choices-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* Choice buttons with animated entrance */
.choice-button {
  background: linear-gradient(135deg, rgba(76, 175, 80, 0.9) 0%, rgba(69, 160, 73, 0.9) 100%);
  color: white;
  border: none;
  padding: 14px 20px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  font-size: 1.05rem;
  text-align: left;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  transform-origin: left center;
}

.choice-button:hover {
  background: linear-gradient(135deg, rgba(76, 175, 80, 1) 0%, rgba(69, 160, 73, 1) 100%);
  transform: translateX(8px) scale(1.03);
  box-shadow: 0 5px 15px rgba(76, 175, 80, 0.4);
}

/* End level container */
.end-container {
  text-align: center;
  margin-top: 2rem;
  padding-top: 1.5rem;
  border-top: 1px dashed rgba(255, 255, 255, 0.2);
}

.end-text {
  font-size: 1.3rem;
  margin-bottom: 1.5rem;
  font-weight: 500;
  letter-spacing: 0.5px;
}

/* Level complete button with shine effect */
.level-complete-button {
  padding: 14px 28px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1.1rem;
  font-weight: bold;
  transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1);
  position: relative;
  overflow: hidden;
  min-width: 200px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.level1-button {
  background: linear-gradient(to right, #4CAF50, #2E7D32);
  color: white;
}

.level1-button:hover {
  background: linear-gradient(to right, #43A047, #1B5E20);
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 20px rgba(76, 175, 80, 0.3);
}

.level1-button::after {
  content: '';
  position: absolute;
  top: -50%;
  left: -60%;
  width: 200%;
  height: 200%;
  background: linear-gradient(
    to bottom right,
    transparent 45%,
    rgba(255, 255, 255, 0.4) 50%,
    transparent 55%
  );
  transform: rotate(30deg);
  animation: shine 2.5s infinite;
  opacity: 0.7;
}

.restart-button {
  background: linear-gradient(to right, #ff5555, #d32f2f);
  color: white;
}

.restart-button:hover {
  background: linear-gradient(to right, #f44336, #b71c1c);
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 20px rgba(255, 85, 85, 0.3);
}

/* Loading and error states */
.loading-overlay,
.error-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.95);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  backdrop-filter: blur(5px);
}

.loading-content,
.error-content {
  color: white;
  text-align: center;
  padding: 2.5rem;
  max-width: 500px;
  background: rgba(30, 30, 30, 0.8);
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
}

.error-content button {
  margin-top: 1.5rem;
  padding: 12px 24px;
  background: linear-gradient(to right, #4CAF50, #2E7D32);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.error-content button:hover {
  background: linear-gradient(to right, #43A047, #1B5E20);
  transform: translateY(-2px);
}

/* Loading spinner animation */
.spinner {
  width: 50px;
  height: 50px;
  margin: 0 auto 1.5rem;
  border: 5px solid rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  border-top-color: #4CAF50;
  animation: spin 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
}

/* Animation keyframes */
@keyframes spin {
  to { transform: rotate(360deg); }
}

@keyframes shine {
  0% { transform: translateX(-100%) rotate(30deg); }
  100% { transform: translateX(200%) rotate(30deg); }
}

/* Transition animations */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.4s ease-out;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

.slide-left-enter-active, .slide-left-leave-active,
.slide-right-enter-active, .slide-right-leave-active {
  transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}
.slide-left-enter {
  transform: translateX(100%);
  opacity: 0;
}
.slide-left-leave-to {
  transform: translateX(-30%);
  opacity: 0;
}
.slide-right-enter {
  transform: translateX(-100%);
  opacity: 0;
}
.slide-right-leave-to {
  transform: translateX(30%);
  opacity: 0;
}

.choice-enter-active {
  transition: all 0.4s ease-out;
}
.choice-leave-active {
  transition: all 0.3s ease-in;
}
.choice-enter {
  opacity: 0;
  transform: translateY(30px) scale(0.9);
}
.choice-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}
.choice-move {
  transition: transform 0.4s ease;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .text-overlay {
    padding: 1.5rem;
    max-height: 60vh;
  }
  
  .story-text {
    font-size: 1.1rem;
    line-height: 1.6;
  }
  
  .choice-button {
    padding: 12px 18px;
    font-size: 1rem;
  }

  .end-text {
    font-size: 1.2rem;
  }

  .level-complete-button {
    padding: 12px 20px;
    font-size: 1rem;
    min-width: 180px;
  }
}

@media (max-width: 480px) {
  .text-overlay {
    padding: 1.2rem;
    max-height: 65vh;
  }

  .story-text {
    font-size: 1rem;
  }

  .choices-container {
    gap: 0.8rem;
  }

  .choice-button {
    padding: 10px 16px;
  }

  .loading-content,
  .error-content {
    padding: 1.5rem;
    width: 90%;
  }
}
</style>