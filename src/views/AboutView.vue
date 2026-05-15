<script setup lang="ts">
import { computed, nextTick, onMounted, ref } from 'vue'

type ChatRole = 'user' | 'assistant'
type ChatMessage = {
  id: string
  role: ChatRole
  content: string
  createdAt: number
}

type ChatThread = {
  id: string
  title: string
  messages: ChatMessage[]
  createdAt: number
}

function newId() {
  return typeof crypto !== 'undefined' && 'randomUUID' in crypto
    ? crypto.randomUUID()
    : `${Date.now()}-${Math.random().toString(16).slice(2)}`
}

function createMessage(role: ChatRole, content: string): ChatMessage {
  return { id: newId(), role, content, createdAt: Date.now() }
}

const threads = ref<ChatThread[]>([
  {
    id: newId(),
    title: 'Create Chatbot GPT...',
    createdAt: Date.now(),
    messages: [
      createMessage('user', 'Create a chatbot gpt using python language what will be step for that'),
      createMessage(
        'assistant',
        `Sure, I can help you get started with creating a chatbot using GPT in Python. Here are the basic steps you'll need to follow:\n\n1. Install the required libraries: You'll need to install the transformers library from Hugging Face to use GPT. You can install it using pip.\n\n2. Load the pre-trained model: GPT comes in several sizes and versions, so you'll need to choose one that fits your needs. You can load a pre-trained GPT model. This loads the 1.3B parameter version of GPT-Neo, which is a powerful and relatively recent model.\n\n3. Create a chatbot loop: You'll need to create a loop that takes user input, generates a response using the GPT model, and outputs it to the user.\n\n4. Add some personality to the chatbot: While GPT can generate text, it doesn't have any inherent personality or style. You can make your chatbot more interesting by adding custom prompts or responses that reflect your desired personality.\n\nThese are just the basic steps to get started with a GPT chatbot in Python. Depending on your requirements, you may need to add more features or complexity to the chatbot. Good luck!`,
      ),
      createMessage('user', 'What is use of that chatbot ?'),
      createMessage(
        'assistant',
        `Chatbots can be used for a wide range of purposes, including:\n\nCustomer service chatbots can handle frequently asked questions, provide basic support, and help customers navigate a product or service.\n\nThey can also be used for scheduling, onboarding, lead generation, internal tools, or as a learning assistant — it depends on what you build into the workflow.`,
      ),
    ],
  },
  {
    id: newId(),
    title: 'What is UI/UX design?',
    createdAt: Date.now(),
    messages: [],
  },
  {
    id: newId(),
    title: 'Create POS System',
    createdAt: Date.now(),
    messages: [],
  },
])

const activeThreadId = ref(threads.value[0]?.id ?? newId())
const activeThread = computed(() => threads.value.find((t) => t.id === activeThreadId.value) ?? threads.value[0])
const activeMessages = computed(() => activeThread.value?.messages ?? [])

const input = ref('')
const isThinking = ref(false)

const scrollerEl = ref<HTMLElement | null>(null)
const composerTextareaEl = ref<HTMLTextAreaElement | null>(null)

const hasConversation = computed(() => activeMessages.value.length > 0)

const canSend = computed(() => {
  const hasText = input.value.trim().length > 0
  return hasText && !isThinking.value
})

async function scrollToBottom() {
  await nextTick()
  scrollerEl.value?.scrollTo({ top: scrollerEl.value.scrollHeight, behavior: 'smooth' })
}

function resizeComposer() {
  const el = composerTextareaEl.value
  if (!el) return
  el.style.height = 'auto'
  const next = Math.min(el.scrollHeight, 200)
  el.style.height = `${next}px`
}

function buildAssistantReply(userText: string) {
  const text = userText.trim()
  if (!text) return '我没看到你输入内容。'
  if (text.length <= 12) return `收到：${text}`
  return `我收到了你的消息：\n\n${text}\n\n（这里先用本地假回复占位，后续可以接入真实接口。）`
}

function newChat() {
  const thread: ChatThread = { id: newId(), title: 'New chat', messages: [], createdAt: Date.now() }
  threads.value = [thread, ...threads.value]
  activeThreadId.value = thread.id
  input.value = ''
  isThinking.value = false
  void nextTick(() => {
    resizeComposer()
    void scrollToBottom()
  })
}

