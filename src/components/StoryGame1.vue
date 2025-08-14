<template>
  <div class="story-container">
    <!-- Loading state -->
    <div v-if="loading" class="loading-overlay">
      <div class="loading-content">
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

    <!-- Main content -->
    <div v-if="!loading && !error">
      <!-- Full-page background image -->
      <div 
        class="story-background" 
        :style="{ backgroundImage: currentPassage.image ? `url(/images/${currentPassage.image})` : 'none' }"
      ></div>
      
      <!-- Translucent text overlay -->
      <div class="text-overlay">
        <h1 v-if="shouldShowName">{{ currentPassage.name }}</h1>
        <p class="story-text">{{ currentPassage.text }}</p>
        <div class="choices-container">
          <button 
            v-for="(choice, index) in currentPassage.choices" 
            :key="index"
            @click="goToPassage(choice.next)"
            class="choice-button"
          >
            {{ choice.text }}
          </button>
        </div>
        <div v-if="!currentPassage.choices || currentPassage.choices.length === 0" class="end-container">
          <p class="end-text">The story ends here...</p>
          <button @click="restartGame" class="restart-button">Play Again</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      storyData: null,
      currentPid: null,
      loading: true,
      error: false
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
    goToPassage(nextPid) {
      this.currentPid = nextPid;
    },
    restartGame() {
      this.currentPid = this.storyData.startnode;
    },
    async loadStoryData() {
      try {
        this.loading = true;
        this.error = false;
        const response = await fetch("/story1.json");
        this.storyData = await response.json();
        this.currentPid = this.storyData.startnode;
      } catch (err) {
        console.error("Failed to load story data:", err);
        this.error = true;
      } finally {
        this.loading = false;
      }
    }
  },
  created() {
    this.loadStoryData();
  }
};
</script>

<style scoped>
.story-container {
  position: relative;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
}

/* Background image styling */
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
  filter: brightness(0.8);
}

/* Text overlay styling */
.text-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(34, 33, 33, 0.75);
  padding: 2rem;
  color: white;
  z-index: 1;
  max-height: 50vh;
  overflow-y: auto;
  border-top: 2px solid #4CAF50;
}

.story-text {
  font-size: 1.1rem;
  line-height: 1.6;
  margin-bottom: 1.5rem;
}

.choices-container {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.choice-button {
  background: rgba(76, 175, 80, 0.8);
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 1rem;
  text-align: left;
}

.choice-button:hover {
  background: rgba(69, 160, 73, 0.9);
  transform: translateX(5px);
}

.end-container {
  text-align: center;
  margin-top: 1rem;
}

.end-text {
  font-size: 1.2rem;
  margin-bottom: 1rem;
}

.restart-button {
  background: #ff5555;
  color: white;
  padding: 12px 25px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.restart-button:hover {
  background: #ff3333;
  transform: scale(1.05);
}

/* Loading and error states */
.loading-overlay,
.error-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.loading-content,
.error-content {
  color: white;
  text-align: center;
  padding: 2rem;
  max-width: 500px;
}

.error-content button {
  margin-top: 1rem;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .text-overlay {
    padding: 1.5rem;
    max-height: 60vh;
  }
  
  .story-text {
    font-size: 1rem;
  }
  
  .choice-button {
    padding: 10px 15px;
  }
}
</style>