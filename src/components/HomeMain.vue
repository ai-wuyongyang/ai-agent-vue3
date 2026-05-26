<template>
  <el-main class="home-main">
    <div class="top-bar">
      <el-select v-model="selectedModel" class="model-select" placeholder="请选择模型">
        <el-option v-for="item in modelOptions" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </div>

    <div class="chat-shell">
      <el-scrollbar ref="scrollbarRef" class="chat-list">
        <div class="chat-list-inner">
          <div v-for="item in messages" :key="item.id" :class="['message-row', `message-row--${item.role}`]">
            <div :class="['message-bubble', `message-bubble--${item.role}`]">
              <div :class="['message-markdown', `message-markdown--${item.role}`]">
                <template v-if="item.role === 'assistant'">
                  <MarkdownRender :custom-id="`ai-chat-${item.id}`" :content="item.content" :final="!item.loading"
                    :typewriter="item.loading" smooth-streaming="auto" :code-block-stream="true"
                    :code-block-props="codeBlockProps" />
                  <div v-if="item.loading && !item.content" class="message-streaming">
                    正在生成...
                  </div>
                </template>
                <MarkdownRender v-else :custom-id="`ai-user-${item.id}`" :content="item.content" :final="true" />
              </div>
            </div>
          </div>
        </div>
      </el-scrollbar>

      <div class="sender-wrap">
        <XSender ref="senderRef" :loading="isStreaming" placeholder="输入你的问题，点击回车发起流式请求" @submit="handleSubmit" />
      </div>
    </div>
  </el-main>
</template>

<script setup lang="ts">
import { nextTick, onBeforeUnmount, ref } from 'vue'
import { ElMessage } from 'element-plus'
import type { ElScrollbar } from 'element-plus'
import { XSender } from 'vue-element-plus-x'
import MarkdownRender from 'markstream-vue'
import 'markstream-vue/index.css'

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

type StreamRequestMessage = Pick<ChatMessage, 'role' | 'content'>

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? ''
const senderRef = ref<SenderRef | null>(null)
const scrollbarRef = ref<InstanceType<typeof ElScrollbar> | null>(null)
const isStreaming = ref(false)
const selectedModel = ref('qwen-coder-turbo')
const modelOptions = [
  { label: 'Qwen Coder Turbo', value: 'qwen-coder-turbo' },
  { label: 'Qwen Turbo', value: 'qwen-turbo' },
  { label: 'Qwen Plus', value: 'qwen-plus' },
]
const codeBlockProps = {
  theme: {
    light: 'vitesse-light',
    dark: 'vitesse-dark',
  },
  showHeader: true,
  showCopyButton: true,
  showExpandButton: true,
  showCollapseButton: true,
  showTooltips: true,
  showPreviewButton: false,
  monacoOptions: {
    readOnly: true,
    wordWrap: 'on',
    fontSize: 13,
    lineHeight: 22,
    minimap: {
      enabled: false,
    },
    padding: {
      top: 14,
      bottom: 14,
    },
    scrollBeyondLastLine: false,
    automaticLayout: true,
    fontFamily: "'JetBrains Mono', 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace",
  },
}
const messages = ref<ChatMessage[]>([])

let messageId = messages.value.length
let abortController: AbortController | null = null

const scrollToBottom = () => {
  nextTick(() => {
    const wrapRef = scrollbarRef.value?.wrapRef
    if (!wrapRef) {
      return
    }

    wrapRef.scrollTo({
      top: wrapRef.scrollHeight,
      behavior: 'smooth',
    })
  })
}

const stopStreamRequest = () => {
  if (abortController) {
    abortController.abort()
    abortController = null
  }
}

const createPayloadMessages = (): StreamRequestMessage[] => {
  return messages.value.map(({ role, content }) => ({
    role,
    content,
  }))
}

const appendAssistantPlaceholder = () => {
  isStreaming.value = true

  messages.value.push({
    id: ++messageId,
    role: 'assistant',
    placement: 'start',
    variant: 'borderless',
    content: '',
    loading: true,
  })
}

const updateCurrentAssistantMessage = (updater: (message: ChatMessage) => ChatMessage) => {
  const currentIndex = messages.value.length - 1
  const currentMessage = messages.value[currentIndex]

  if (!currentMessage || currentMessage.role !== 'assistant') {
    throw new Error('当前没有可写入的助手消息')
  }

  messages.value[currentIndex] = updater(currentMessage)
}

