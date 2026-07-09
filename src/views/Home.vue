<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const roomName = ref('')

function joinRoom() {
  // 沒輸入房間名稱時，預設進入 lobby（跟後端 DEFAULT_ROOM 對應）
  const name = roomName.value.trim() || 'lobby'
  router.push(`/room/${encodeURIComponent(name)}`)
}
</script>

<template>
  <div class="home">
    <h1>WebSocket 聊天室</h1>
    <p>輸入房間名稱以加入，留空則進入預設房間</p>
    <form @submit.prevent="joinRoom">
      <input v-model="roomName" type="text" placeholder="房間名稱，留空則為 lobby" autocomplete="off" />
      <button type="submit">加入房間</button>
    </form>
  </div>
</template>

<style scoped>
.home {
  max-width: 500px;
  margin: 40px auto;
  font-family: sans-serif;
  text-align: center;
}
form {
  display: flex;
  gap: 8px;
  margin-top: 16px;
}
input {
  flex: 1;
  padding: 6px;
}
button {
  padding: 6px 12px;
}
</style>