function openThread(id: string) {
  activeThreadId.value = id
  input.value = ''
  isThinking.value = false
  void nextTick(() => {
    resizeComposer()
    void scrollToBottom()
  })
}

function clearAll() {
  const thread: ChatThread = { id: newId(), title: 'New chat', messages: [], createdAt: Date.now() }
  threads.value = [thread]
  activeThreadId.value = thread.id
  input.value = ''
  isThinking.value = false
  void nextTick(() => {
    resizeComposer()
    void scrollToBottom()
  })
}

async function send() {
  if (!canSend.value) return

  const content = input.value.trim()
  input.value = ''
  resizeComposer()

  const thread = activeThread.value
  if (!thread) return
  thread.messages.push(createMessage('user', content))
  if (thread.title === 'New chat') thread.title = content.length > 24 ? `${content.slice(0, 24)}…` : content

  isThinking.value = true
  await scrollToBottom()

  const reply = buildAssistantReply(content)
  await new Promise((r) => setTimeout(r, 500))

  thread.messages.push(createMessage('assistant', reply))

  isThinking.value = false
  await scrollToBottom()
}

function onComposerKeydown(e: KeyboardEvent) {
  if (e.key !== 'Enter') return
  if (e.shiftKey) return
  e.preventDefault()
  void send()
}

onMounted(() => {
  resizeComposer()
  void scrollToBottom()
})
</script>

<template>
  <div class="stage">
    <div class="frame">
      <div class="app">
        <aside class="side">
          <div class="sideTop">
            <div class="logo">CHAT AI+</div>
            <div class="sideActions">
              <button class="primaryBtn" type="button" @click="newChat">
                <span class="btnPlus" aria-hidden="true" />
                <span>New chat</span>
              </button>
              <button class="iconBtn" type="button" aria-label="Search">
                <span class="iSearch" aria-hidden="true" />
              </button>
            </div>
          </div>

          <div class="sideMeta">
            <div class="metaTitle">Your conversations</div>
            <button class="linkBtn" type="button" @click="clearAll">Clear All</button>
          </div>

          <div class="chatList">
            <button
              v-for="t in threads"
              :key="t.id"
              class="chatItem"
              type="button"
              :data-active="t.id === activeThreadId"
              @click="openThread(t.id)"
            >
              <span class="chatBullet" aria-hidden="true" />
              <span class="chatTitle">{{ t.title }}</span>
              <span class="chatTools" aria-hidden="true">
                <span class="miniIcon" data-i="edit" />
                <span class="miniIcon" data-i="more" />
              </span>
            </button>
          </div>

          <div class="sideBottom">
            <button class="settings" type="button">
              <span class="miniIcon" data-i="gear" aria-hidden="true" />
              <span>Settings</span>
            </button>
            <div class="userCard">
              <div class="userAvatar">A</div>
              <div class="userInfo">
                <div class="userName">Andrew Neilson</div>
                <div class="userPlan">Free</div>
              </div>
            </div>
          </div>
        </aside>

        <section class="main">
          <div class="upgrade">Upgrade to Pro</div>

          <div ref="scrollerEl" class="thread" :data-empty="!hasConversation">
            <div v-for="m in activeMessages" :key="m.id" class="turn" :data-role="m.role">
              <div class="turnHead">
                <div class="avatar" :data-role="m.role">{{ m.role === 'user' ? 'A' : 'C' }}</div>
                <div v-if="m.role === 'assistant'" class="tag">CHAT AI+</div>
              </div>

              <div class="turnBody" :data-role="m.role">
                <div class="content">{{ m.content }}</div>
              </div>

              <div v-if="m.role === 'assistant'" class="turnActions">
                <button class="actionBtn" type="button" aria-label="Like"><span class="miniIcon" data-i="like" /></button>
                <button class="actionBtn" type="button" aria-label="Dislike"><span class="miniIcon" data-i="dislike" /></button>
                <button class="actionBtn" type="button" aria-label="Copy"><span class="miniIcon" data-i="copy" /></button>
                <button class="regen" type="button">Regenerate</button>
              </div>
            </div>

            <div v-if="isThinking" class="turn" data-role="assistant">
              <div class="turnHead">
                <div class="avatar" data-role="assistant">C</div>
                <div class="tag">CHAT AI+</div>
              </div>
              <div class="turnBody" data-role="assistant">
                <div class="thinkingDots" aria-label="Thinking">
                  <span class="dot" />
                  <span class="dot" />
                  <span class="dot" />
                </div>
              </div>
            </div>
          </div>

          <div class="composerDock">
            <div class="composer">
              <textarea
                ref="composerTextareaEl"
                v-model="input"
                class="composerInput"
                rows="1"
                placeholder="What's in your mind?..."
                :disabled="isThinking"
                @input="resizeComposer"
                @keydown="onComposerKeydown"
              />
              <button class="sendBtn" type="button" :disabled="!canSend" @click="send" aria-label="Send">
                <span class="sendPlane" aria-hidden="true" />
              </button>
            </div>
          </div>
        </section>
      </div>
    </div>
  </div>
