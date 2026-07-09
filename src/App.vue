<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const WS_URL = 'ws://localhost:8080'

const status = ref('連線中...')
const messages = ref([])
const input = ref('')
let ws = null

function sendMessage() {
  const text = input.value.trim()
  if (!text || !ws || ws.readyState !== WebSocket.OPEN) return
  ws.send(text)
  input.value = ''
}

onMounted(() => {
  ws = new WebSocket(WS_URL)

  ws.onopen = () => {
    status.value = '已連線'
  }
  ws.onclose = () => {
    status.value = '連線已中斷'
  }
  ws.onerror = () => {
    status.value = '連線錯誤'
  }
  ws.onmessage = (event) => {
    messages.value.push(event.data)
  }
})

onBeforeUnmount(() => {
  ws?.close()
})
</script>

<template>
  <div class="app">
    <h1>WebSocket 簡易聊天室</h1>
    <p class="status">{{ status }}</p>
    <div class="messages">
      <div v-for="(msg, i) in messages" :key="i">{{ msg }}</div>
    </div>
    <form @submit.prevent="sendMessage">
      <input v-model="input" type="text" placeholder="輸入訊息..." autocomplete="off" />
      <button type="submit">送出</button>
    </form>
  </div>
</template>

<style scoped>
.app {
  max-width: 500px;
  margin: 40px auto;
  font-family: sans-serif;
}
.status {
  font-size: 14px;
  color: gray;
}
.messages {
  border: 1px solid #ccc;
  height: 300px;
  overflow-y: auto;
  padding: 8px;
  margin-bottom: 8px;
  text-align: left;
}
form {
  display: flex;
  gap: 8px;
}
input {
  flex: 1;
  padding: 6px;
}
button {
  padding: 6px 12px;
}
</style>
