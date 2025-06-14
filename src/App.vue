<template>
  <div class="app">
    <header>
      <h1>Draw Lots</h1>
    </header>
    <main>
      <div class="controls">
        <div class="input-group">
          <label for="max">Max Number:</label>
          <input
            type="number"
            id="max"
            v-model.number="maxNumber"
            :disabled="isDrawing"
            @input="validateInput"
          >
        </div>
        <button
          class="draw-button"
          @click="drawLot"
          :disabled="isDrawing || !isValid"
        >
          {{ isDrawing ? 'Drawing...' : 'Draw' }}
        </button>
      </div>

      <div class="result" v-if="displayNumber !== null">
        <div class="number">{{ displayNumber }}</div>
      </div>

      <div class="history-section" v-if="drawnNumbers.length > 0">
        <div class="history-header">
          <h2>History</h2>
          <button 
            class="clear-button" 
            @click="clearHistory"
            :disabled="drawnNumbers.length === 0"
          >
            Clear History
          </button>
        </div>
        <div class="history">
          <div 
            v-for="item in drawnNumbers" 
            :key="item.id" 
            class="history-number" 
            :class="{ 'new-number': item.id === drawnNumbers[drawnNumbers.length - 1].id }"
            :style="{
              '--gradient-opacity': 1 - (drawnNumbers.length - 1 - drawnNumbers.indexOf(item)) / drawnNumbers.length,
              '--gradient-delay': `${drawnNumbers.indexOf(item) * 0.1}s`
            }"
          >
            {{ item.number }}
          </div>
        </div>
      </div>

      <div class="tips-container">
        <h3>Tips</h3>
        <ul class="tips-list">
          <li>Set your maximum number (1-100) to define the range</li>
          <li>Click <span class="button-text draw">Draw</span> to randomly select a number</li>
          <li>Numbers can be drawn multiple times</li>
          <li>Use <span class="button-text clear">Clear History</span> to start fresh</li>
          <li>Each draw is completely random and independent</li>
          <li>History shows the last 10 drawn numbers</li>
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
    const displayNumber = ref(null)
    const drawnNumbers = ref([])
    const isDrawing = ref(false)
    const isValid = ref(true)
    let historyId = 0

    const validateInput = () => {
      if (maxNumber.value < 1) maxNumber.value = 1
      if (maxNumber.value > 100) maxNumber.value = 100
      isValid.value = true
    }

    const clearHistory = () => {
      drawnNumbers.value = []
      currentNumber.value = null
      displayNumber.value = null
      historyId = 0
    }

    const drawLot = () => {
      if (isDrawing.value) return
      
      isDrawing.value = true
      let startTime = null
      const duration = 1000 // 1 second
      
      const animate = (timestamp) => {
        if (!startTime) startTime = timestamp
        const progress = timestamp - startTime
        
        if (progress < duration) {
          // During animation, show random numbers from the same range
          displayNumber.value = Math.floor(Math.random() * maxNumber.value) + 1
          requestAnimationFrame(animate)
        } else {
          // At the end of animation, generate the actual random number
          const finalNumber = Math.floor(Math.random() * maxNumber.value) + 1
          currentNumber.value = finalNumber
          displayNumber.value = finalNumber
          drawnNumbers.value.push({ id: historyId++, number: finalNumber })
          if (drawnNumbers.value.length > 10) {
            drawnNumbers.value.shift()
          }
          isDrawing.value = false
        }
      }
      
      requestAnimationFrame(animate)
    }

    onMounted(() => {
      validateInput()
    })

    watch(maxNumber, () => {
      validateInput()
    })

    return {
      maxNumber,
      currentNumber,
      displayNumber,
      drawnNumbers,
      isDrawing,
      drawLot,
      validateInput,
      clearHistory,
      isValid
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
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
  &:hover:not(:disabled) {
    transform: translateY(-2px);
    background-color: darken(#4a90e2, 10%);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  }
  
  &:disabled {
    background-color: #ccc;
    cursor: not-allowed;
    opacity: 0.7;
  }
}

.result {
  text-align: center;
  margin: 2rem 0;
  
  .number {
    font-size: 4rem;
    font-weight: bold;
    color: var(--secondary-color);
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    
    &.animate {
      animation: bounce 0.5s ease infinite;
    }
  }
}

.history-section {
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
}

.history {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  max-height: 200px;
  overflow-y: auto;
  padding: 0.5rem;
}

.history-number {
  display: inline-block;
  padding: 0.5rem 1rem;
  margin: 0.25rem;
  border-radius: 0.5rem;
  font-size: 1.25rem;
  font-weight: 600;
  color: white;
  background: linear-gradient(135deg, #1a5276, #154360);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  opacity: var(--gradient-opacity, 1);
}

.history-number.new-number {
  animation: slideIn 0.5s ease-out;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  opacity: 1;
}

.history-number::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.2), transparent);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.history-number:hover::before {
  opacity: 1;
}

@keyframes slideIn {
  0% {
    transform: translateX(100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
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
  background: #e74c3c;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
  &:hover:not(:disabled) {
    transform: translateY(-2px);
    background: #c0392b;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  }
  
  &:disabled {
    background: #ccc;
    cursor: not-allowed;
    opacity: 0.7;
  }
}

.button-text {
  display: inline-block;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-weight: 600;
  font-size: 0.9em;
  margin: 0 0.2rem;
  
  &.draw {
    background-color: var(--primary-color);
    color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  &.clear {
    background-color: #e74c3c;
    color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
}
</style> 