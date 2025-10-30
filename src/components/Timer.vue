<template>
  <div class="timer">
    <h1>タイマー</h1>
    <div class="time-display">
      {{ formattedTime }}
    </div>
    <div class="controls">
      <button 
        @click="startTimer" 
        :disabled="isRunning"
        class="btn btn-start"
      >
        開始
      </button>
      <button 
        @click="stopTimer" 
        :disabled="!isRunning"
        class="btn btn-stop"
      >
        停止
      </button>
      <button 
        @click="resetTimer"
        class="btn btn-reset"
      >
        リセット
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TimerComponent',
  data() {
    return {
      elapsedTime: 0,
      isRunning: false,
      intervalId: null
    }
  },
  computed: {
    formattedTime() {
      const totalCentiseconds = Math.floor(this.elapsedTime / 10)
      const minutes = Math.floor(totalCentiseconds / 6000)
      const seconds = Math.floor((totalCentiseconds % 6000) / 100)
      const centiseconds = totalCentiseconds % 100
      return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}.${String(centiseconds).padStart(2, '0')}`
    }
  },
  methods: {
    startTimer() {
      if (!this.isRunning) {
        this.isRunning = true
        const startTime = Date.now() - this.elapsedTime
        this.intervalId = setInterval(() => {
          this.elapsedTime = Date.now() - startTime
        }, 10)
      }
    },
    stopTimer() {
      if (this.isRunning) {
        this.isRunning = false
        clearInterval(this.intervalId)
        this.intervalId = null
      }
    },
    resetTimer() {
      this.stopTimer()
      this.elapsedTime = 0
    }
  },
  beforeUnmount() {
    if (this.intervalId) {
      clearInterval(this.intervalId)
      this.intervalId = null
      this.isRunning = false
    }
  }
}
</script>

<style scoped>
.timer {
  max-width: 400px;
  margin: 0 auto;
  padding: 40px 20px;
}

h1 {
  color: #2c3e50;
  margin-bottom: 30px;
  font-size: 2.5em;
}

.time-display {
  font-size: 4em;
  font-weight: bold;
  color: #42b983;
  margin: 40px 0;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.1em;
}

.controls {
  display: flex;
  gap: 15px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  padding: 15px 30px;
  font-size: 1.2em;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
  min-width: 100px;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-start {
  background-color: #42b983;
  color: white;
}

.btn-start:hover:not(:disabled) {
  background-color: #359268;
}

.btn-stop {
  background-color: #e74c3c;
  color: white;
}

.btn-stop:hover:not(:disabled) {
  background-color: #c0392b;
}

.btn-reset {
  background-color: #3498db;
  color: white;
}

.btn-reset:hover {
  background-color: #2980b9;
}
</style>
