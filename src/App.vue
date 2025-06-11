<template>
  <div class="app">
    <header>
      <h1>Draw Lots!</h1>
    </header>
    <main>
      <div class="controls">
        <div class="input-group">
          <label for="maxNumber">Maximum Number:</label>
          <input 
            type="number" 
            id="maxNumber" 
            v-model.number="maxNumber" 
            min="1" 
            max="100"
            @input="validateInput"
          >
        </div>
        <button 
          class="draw-button" 
          @click="drawLot" 
          :disabled="isDrawing"
        >
          {{ isDrawing ? 'Drawing...' : 'Draw a Lot' }}
        </button>
      </div>
      
      <div class="result-container" v-if="currentNumber !== null">
        <div class="result-number" :class="{ 'animate': isDrawing }">
          {{ currentNumber }}
        </div>
      </div>

      <div class="history-container">
        <div class="history-header">
          <h2>History</h2>
          <button 
            class="clear-button" 
            @click="clearHistory"
            :disabled="drawnNumbers.length === 0"
            :class="{ 'disabled': drawnNumbers.length === 0 }"
          >
            <span class="button-icon">🗑️</span>
            <span class="button-text">Clear History</span>
          </button>
        </div>
        <div class="history-list">
          <div 
            v-for="(number, index) in drawnNumbers" 
            :key="index"
            class="history-item"
            :style="{
              '--gradient-opacity': 1 - (index * 0.1),
              '--gradient-delay': `${index * 0.1}s`
            }"
          >
            {{ number }}
          </div>
        </div>
      </div>

      <div class="tips-container">
        <h3>Tips</h3>
        <ul class="tips-list">
          <li>Set your maximum number (1-100) to define the range</li>
          <li>Click "Draw" to randomly select a number</li>
          <li>Each number can only be drawn once</li>
          <li>Use "Clear History" to start fresh</li>
          <li>Numbers are drawn without replacement</li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue'

export default {
  name: 'App',
  setup() {
    const maxNumber = ref(10)
    const currentNumber = ref(null)
    const drawnNumbers = ref([])
    const isDrawing = ref(false)
    const availableNumbers = ref([])

    const validateInput = () => {
      if (maxNumber.value < 1) maxNumber.value = 1
      if (maxNumber.value > 100) maxNumber.value = 100
    }

    const resetAvailableNumbers = () => {
      availableNumbers.value = Array.from(
        { length: maxNumber.value }, 
        (_, i) => i + 1
      )
    }

    const clearHistory = () => {
      drawnNumbers.value = []
      resetAvailableNumbers()
      currentNumber.value = null
    }

    const drawLot = () => {
      if (isDrawing.value || availableNumbers.value.length === 0) return
      
      isDrawing.value = true
      currentNumber.value = null

      // Quick draw animation
      const duration = 1.5
      const startTime = Date.now()
      
      const animate = () => {
        const elapsed = Date.now() - startTime
        const progress = Math.min(elapsed / (duration * 1000), 1)
        
        if (progress < 1) {
          currentNumber.value = Math.floor(Math.random() * maxNumber.value) + 1
          requestAnimationFrame(animate)
        } else {
          const randomIndex = Math.floor(Math.random() * availableNumbers.value.length)
          currentNumber.value = availableNumbers.value[randomIndex]
          drawnNumbers.value.unshift(currentNumber.value)
          availableNumbers.value.splice(randomIndex, 1)
          isDrawing.value = false
        }
      }

      requestAnimationFrame(animate)
    }

    onMounted(() => {
      resetAvailableNumbers()
    })

    watch(maxNumber, () => {
      resetAvailableNumbers()
    })

    return {
      maxNumber,
      currentNumber,
      drawnNumbers,
      isDrawing,
      drawLot,
      validateInput,
      clearHistory
    }
  }
}
</script>

<style lang="scss">
:root {
  --primary-color: #4a90e2;
  --secondary-color: #f39c12;
  --background-color: #f5f6fa;
  --text-color: #2c3e50;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Arial', sans-serif;
  background-color: var(--background-color);
  color: var(--text-color);
}

