<script setup lang="ts">
import { ref, onMounted, watch, computed, nextTick } from 'vue'
import { useGameStore } from './stores/gameStore'
import { useSaveStore } from './stores/saveStore'
import TopBar from './components/TopBar.vue'
import CharacterPanel from './components/CharacterPanel.vue'
import ActionPanel from './components/ActionPanel.vue'
import LogPanel from './components/LogPanel.vue'
import EventModal from './components/EventModal.vue'
import SaveModal from './components/SaveModal.vue'
import CardCollection from './components/CardCollection.vue'
import HistoryPanel from './components/HistoryPanel.vue'
import GiftModal from './components/GiftModal.vue'
import NewGameModal from './components/NewGameModal.vue'

const gameStore = useGameStore()
const saveStore = useSaveStore()

const showSaveModal = ref(false)
const showCards = ref(false)
const showHistory = ref(false)
const showGiftModal = ref(false)
const showNewGameModal = ref(false)
const showLoadChoiceModal = ref(false)
const autoSaveInfo = ref<{ day: number; time: string } | null>(null)

const theme = computed(() => gameStore.darkMode ? 'dark' : 'light')
const isNewGameModalForced = computed(() => !gameStore.gameInitialized)

watch(() => gameStore.day, () => {
  if (gameStore.gameInitialized) {
    saveStore.autoSave()
  }
})

watch(theme, (newTheme) => {
  document.documentElement.setAttribute('data-theme', newTheme)
})

watch(
  () => gameStore.gameInitialized,
  (initialized) => {
    if (!initialized) {
      nextTick(() => {
        showNewGameModal.value = true
      })
    } else {
      showNewGameModal.value = false
      showLoadChoiceModal.value = false
    }
  },
  { immediate: true }
)

function getAutoSaveInfo() {
  try {
    const data = localStorage.getItem('love_story_game_autosave')
    if (!data) return null
    const save = JSON.parse(data)
    return { day: save.day, time: save.time }
  } catch {
    return null
  }
}

onMounted(() => {
  document.documentElement.setAttribute('data-theme', theme.value)

  const hasSave = saveStore.hasAutoSave()
  if (hasSave) {
    autoSaveInfo.value = getAutoSaveInfo()
    showLoadChoiceModal.value = true
  } else {
    showNewGameModal.value = true
  }
})

function handleLoadAutoSave() {
  showLoadChoiceModal.value = false
  saveStore.loadAutoSave()
}

function handleStartNewFromChoice() {
  showLoadChoiceModal.value = false
  showNewGameModal.value = true
}

function handleReset() {
  if (confirm('确定要开始新游戏吗？当前进度将丢失（卡牌会被保留用于继承记忆模式）')) {
    gameStore.gameInitialized = false
    showNewGameModal.value = true
  }
}
</script>

<template>
  <div class="app-root">
    <div v-if="gameStore.gameInitialized" class="game-container">
      <TopBar 
        @toggle-save="showSaveModal = true"
        @toggle-cards="showCards = true"
        @toggle-history="showHistory = true"
        @toggle-theme="gameStore.toggleDarkMode()"
        @reset="handleReset"
      />
      
      <div class="main-content">
        <div class="left-column">
          <CharacterPanel />
          <ActionPanel @open-gift="showGiftModal = true" />
        </div>
        
        <div class="right-column">
          <LogPanel />
        </div>
      </div>

      <EventModal />
      <SaveModal v-if="showSaveModal" @close="showSaveModal = false" />
      <CardCollection v-if="showCards" @close="showCards = false" />
      <HistoryPanel v-if="showHistory" @close="showHistory = false" />
      <GiftModal v-if="showGiftModal" @close="showGiftModal = false" />
    </div>

    <div v-else class="pre-game-placeholder">
      <div class="placeholder-logo">💝</div>
      <div class="placeholder-title">恋爱物语</div>
      <div class="placeholder-subtitle">请完成开局设定以开始游戏</div>
    </div>

    <div
      v-if="showLoadChoiceModal"
      class="modal-overlay"
    >
      <div class="modal load-choice-modal">
        <div class="modal-header">
          <h2>💾 检测到存档</h2>
        </div>
        <div class="modal-body">
          <div class="save-info-card">
            <div class="save-info-icon">📂</div>
            <div class="save-info-content">
              <div class="save-info-title">自动存档</div>
              <div class="save-info-detail" v-if="autoSaveInfo">
                进度：第 {{ autoSaveInfo.day }} 天 · {{ autoSaveInfo.time }}
              </div>
              <div class="save-info-detail" v-else>
                进度：已保存
              </div>
            </div>
          </div>
          <p class="choice-hint">你可以继续上次的冒险，或者开启全新的周目体验～</p>
        </div>
        <div class="modal-footer choice-footer">
          <button class="btn btn-secondary" @click="handleStartNewFromChoice">
            🌟 开启新周目
          </button>
          <button class="btn btn-primary" @click="handleLoadAutoSave">
            ▶️ 继续游戏
          </button>
        </div>
      </div>
    </div>

    <NewGameModal
      v-if="showNewGameModal"
      :force-mode="isNewGameModalForced"
      @close="showNewGameModal = false"
    />
  </div>