</template>

<style scoped>
.stage {
  min-height: 100vh;
  background:
    radial-gradient(1200px 520px at 50% 0%, rgba(123, 87, 255, 0.22), transparent 55%),
    radial-gradient(900px 540px at 50% 100%, rgba(0, 173, 255, 0.2), transparent 55%),
    #0b0e14;
  padding: 28px;
  display: grid;
  place-items: center;
}

.frame {
  width: min(1024px, calc(100vw - 56px));
  aspect-ratio: 2 / 1;
  border-radius: 26px;
  overflow: hidden;
  background: #ffffff;
  box-shadow: 0 70px 110px rgba(0, 0, 0, 0.5);
}

.app {
  height: 100%;
  display: grid;
  grid-template-columns: 270px 1fr;
  background: #f3f6fd;
  color: #0f172a;
}

.side {
  padding: 18px 14px;
}

.sideTop {
  display: grid;
  gap: 14px;
  padding: 2px 6px 10px;
}

.logo {
  letter-spacing: 2px;
  font-weight: 600;
  font-size: 13px;
}

.sideActions {
  display: grid;
  grid-template-columns: 1fr 40px;
  gap: 10px;
  align-items: center;
}

.primaryBtn {
  height: 38px;
  border-radius: 14px;
  border: none;
  background: #5969ff;
  color: #ffffff;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 0 14px;
  font-weight: 600;
}

.primaryBtn:hover {
  filter: brightness(0.98);
}

.btnPlus {
  width: 14px;
  height: 14px;
  position: relative;
  display: inline-block;
}

.btnPlus::before,
.btnPlus::after {
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  width: 12px;
  height: 2px;
  background: currentColor;
  transform: translate(-50%, -50%);
  border-radius: 999px;
}

.btnPlus::after {
  width: 2px;
  height: 12px;
}

.iconBtn {
  width: 40px;
  height: 40px;
  border-radius: 14px;
  border: 1px solid rgba(15, 23, 42, 0.1);
  background: #ffffff;
  cursor: pointer;
  display: grid;
  place-items: center;
}

.iSearch {
  width: 15px;
  height: 15px;
  border: 2px solid rgba(15, 23, 42, 0.45);
  border-radius: 999px;
  position: relative;
  display: inline-block;
}

.iSearch::after {
  content: '';
  position: absolute;
  width: 8px;
  height: 2px;
  background: rgba(15, 23, 42, 0.45);
  right: -6px;
  bottom: -4px;
  transform: rotate(45deg);
  border-radius: 999px;
}

.sideMeta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 6px 8px;
}

.metaTitle {
  font-size: 11px;
  color: rgba(15, 23, 42, 0.55);
}

.linkBtn {
  border: none;
  background: transparent;
  cursor: pointer;
  color: #5969ff;
  font-size: 11px;
  font-weight: 600;
}

.chatList {
  background: #ffffff;
  border-radius: 18px;
  padding: 10px 8px;
  box-shadow: 0 10px 22px rgba(17, 24, 39, 0.06);
  display: grid;
  gap: 2px;
  height: calc(100% - 190px);
  overflow: auto;
}