.app {
  min-height: 100vh;
  padding: 2rem;
  max-width: 800px;
  margin: 0 auto;
}

header {
  text-align: center;
  margin-bottom: 2rem;
  
  h1 {
    font-size: 2.5rem;
    color: var(--primary-color);
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  }
}

.controls {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;

  .input-group {
    margin-bottom: 1rem;
    
    label {
      display: block;
      margin-bottom: 0.5rem;
      font-weight: bold;
    }
    
    input {
      width: 100%;
      padding: 0.8rem;
      border: 2px solid #e0e0e0;
      border-radius: 6px;
      font-size: 1rem;
      transition: border-color 0.3s ease;
      
      &:focus {
        outline: none;
        border-color: var(--primary-color);
      }
    }
  }
}

.draw-button {
  width: 100%;
  padding: 1rem;
  background-color: var(--primary-color);
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.2s ease, background-color 0.2s ease;
  
  &:hover:not(:disabled) {
    transform: translateY(-2px);
    background-color: darken(#4a90e2, 10%);
  }
  
  &:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
}

.result-container {
  text-align: center;
  margin: 2rem 0;
  
  .result-number {
    font-size: 4rem;
    font-weight: bold;
    color: var(--secondary-color);
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    
    &.animate {
      animation: bounce 0.5s ease infinite;
    }
  }
}

.history-container {
  margin-top: 2rem;
  background: white;
  border-radius: 10px;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  .history-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;

    h2 {
      margin: 0;
      color: var(--primary-color);
      font-size: 1.5rem;
    }
  }

  .history-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    max-height: 200px;
    overflow-y: auto;
    padding: 0.5rem;
  }

  .history-item {
    background: var(--primary-color);
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 5px;
    font-weight: bold;
    opacity: var(--gradient-opacity);
    animation: fadeIn 0.3s ease-out forwards;
    animation-delay: var(--gradient-delay);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s ease;

    &:hover {
      transform: translateY(-2px);
    }
  }
}

.tips-container {
  margin-top: 2rem;
  background: white;
  border-radius: 10px;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  h3 {
    color: var(--primary-color);
    margin: 0 0 1rem 0;
    font-size: 1.2rem;
  }

  .tips-list {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      padding: 0.5rem 0;
      color: #666;
      position: relative;
      padding-left: 1.5rem;

      &:before {
        content: "•";
        color: var(--primary-color);
        position: absolute;
        left: 0;
        font-size: 1.2rem;
      }

      &:not(:last-child) {
        border-bottom: 1px solid #eee;
      }
    }
  }
}

.clear-button {
  background: linear-gradient(145deg, #e74c3c, #c0392b);
  color: white;
  border: none;
  padding: 0.3rem 0.6rem;
  border-radius: 6px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  box-shadow: 0 4px 15px rgba(231, 76, 60, 0.2);
  position: relative;
  overflow: hidden;

  &:hover:not(.disabled) {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(231, 76, 60, 0.3);
    background: linear-gradient(145deg, #c0392b, #e74c3c);

    .button-icon {
      transform: rotate(15deg);
    }
  }

  &:active:not(.disabled) {
    transform: translateY(0);
    box-shadow: 0 2px 10px rgba(231, 76, 60, 0.2);
  }

  &.disabled {
    background: #ccc;
    cursor: not-allowed;
    box-shadow: none;
    opacity: 0.7;
  }

  .button-icon {
    font-size: 1.1rem;
    transition: transform 0.3s ease;
  }

  .button-text {
    font-weight: 600;
    letter-spacing: 0.5px;
  }

  &::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(
      45deg,
      transparent 0%,
      rgba(255, 255, 255, 0.1) 50%,
      transparent 100%
    );
    transform: translateX(-100%);
    transition: transform 0.6s ease;
  }

  &:hover:not(.disabled)::after {
    transform: translateX(100%);
  }
}

@keyframes bounce {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: var(--gradient-opacity);
    transform: translateY(0);
  }
}
</style> 