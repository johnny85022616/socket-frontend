<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const roomId = route.params.roomId  // 從路由參數取得房間名稱

// 用房間名稱組出連線網址，例如 ws://localhost:8080?room=room1
const WS_URL = `ws://localhost:8080?room=${encodeURIComponent(roomId)}`

const status = ref('連線中...')   // 顯示目前的連線狀態
const messages = ref([])          // 收到的所有訊息（畫面上的訊息列表）
const input = ref('')             // 輸入框目前打的文字，跟 <input> 雙向綁定
let ws = null                     // WebSocket 連線實例，不需要響應式，用一般變數即可

// 送出訊息：把輸入框內容透過 WebSocket 送給 server
function sendMessage() {
  const text = input.value.trim()
  // 沒有輸入內容，或連線還沒建立好，就不送出
  if (!text || !ws || ws.readyState !== WebSocket.OPEN) return
  ws.send(text)
  input.value = ''
}

// 元件掛載時建立 WebSocket 連線，並註冊各種事件的處理方式
onMounted(() => {
  ws = new WebSocket(WS_URL)

  // 連線成功建立
  ws.onopen = () => {
    status.value = '已連線'
  }
  // 連線關閉（server 關閉、網路中斷等）
  ws.onclose = () => {
    status.value = '連線已中斷'
  }
  // 連線發生錯誤
  ws.onerror = () => {
    status.value = '連線錯誤'
  }
  // 收到 server 傳來的訊息（自己送出的訊息，server 廣播回來也會觸發這裡）
  ws.onmessage = (event) => {
    messages.value.push(event.data)
  }
})

// 元件卸載前（例如切換頁面）主動關閉連線，避免殘留連線占用資源
onBeforeUnmount(() => {
  ws?.close()
})
</script>

<template>
  <div class="app">
    <router-link to="/">← 回首頁</router-link>
    <h1>房間：{{ roomId }}</h1>
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
