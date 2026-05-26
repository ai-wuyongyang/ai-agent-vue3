<template>
  <el-main class="home-main">
    <div class="top-bar">
      <el-select v-model="selectedModel" class="model-select" placeholder="请选择模型">
        <el-option v-for="item in modelOptions" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </div>

    <div class="chat-shell">
      <BubbleList ref="bubbleListRef" class="chat-list" :list="messages" max-height="calc(100vh - 240px)"
        :auto-scroll="true" :show-back-button="true" />

      <div class="sender-wrap">
        <XSender ref="senderRef" :loading="isStreaming" placeholder="输入你的问题，点击回车体验前端模拟流式输出" @submit="handleSubmit" />
      </div>
    </div>
  </el-main>
</template>

<script setup lang="ts">
import { nextTick, onBeforeUnmount, ref } from 'vue'
import { BubbleList, XSender } from 'vue-element-plus-x'

interface ChatMessage {
  id: number
  role: 'user' | 'assistant'
  placement: 'start' | 'end'
  content: string
  loading?: boolean
  variant?: 'filled' | 'borderless' | 'outlined' | 'shadow'
  shape?: 'round' | 'corner'
}

interface SenderValue {
  text: string
}

interface SenderRef {
  getModelValue: () => SenderValue
  clear: () => void
}

interface BubbleListRef {
  scrollToBottom: (smooth?: boolean) => void
}

const senderRef = ref<SenderRef | null>(null)
const bubbleListRef = ref<BubbleListRef | null>(null)
const isStreaming = ref(false)
const selectedModel = ref('gpt-4o-mini')
const modelOptions = [
  { label: 'GPT-4o Mini', value: 'gpt-4o-mini' },
  { label: 'GPT-4.1', value: 'gpt-4.1' },
  { label: 'Claude 3.5 Sonnet', value: 'claude-3-5-sonnet' },
  { label: 'DeepSeek Chat', value: 'deepseek-chat' },
]
const messages = ref<ChatMessage[]>([])

let messageId = messages.value.length
let streamTimer: number | null = null

const scrollToBottom = () => {
  nextTick(() => bubbleListRef.value?.scrollToBottom(true))
}

const stopStreamingTimer = () => {
  if (streamTimer !== null) {
    window.clearTimeout(streamTimer)
    streamTimer = null
  }
}

const createAssistantReply = (question: string) => {
  return [
    `当前选择模型：${selectedModel.value}。`,
    `你刚刚问的是：“${question}”。`,
    '下面我用前端模拟流式渲染来回复你。',
    '第一步，先立即把用户消息插入消息列表。',
    '第二步，预创建一条 AI 消息，并把它的内容初始化为空字符串。',
    '第三步，通过定时器把完整回答拆成多个文本片段，依次追加到最后一条 AI 消息里。',
    '这样用户看到的效果就是内容一边生成、一边展示。',
    '如果后面你接真实接口，只需要把这个定时器替换成 SSE、Fetch Stream 或 WebSocket 的数据流即可。',
  ].join('')
}

const chunkText = (text: string) => {
  const chunks: string[] = []
  let cursor = 0

  while (cursor < text.length) {
    const size = Math.floor(Math.random() * 8) + 4
    chunks.push(text.slice(cursor, cursor + size))
    cursor += size
  }

  return chunks
}

const streamAssistantReply = (question: string) => {
  isStreaming.value = true

  messages.value.push({
    id: ++messageId,
    role: 'assistant',
    placement: 'start',
    variant: 'borderless',
    content: '',
    loading: true,
  })

  const chunks = chunkText(createAssistantReply(question))
  let index = 0

  const pushNextChunk = () => {
    const currentMessage = messages.value[messages.value.length - 1]

    if (!currentMessage || currentMessage.role !== 'assistant') {
      isStreaming.value = false
      return
    }

    if (index >= chunks.length) {
      currentMessage.loading = false
      isStreaming.value = false
      stopStreamingTimer()
      scrollToBottom()
      return
    }

    currentMessage.content = `${currentMessage.content ?? ''}${chunks[index]}`
    currentMessage.loading = false
    index += 1
    scrollToBottom()

    streamTimer = window.setTimeout(pushNextChunk, 80 + Math.random() * 120)
  }

  pushNextChunk()
}

const handleSubmit = () => {
  if (isStreaming.value) return

  const text = senderRef.value?.getModelValue()?.text.trim() ?? ''

  if (!text) return

  messages.value.push({
    id: ++messageId,
    role: 'user',
    placement: 'end',
    variant: 'filled',
    content: text,
  })

  senderRef.value?.clear()
  scrollToBottom()
  streamAssistantReply(text)
}

onBeforeUnmount(() => {
  stopStreamingTimer()
})
</script>

<style scoped lang="scss">
.home-main {
  height: 100%;
  padding: 24px;
  background: #f5f7fa;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.top-bar {
  display: flex;
  align-items: center;
}

.chat-shell {
  width: min(960px, 100%);
  height: 100%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.model-select {
  width: 220px;
}

.chat-list {
  flex: 1;
  min-height: 0;
  padding: 8px 4px;
  border-radius: 16px;
  background: #fafafa;
}

.sender-wrap {
  width: min(760px, 100%);
  margin: 0 auto;
}
</style>