.chatItem {
  width: 100%;
  height: 34px;
  border-radius: 12px;
  border: none;
  background: transparent;
  cursor: pointer;
  display: grid;
  grid-template-columns: 18px 1fr auto;
  align-items: center;
  gap: 8px;
  padding: 0 10px;
  text-align: left;
  color: rgba(15, 23, 42, 0.82);
}

.chatItem[data-active='true'] {
  background: rgba(89, 105, 255, 0.12);
}

.chatBullet {
  width: 10px;
  height: 10px;
  border-radius: 999px;
  border: 2px solid rgba(15, 23, 42, 0.2);
}

.chatTitle {
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 12px;
}

.chatTools {
  display: inline-flex;
  gap: 8px;
  opacity: 0.7;
}

.sideBottom {
  padding: 12px 6px 6px;
  display: grid;
  gap: 10px;
}

.settings {
  height: 36px;
  border-radius: 14px;
  border: 1px solid rgba(15, 23, 42, 0.1);
  background: rgba(255, 255, 255, 0.65);
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 0 12px;
}

.userCard {
  border-radius: 18px;
  background: #ffffff;
  border: 1px solid rgba(15, 23, 42, 0.08);
  padding: 10px 12px;
  display: grid;
  grid-template-columns: 34px 1fr;
  gap: 10px;
  align-items: center;
}

.userAvatar {
  width: 34px;
  height: 34px;
  border-radius: 999px;
  display: grid;
  place-items: center;
  background: rgba(89, 105, 255, 0.14);
  font-weight: 700;
}

.userInfo {
  min-width: 0;
}

