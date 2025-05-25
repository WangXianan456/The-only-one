<template>
  <div class="chat-window">
    <div class="message-container" ref="messageContainer">
      <div v-for="msg in messages" :key="msg.id" :class="['message', msg.role]">
        <div class="avatar">
          <el-avatar :icon="msg.role === 'user' ? 'UserFilled' : 'Service'" />
        </div>
        <div class="content">
          <div class="markdown-body" v-html="markdownToHtml(msg.content)"></div>
        </div>
      </div>
    </div>
    
    <div class="input-area">
      <el-input
        v-model="inputMessage"
        type="textarea"
        :rows="3"
        placeholder="输入您的问题..."
        @keyup.enter.ctrl="sendMessage"
      />
      <el-button type="primary" @click="sendMessage" :loading="loading">
        发送
      </el-button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { marked } from 'marked'
import highlight from 'highlight.js'

const props = defineProps(['chatId'])
const messages = ref([])
const inputMessage = ref('')
const loading = ref(false)
const messageContainer = ref(null)

const markdownToHtml = (content) => {
  return marked(content, {
    highlight: (code, lang) => {
      return highlight.highlightAuto(code).value
    }
  })
}

const sendMessage = async () => {
  if (!inputMessage.value.trim()) return
  
  const userMessage = {
    id: Date.now(),
    role: 'user',
    content: inputMessage.value
  }
  
  messages.value.push(userMessage)
  inputMessage.value = ''
  loading.value = true

  try {
    const response = await fetch('http://localhost:3000/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: userMessage.content
      })
    })
    
    const data = await response.json()
    messages.value.push({
      id: Date.now(),
      role: 'assistant',
      content: data.response
    })
  } catch (error) {
    console.error('Error:', error)
  } finally {
    loading.value = false
  }
}

watch(() => messages.value, () => {
  setTimeout(() => {
    if (messageContainer.value) {
      messageContainer.value.scrollTop = messageContainer.value.scrollHeight
    }
  }, 100)
}, { deep: true })
</script>

<style scoped>
.chat-window {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.message-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.message {
  display: flex;
  margin-bottom: 20px;
}

.message.user {
  flex-direction: row-reverse;
}

.avatar {
  margin: 0 10px;
}

.content {
  max-width: 80%;
  padding: 12px;
  border-radius: 8px;
  background: #f4f4f5;
}

.message.user .content {
  background: #ecf5ff;
}

.input-area {
  padding: 20px;
  border-top: 1px solid #dcdfe6;
}

:deep(.markdown-body) {
  font-size: 14px;
  line-height: 1.6;
}

:deep(.markdown-body pre) {
  background-color: #282c34;
  border-radius: 6px;
  padding: 16px;
}
</style>
