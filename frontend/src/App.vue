<template>
  <div class="app-container">
    <el-container>
      <el-aside width="250px">
        <div class="sidebar">
          <div class="logo">AI Chat</div>
          <el-button type="primary" @click="newChat">新对话</el-button>
          <div class="chat-list">
            <div v-for="chat in chatHistory" 
                 :key="chat.id" 
                 @click="selectChat(chat.id)"
                 :class="['chat-item', { active: currentChatId === chat.id }]">
              {{ chat.title }}
            </div>
          </div>
        </div>
      </el-aside>
      <el-main>
        <ChatWindow :chatId="currentChatId" />
      </el-main>
    </el-container>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ChatWindow from './components/ChatWindow.vue'

const chatHistory = ref([])
const currentChatId = ref(null)

const newChat = () => {
  const chatId = Date.now()
  chatHistory.value.push({
    id: chatId,
    title: '新对话'
  })
  currentChatId.value = chatId
}

const selectChat = (id) => {
  currentChatId.value = id
}
</script>

<style scoped>
.app-container {
  height: 100vh;
}

.sidebar {
  padding: 20px;
  height: 100%;
  background: #f5f7fa;
}

.logo {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
}

.chat-list {
  margin-top: 20px;
}

.chat-item {
  padding: 10px;
  margin: 5px 0;
  cursor: pointer;
  border-radius: 4px;
}

.chat-item.active {
  background: #ecf5ff;
}
</style>