const streamAssistantReply = async (payloadMessages: StreamRequestMessage[]) => {
  appendAssistantPlaceholder()
  const decoder = new TextDecoder('utf-8')
  abortController = new AbortController()

  try {
    const response = await fetch(`${API_BASE_URL}/api/chat/stream`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: selectedModel.value,
        messages: payloadMessages,
      }),
      signal: abortController.signal,
    })

    if (!response.ok || !response.body) {
      const message = await response.text()
      throw new Error(message || '流式请求失败')
    }

    const reader = response.body.getReader()

    while (true) {
      const { done, value } = await reader.read()

      if (done) {
        break
      }

      const chunkText = decoder.decode(value, { stream: true })
      if (!chunkText) {
        continue
      }

      updateCurrentAssistantMessage((currentMessage) => ({
        ...currentMessage,
        content: `${currentMessage.content}${chunkText}`,
      }))
      scrollToBottom()
    }

    const finalChunk = decoder.decode()
    if (finalChunk) {
      updateCurrentAssistantMessage((currentMessage) => ({
        ...currentMessage,
        content: `${currentMessage.content}${finalChunk}`,
      }))
    }
  } catch (error) {
    if (error instanceof DOMException && error.name === 'AbortError') {
      return
    }

    const currentMessage = messages.value[messages.value.length - 1]
    if (currentMessage?.role === 'assistant' && !currentMessage.content) {
      updateCurrentAssistantMessage((message) => ({
        ...message,
        content: '请求失败，请稍后重试。',
      }))
    }
    ElMessage.error(error instanceof Error ? error.message : '流式请求失败')
  } finally {
    if (messages.value[messages.value.length - 1]?.role === 'assistant') {
      updateCurrentAssistantMessage((currentMessage) => ({
        ...currentMessage,
        loading: false,
      }))
    }

    isStreaming.value = false
    abortController = null
    scrollToBottom()
  }
}

const handleSubmit = async () => {
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
  await streamAssistantReply(createPayloadMessages())
}

onBeforeUnmount(() => {
  stopStreamRequest()
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
  border-radius: 16px;
  background: #fafafa;
}

.chat-list-inner {
  padding: 8px 4px;
}

.message-row {
  display: flex;
  margin-bottom: 16px;
}

.message-row--assistant {
  justify-content: flex-start;
}

.message-row--user {
  justify-content: flex-end;
}

.message-bubble {
  max-width: min(100%, 760px);
  padding: 14px 16px;
  border-radius: 16px;
}

.message-bubble--assistant {
  background: #ffffff;
  color: #303133;
  box-shadow: 0 6px 18px rgba(15, 23, 42, 0.06);
}

.message-bubble--user {
  background: #409eff;
  color: #ffffff;
}

.sender-wrap {
  width: min(760px, 100%);
  margin: 0 auto;
}

.message-streaming {
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.75;
}

.message-streaming--user {
  color: inherit;
}

.message-markdown {
  line-height: 1.75;
  word-break: break-word;
}

.message-markdown--assistant {
  color: #303133;
}

.message-markdown--user {
  color: inherit;
}

.message-markdown :deep(*:first-child) {
  margin-top: 0;
}

.message-markdown :deep(*:last-child) {
  margin-bottom: 0;
}

.message-markdown :deep(p),
.message-markdown :deep(ul),
.message-markdown :deep(ol),
.message-markdown :deep(pre),
.message-markdown :deep(blockquote),
.message-markdown :deep(table) {
  margin: 0 0 12px;
}

.message-markdown :deep(ul),
.message-markdown :deep(ol) {
  padding-left: 20px;
}

.message-markdown :deep(li + li) {
  margin-top: 4px;
}

.message-markdown :deep(a) {
  color: #409eff;
  text-decoration: none;
}

.message-markdown :deep(a:hover) {
  text-decoration: underline;
}

.message-markdown :deep(code:not(pre code)) {
  padding: 2px 6px;
  border-radius: 6px;
  background: rgba(64, 158, 255, 0.1);
  font-size: 0.92em;
}

.message-markdown :deep(pre) {
  overflow-x: auto;
  border-radius: 12px;
  border: 1px solid #ebeef5;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

.message-markdown :deep(pre code) {
  font-family: 'JetBrains Mono', 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;
  font-size: 13px;
  line-height: 1.7;
}

.message-markdown :deep(blockquote) {
  margin-left: 0;
  padding-left: 12px;
  border-left: 4px solid #dcdfe6;
  color: #606266;
}

.message-markdown :deep(table) {
  width: 100%;
  border-collapse: collapse;
}

.message-markdown :deep(th),
.message-markdown :deep(td) {
  padding: 8px 12px;
  border: 1px solid #ebeef5;
  text-align: left;
}

.message-markdown :deep(th) {
  background: #f5f7fa;
}

.message-bubble--assistant .message-markdown :deep(pre) {
  margin-bottom: 14px;
}

.message-bubble--assistant .message-markdown :deep(table) {
  background: #ffffff;
}
</style>