</template>

<style scoped>
.app-root {
  min-height: 100vh;
}

.game-container {
  min-height: 100vh;
  padding: 16px;
  max-width: 1400px;
  margin: 0 auto;
}

.pre-game-placeholder {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, var(--bg-primary), var(--bg-secondary));
  gap: 16px;
}

.placeholder-logo {
  font-size: 80px;
  animation: pulse 2s ease-in-out infinite;
}

.placeholder-title {
  font-size: 36px;
  font-weight: 800;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.placeholder-subtitle {
  font-size: 14px;
  color: var(--text-secondary);
  margin-top: 8px;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.08); opacity: 0.9; }
}

.main-content {
  display: grid;
  grid-template-columns: 1fr 400px;
  gap: 16px;
  margin-top: 16px;
}

.left-column {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.right-column {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 16px;
}

.modal {
  background: var(--bg-primary);
  border-radius: var(--radius-lg);
  width: 100%;
  max-width: 480px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.load-choice-modal {
  animation: slideUp 0.35s cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-header {
  padding: 24px 28px 16px;
  border-bottom: 1px solid var(--border-color);
}

.modal-header h2 {
  margin: 0;
  font-size: 20px;
  font-weight: 700;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.modal-body {
  padding: 24px 28px;
}

.save-info-card {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px;
  background: var(--bg-secondary);
  border-radius: var(--radius-md);
  border: 1px solid var(--border-color);
}

.save-info-icon {
  font-size: 40px;
  width: 60px;
  height: 60px;
  background: var(--accent-light);
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.save-info-title {
  font-size: 16px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 4px;
}

.save-info-detail {
  font-size: 13px;
  color: var(--text-secondary);
}

.choice-hint {
  margin-top: 16px;
  margin-bottom: 0;
  font-size: 13px;
  color: var(--text-secondary);
  text-align: center;
  line-height: 1.6;
}

.choice-footer {
  display: flex;
  gap: 12px;
  padding: 20px 28px 28px;
}

.btn {
  flex: 1;
  padding: 12px 16px;
  border-radius: var(--radius-md);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}

.btn-secondary {
  background: var(--bg-tertiary);
  color: var(--text-primary);
}

.btn-secondary:hover {
  background: var(--border-color);
  transform: translateY(-1px);
}

.btn-primary {
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  color: white;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 14px rgba(var(--accent-primary-rgb, 236, 72, 153), 0.4);
}

@media (max-width: 900px) {
  .main-content {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .game-container {
    padding: 12px;
  }

  .placeholder-logo {
    font-size: 64px;
  }

  .placeholder-title {
    font-size: 28px;
  }

  .choice-footer {
    flex-direction: column-reverse;
    padding: 16px 20px 24px;
  }

  .modal-header {
    padding: 20px 24px 12px;
  }

  .modal-body {
    padding: 20px 24px;
  }
}
</style>
