<template>
  <div>
    <h1>MQTT消息接收</h1>
    <div>
      <h2>最新接收到的消息：</h2>
      <p>{{ receivedMessage }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import mqtt from 'mqtt'

const client = ref(null);
const receivedMessage = ref('')

client.value = mqtt.connect('ws://broker.emqx.io:8083/mqtt', {
  clientId: 'vue3_client_' + Math.random().toString(16).substr(2, 8)
})

client.value.on('connect', () => {
  console.log('MQTT连接成功')
  client.value.subscribe('vue3/demo', (error) => {
    if (!error) {
      console.log('MQTT订阅成功')
    }
  })
})

// 添加消息接收处理
client.value.on('message', (topic, message) => {
  receivedMessage.value = message.toString()
  console.log('MQTT收到消息:', message.toString())
})

onUnmounted(() => {
  client.value.end()
})
</script>

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