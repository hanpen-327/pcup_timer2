<template>
  <div class="timer" :class="backgroundClass">
    <div class="display">{{ formattedTime }}</div>
    <div class="controls">
      <button @click="toggle">{{ isRunning ? '一時停止' : '開始' }}</button>
      <button @click="reset">リセット（動作中でも継続）</button>
      <button @click="resetAndStop">リセットして停止</button>
    </div>
    <div class="key-settings">
      <label>
        リセットキー: 
        <input 
          v-model="resetKey" 
          type="text" 
          maxlength="1" 
          placeholder="k"
          @keydown="captureKey"
        />
      </label>
      <span class="key-hint">現在: {{ resetKey || 'k' }}</span>
    </div>
    <!-- 通知表示エリア -->
    <transition name="notification">
      <div v-if="notification" class="notification">
        {{ notification }}
      </div>
    </transition>
  </div>
</template>

<script>
import { ref, computed, onUnmounted, onMounted } from 'vue'

export default {
  name: 'TimerComponent',
  setup () {
    const elapsed = ref(0) // ミリ秒単位の経過時間
    const isRunning = ref(false)
    const intervalId = ref(null)
    const resetKey = ref('k') // リセットキーの設定
    const colorState = ref(0) // 0: 通常, 1: 黄色, 2: 赤色
    const notification = ref('') // 通知メッセージ
    let startTime = 0 // Date.now() - elapsed の形で使う
    let notificationTimer = null // 通知を自動で消すタイマー

    const start = () => {
      if (isRunning.value) return
      // 続きから開始できるように startTime を補正
      startTime = Date.now() - elapsed.value
      // 高精度表示のため更新間隔は 16ms に（約60fps）
      intervalId.value = setInterval(() => {
        elapsed.value = Date.now() - startTime
        // 0.01秒（10ミリ秒）になったら黄色に変更
        if (elapsed.value >= 10 && colorState.value === 0) {
          colorState.value = 1
        }
      }, 16)
      isRunning.value = true
    }

    const pause = () => {
      if (intervalId.value) {
        clearInterval(intervalId.value)
        intervalId.value = null
      }
      isRunning.value = false
    }

    const toggle = () => {
      if (isRunning.value) pause()
      else start()
    }

    // 動作中でもリセットしてそのままカウントを継続する
    const reset = () => {
      // 30秒(30000ミリ秒)未満の場合
      if (elapsed.value < 30000) {
        // 黄色の状態なら赤色に変更
        if (colorState.value === 1) {
          colorState.value = 2
        }
        return
      }
      
      // 30秒以上の場合はリセット
      elapsed.value = 0
      colorState.value = 0
      if (isRunning.value) {
        // 走行中なら startTime を現在に合わせることで 0 から継続
        startTime = Date.now()
      }
    }

    // リセットして停止したい場合
    const resetAndStop = () => {
      // 30秒以内の場合
      if (elapsed.value < 30000) {
        // 黄色の状態なら赤色に変更
        if (colorState.value === 1) {
          colorState.value = 2
        }
        return
      }
      
      // 30秒以上の場合はリセットして停止
      pause()
      elapsed.value = 0
      colorState.value = 0
    }

    // キーボードイベントハンドラ
    const handleKeyPress = (event) => {
      const key = resetKey.value.toLowerCase()
      if (event.key.toLowerCase() === key) {
        reset()
      }
    }

    // キー入力をキャプチャして設定
    const captureKey = (event) => {
      event.preventDefault()
      if (event.key.length === 1) {
        resetKey.value = event.key.toLowerCase()
      }
    }

    // 通知を表示する関数
    const showNotification = (message) => {
      notification.value = message
      // 既存のタイマーをクリア
      if (notificationTimer) {
        clearTimeout(notificationTimer)
      }
      // 3秒後に通知を消す
      notificationTimer = setTimeout(() => {
        notification.value = ''
      }, 3000)
    }

    // ==================== メッセージ受信処理 ====================
    
    // window.postMessage からの通知を受信
    const handleMessage = (event) => {
      if (event.data && event.data.type === 'notify') {
        console.log('通知を受信:', event.data.message)
        const message = event.data.message || '通知を受信しました'
        // 通知を表示
        showNotification(message)
        // リセットボタンと同じ挙動を実行
        reset()
      }
    }

    // ==================== ライフサイクル ====================

    onMounted(() => {
      // キーボードイベントリスナーを登録
      window.addEventListener('keydown', handleKeyPress)
      // postMessage イベントリスナーを登録
      window.addEventListener('message', handleMessage)
    })

    onUnmounted(() => {
      // タイマーのクリーンアップ
      if (intervalId.value) clearInterval(intervalId.value)
      if (notificationTimer) clearTimeout(notificationTimer)
      // イベントリスナーの削除
      window.removeEventListener('keydown', handleKeyPress)
      window.removeEventListener('message', handleMessage)
    })

    // 表示フォーマット: mm:ss.mmm （必要なら hh 部分も拡張できます）
    const formattedTime = computed(() => {
      const totalMs = Math.max(0, Math.floor(elapsed.value))
      const ms = totalMs % 1000
      const totalSeconds = Math.floor(totalMs / 1000)
      const seconds = totalSeconds % 60
      const minutes = Math.floor(totalSeconds / 60)
      const pad = (n, d = 2) => String(n).padStart(d, '0')
      const padMs = (n) => String(n).padStart(3, '0')
      return `${pad(minutes)}:${pad(seconds)}.${padMs(ms)}`
    })

    // 背景色のクラスを計算
    const backgroundClass = computed(() => {
      if (colorState.value === 1) return 'bg-yellow'
      if (colorState.value === 2) return 'bg-red'
      return ''
    })

    return {
      isRunning,
      formattedTime,
      toggle,
      reset,
      resetAndStop,
      resetKey,
      captureKey,
      backgroundClass,
      notification
    }
  }
}
</script>

<style scoped>
.timer {
  display: inline-block;
  text-align: center;
  font-family: sans-serif;
}
.display {
  font-size: 2rem;
  margin-bottom: 0.5rem;
  letter-spacing: 0.05em;
}
.controls > button {
  margin: 0 0.25rem;
  padding: 0.5rem 0.75rem;
  cursor: pointer;
}
.key-settings {
  margin-top: 1rem;
  padding: 0.5rem;
  border-top: 1px solid #ccc;
}
.key-settings label {
  display: block;
  margin-bottom: 0.5rem;
}
.key-settings input {
  width: 3rem;
  text-align: center;
  padding: 0.25rem;
  margin: 0 0.5rem;
  font-size: 1rem;
  text-transform: lowercase;
}
.key-hint {
  color: #666;
  font-size: 0.9rem;
  margin-left: 0.5rem;
}
.bg-yellow {
  background-color: #ffeb3b;
  transition: background-color 0.3s;
}
.bg-red {
  background-color: #f44336;
  transition: background-color 0.3s;
}
.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background-color: #4caf50;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  font-size: 1rem;
}
.notification-enter-active, .notification-leave-active {
  transition: all 0.3s ease;
}
.notification-enter-from {
  opacity: 0;
  transform: translateY(-20px);
}
.notification-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}
</style>