<template>
  <div>
    <!-- <h1>MQTT消息接收</h1>
    <div>
      <h2>最新接收到的消息：</h2>
      <p>{{ receivedMessage }}</p>
    </div> -->
    <div class="chat-container">
      <div class="chat-header">
        <h1>聊天室</h1>
      </div>
      <div class="chat-messages">
        <div v-for="(item, index) in messageList" :key="index" :class="['message-item', item.isMe ? 'me' : 'other']">
          <div class="message-content">
            {{ item.message }}
          </div>
          <div class="message-time">
            {{ item.time }}
          </div>
        </div>
      </div>
      <div class="chat-input">
        <input v-model="message" placeholder="输入消息..." @keyup.enter="sendMessage" />
        <button @click="sendMessage">发送</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import mqtt from 'mqtt'

const client = ref<any>(null);
// const receivedMessage = ref('')
const messageList = ref<any[]>([])
// wss://broker.emqx.io:8084/mqtt
// ws://broker.emqx.io:8083/mqtt
client.value = mqtt.connect('wss://broker.emqx.io:8084/mqtt', {
  clientId: 'vue3_client_' + Math.random().toString(16).substr(2, 8),
  protocol: 'wss',  // 明确指定协议
  path: '/mqtt',   // 与vite.config.ts中的代理路径一致
  port: 8084,      // 代理目标端口
  rejectUnauthorized: false  // 如果使用自签名证书
})

client.value.on('connect', () => {
  console.log('连接成功')
  client.value.subscribe('vue3/demo', (error: any) => {
    if (!error) {
      console.log('订阅成功')
    }
  })
})

// 添加消息接收处理
client.value.on('message', (topic: any, message: any) => {
  // receivedMessage.value = message.toString()
  console.log('收到消息:', message.toString())
  messageList.value.push({
    message: message.toString(),
    time: new Date().toLocaleString(),
    isMe: false
  })
})


const clientSend = ref<any>(null);

clientSend.value = mqtt.connect("wss://broker.emqx.io:8084/mqtt", {
  clientId: 'vue3_client_' + Math.random().toString(16).substr(2, 8),
  protocol: 'wss',  // 明确指定协议
  path: '/mqtt',   // 与vite.config.ts中的代理路径一致
  port: 8084,      // 代理目标端口
  rejectUnauthorized: false  // 如果使用自签名证书
})
clientSend.value.on('connect', () => {
  console.log('连接成功')
  // 向主机发送
  clientSend.value.subscribe('vue3/machine')
})

clientSend.value.on('message', (topic: any, payload: any) => {
  console.log('订阅当前主题：', topic, payload.toString());
  // receivedMessage.value = payload.toString()
  messageList.value.push({
    time: new Date().toLocaleString(),
    message: payload.toString(),
    isMe: true
  })
})

const message = ref('') // 输入的消息
// 发布
const sendMessage = () => {
  if (message.value.trim()) {
    // 发布
    clientSend.value.publish('vue3/machine', message.value, {
      qos: 1,  // 设置QoS级别为1，确保消息至少传递一次
      retain: false
    }, (error: any) => {
      if (error) {
        console.log('发布失败：', error);
      } else {
        console.log('发布成功', message.value);
        message.value = ''  // 清空输入
      }
    })
  }
}




onUnmounted(() => {
  client.value.end()
  clientSend.value.end()
})
</script>
<style scoped>
.chat-container {
  /* min-width: 500px; */
  margin: 0 auto;
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  background-color: #f5f5f5;
}

.chat-header {
  background-color: #42b983;
  color: white;
  padding: 15px;
  text-align: center;
}

.chat-messages {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
}

.message-item {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
}

.message-item.me {
  align-items: flex-end;
}

.message-item.other {
  align-items: flex-start;
}

.message-content {
  max-width: 70%;
  padding: 10px;
  border-radius: 8px;
  position: relative;
}

.message-item.me .message-content {
  background-color: #42b983;
  color: white;
  border-bottom-right-radius: 0;
}

.message-item.other .message-content {
  background-color: white;
  color: #333;
  border-bottom-left-radius: 0;
}

.message-time {
  font-size: 12px;
  color: #999;
  margin-top: 5px;
}

.chat-input {
  display: flex;
  padding: 10px;
  background-color: white;
  border-top: 1px solid #ddd;
}

.chat-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-right: 10px;
}

.chat-input button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.chat-input button:hover {
  background-color: #3aa876;
}
</style>
<!-- 
<template>
  <div>
    <h1>MQTT消息接收</h1>
    <div>
      <h2>最新接收到的消息：</h2>
      <p>{{ receivedMessage }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import mqttClient from '@/utils/mqttClient'

const receivedMessage = ref('')

// 连接MQTT（使用相同的broker地址）
mqttClient.connect('ws://broker.emqx.io:8083/mqtt', {
  clientId: 'vue3_receiver_' + Math.random().toString(16).substr(2, 8)
})

// 订阅相同的主题
onMounted(() => {
  mqttClient.subscribe('vue3/demo', (msg) => {
    receivedMessage.value = msg
    console.log('Received message:', msg)
  })
})

// 断开连接
onUnmounted(() => {
  mqttClient.disconnect()
})
</script> -->