.userName {
  font-size: 12px;
  font-weight: 700;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.userPlan {
  font-size: 11px;
  color: rgba(15, 23, 42, 0.55);
}

.main {
  min-width: 0;
  position: relative;
  padding: 18px 18px 16px 0;
}

.upgrade {
  position: absolute;
  right: 6px;
  top: 50%;
  transform: translateY(-50%) rotate(-90deg);
  transform-origin: right center;
  background: linear-gradient(180deg, #5969ff, #6a44ff);
  color: #ffffff;
  padding: 10px 14px;
  border-radius: 999px;
  font-size: 11px;
  font-weight: 700;
  box-shadow: 0 16px 30px rgba(89, 105, 255, 0.35);
}

.thread {
  height: 100%;
  padding: 10px 36px 88px 10px;
  overflow: auto;
  display: grid;
  gap: 18px;
}

.turn {
  width: min(680px, 100%);
  justify-self: center;
}

.turnHead {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 2px 8px;
}

.avatar {
  width: 28px;
  height: 28px;
  border-radius: 999px;
  display: grid;
  place-items: center;
  font-weight: 700;
  font-size: 12px;
  background: #ffffff;
  border: 1px solid rgba(15, 23, 42, 0.1);
}

.avatar[data-role='assistant'] {
  background: rgba(89, 105, 255, 0.14);
  border-color: rgba(89, 105, 255, 0.25);
}

.tag {
  font-size: 11px;
  color: #5969ff;
  font-weight: 700;
}

.turnBody {
  border-radius: 18px;
  padding: 14px 16px;
  background: #ffffff;
  box-shadow: 0 16px 26px rgba(15, 23, 42, 0.06);
  border: 1px solid rgba(15, 23, 42, 0.06);
}

.turnBody[data-role='user'] {
  background: transparent;
  box-shadow: none;
  border: none;
  padding: 0;
}

.content {
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.65;
  font-size: 12px;
  color: rgba(15, 23, 42, 0.82);
}

.turnActions {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 6px 0;
}

.actionBtn {
  width: 28px;
  height: 28px;
  border-radius: 10px;
  border: 1px solid rgba(15, 23, 42, 0.08);
  background: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  display: grid;
  place-items: center;
}

.regen {
  margin-left: auto;
  height: 28px;
  border-radius: 999px;
  border: 1px solid rgba(15, 23, 42, 0.08);
  background: rgba(255, 255, 255, 0.9);
  cursor: pointer;
  padding: 0 12px;
  font-size: 11px;
  font-weight: 600;
  color: rgba(15, 23, 42, 0.65);
}

.composerDock {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 12px;
  display: grid;
  place-items: center;
  padding-right: 42px;
}

.composer {
  width: min(520px, calc(100% - 36px));
  border-radius: 999px;
  background: #ffffff;
  box-shadow: 0 18px 40px rgba(15, 23, 42, 0.12);
  border: 1px solid rgba(15, 23, 42, 0.08);
  display: grid;
  grid-template-columns: 1fr 42px;
  gap: 10px;
  align-items: center;
  padding: 8px 8px 8px 16px;
}

.composerInput {
  width: 100%;
  resize: none;
  border: none;
  outline: none;
  background: transparent;
  color: rgba(15, 23, 42, 0.85);
  font-size: 12px;
  line-height: 1.6;
  max-height: 200px;
  padding: 8px 0;
}

.sendBtn {
  width: 38px;
  height: 38px;
  border-radius: 999px;
  border: none;
  background: linear-gradient(180deg, #5969ff, #6a44ff);
  cursor: pointer;
  display: grid;
  place-items: center;
  color: #ffffff;
  box-shadow: 0 12px 26px rgba(89, 105, 255, 0.35);
}

.sendBtn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.sendPlane {
  width: 0;
  height: 0;
  border-left: 10px solid currentColor;
  border-top: 7px solid transparent;
  border-bottom: 7px solid transparent;
  transform: translateX(2px);
}

.thinkingDots {
  display: inline-flex;
  gap: 6px;
  align-items: center;
  height: 22px;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 999px;
  background: rgba(15, 23, 42, 0.35);
  animation: blink 1.05s infinite ease-in-out;
}

.dot:nth-child(2) {
  animation-delay: 0.15s;
}

.dot:nth-child(3) {
  animation-delay: 0.3s;
}

@keyframes blink {
  0%,
  100% {
    opacity: 0.25;
    transform: translateY(0);
  }
  50% {
    opacity: 1;
    transform: translateY(-1px);
  }
}

.miniIcon {
  width: 14px;
  height: 14px;
  display: inline-block;
  position: relative;
}

.miniIcon[data-i='edit'] {
  border: 1.6px solid rgba(15, 23, 42, 0.4);
  border-radius: 3px;
}

.miniIcon[data-i='edit']::after {
  content: '';
  position: absolute;
  width: 9px;
  height: 2px;
  background: rgba(15, 23, 42, 0.4);
  left: 2px;
  top: 6px;
  transform: rotate(-25deg);
  border-radius: 999px;
}

.miniIcon[data-i='more'] {
  background: radial-gradient(circle, rgba(15, 23, 42, 0.45) 2px, transparent 3px) 50% 30% / 100% 50% no-repeat;
}

.miniIcon[data-i='more']::before,
.miniIcon[data-i='more']::after {
  content: '';
  position: absolute;
  left: 50%;
  width: 4px;
  height: 4px;
  border-radius: 999px;
  background: rgba(15, 23, 42, 0.45);
  transform: translateX(-50%);
}

.miniIcon[data-i='more']::before {
  top: 5px;
}

.miniIcon[data-i='more']::after {
  bottom: 1px;
}

.miniIcon[data-i='gear'] {
  border: 1.6px solid rgba(15, 23, 42, 0.45);
  border-radius: 999px;
}

.miniIcon[data-i='gear']::after {
  content: '';
  position: absolute;
  inset: 3px;
  border: 1.6px solid rgba(15, 23, 42, 0.25);
  border-radius: 999px;
}

.miniIcon[data-i='like'],
.miniIcon[data-i='dislike'] {
  border: 1.6px solid rgba(15, 23, 42, 0.4);
  border-radius: 4px;
}

.miniIcon[data-i='copy'] {
  border: 1.6px solid rgba(15, 23, 42, 0.4);
  border-radius: 3px;
}

.miniIcon[data-i='copy']::after {
  content: '';
  position: absolute;
  inset: 3px 1px 1px 3px;
  border: 1.6px solid rgba(15, 23, 42, 0.2);
  border-radius: 3px;
}

@media (max-width: 980px) {
  .stage {
    padding: 12px;
  }

  .frame {
    aspect-ratio: auto;
    height: auto;
  }

  .app {
    grid-template-columns: 1fr;
  }

  .side {
    display: none;
  }

  .main {
    padding: 18px;
  }

  .composerDock {
    padding-right: 0;
  }

  .upgrade {
    display: none;
  }
}
</style>
