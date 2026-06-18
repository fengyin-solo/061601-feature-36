<script setup lang="ts">
import { ref, computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import gameConfig from '../config/gameConfig'
import type { StartPreset, StartPresetModifiers } from '../types/game'

const props = withDefaults(defineProps<{
  forceMode?: boolean
}>(), {
  forceMode: false
})

const emit = defineEmits<{
  (e: 'close'): void
}>()

const gameStore = useGameStore()
const selectedPresetId = ref<string>('normal')

const presets = computed(() => gameConfig.startPresets)
const selectedPreset = computed<StartPreset>(() =>
  presets.value.find(p => p.id === selectedPresetId.value) || presets.value[0]
)

interface ModifierDisplay {
  label: string
  value: string
  positive: boolean
}

const modifierDisplays = computed<ModifierDisplay[]>(() => {
  const mod = selectedPreset.value.modifiers
  const items: ModifierDisplay[] = []

  if (mod.initialResources !== undefined) {
    const diff = mod.initialResources - gameConfig.initialResources
    items.push({
      label: '初始代币',
      value: diff > 0 ? `+${diff}` : `${diff}`,
      positive: diff >= 0
    })
  }
  if (mod.maxActionsPerDay !== undefined) {
    const diff = mod.maxActionsPerDay - gameConfig.maxActionsPerDay
    items.push({
      label: '每日行动力',
      value: diff > 0 ? `+${diff}` : `${diff}`,
      positive: diff >= 0
    })
  }
  if (mod.baseAffinityBonus !== undefined) {
    items.push({
      label: '初始好感加成',
      value: `+${mod.baseAffinityBonus}`,
      positive: true
    })
  }
  if (mod.baseMoodBonus !== undefined) {
    items.push({
      label: '初始心情加成',
      value: `+${mod.baseMoodBonus}`,
      positive: true
    })
  }
  if (mod.affinityGainMultiplier !== undefined) {
    const pct = Math.round((mod.affinityGainMultiplier - 1) * 100)
    items.push({
      label: '好感获取',
      value: pct > 0 ? `+${pct}%` : `${pct}%`,
      positive: pct >= 0
    })
  }
  if (mod.resourceGainMultiplier !== undefined) {
    const pct = Math.round((mod.resourceGainMultiplier - 1) * 100)
    items.push({
      label: '资源获取',
      value: pct > 0 ? `+${pct}%` : `${pct}%`,
      positive: pct >= 0
    })
  }
  if (mod.moodDecayMultiplier !== undefined) {
    const pct = Math.round((mod.moodDecayMultiplier - 1) * 100)
    items.push({
      label: '每日心情衰减',
      value: pct > 0 ? `+${pct}%` : `${pct}%`,
      positive: pct < 0
    })
  }
  if (mod.affinityDecayMultiplier !== undefined) {
    const pct = Math.round((mod.affinityDecayMultiplier - 1) * 100)
    items.push({
      label: '每日好感衰减',
      value: pct > 0 ? `+${pct}%` : `${pct}%`,
      positive: pct < 0
    })
  }
  if (mod.workRewardsMultiplier !== undefined) {
    const pct = Math.round((mod.workRewardsMultiplier - 1) * 100)
    items.push({
      label: '打工收益',
      value: pct > 0 ? `+${pct}%` : `${pct}%`,
      positive: pct >= 0
    })
  }
  if (mod.unlockAllCharacters) {
    items.push({
      label: '角色解锁',
      value: '全部解锁',
      positive: true
    })
  }
  if (mod.startWithCards) {
    items.push({
      label: '卡牌继承',
      value: '保留历史卡牌',
      positive: true
    })
  }

  return items
})

function selectPreset(id: string) {
  selectedPresetId.value = id
}

function confirmStart() {
  gameStore.resetGame(selectedPresetId.value)
  if (!props.forceMode) {
    emit('close')
  }
}

function handleOverlayClick(e: MouseEvent) {
  if (props.forceMode) return
  if ((e.target as HTMLElement).classList.contains('modal-overlay')) {
    emit('close')
  }
}

function handleClose() {
  if (!props.forceMode) {
    emit('close')
  }
}
</script>

<template>
  <div class="modal-overlay" @click="handleOverlayClick">
    <div class="modal new-game-modal">
      <div class="modal-header">
        <h2>🎮 选择你的开局</h2>
        <button
          v-if="!forceMode"
          class="close-btn"
          @click="handleClose"
        >✕</button>
      </div>

      <div class="modal-body">
        <div class="preset-list">
          <div
            v-for="preset in presets"
            :key="preset.id"
            class="preset-card"
            :class="{ selected: selectedPresetId === preset.id }"
            @click="selectPreset(preset.id)"
          >
            <div class="preset-icon">{{ preset.icon }}</div>
            <div class="preset-name">{{ preset.name }}</div>
            <div class="preset-tagline">{{ preset.tagline }}</div>
          </div>
        </div>

        <div class="preset-detail card">
          <div class="detail-header">
            <span class="detail-icon">{{ selectedPreset.icon }}</span>
            <div>
              <h3>{{ selectedPreset.name }}</h3>
              <p class="tagline">{{ selectedPreset.tagline }}</p>
            </div>
          </div>

          <p class="preset-description">{{ selectedPreset.description }}</p>

          <div v-if="modifierDisplays.length > 0" class="modifiers-section">
            <h4>属性调整</h4>
            <div class="modifier-grid">
              <div
                v-for="(mod, idx) in modifierDisplays"
                :key="idx"
                class="modifier-item"
                :class="{ positive: mod.positive, negative: !mod.positive }"
              >
                <span class="mod-label">{{ mod.label }}</span>
                <span class="mod-value">{{ mod.value }}</span>
              </div>
            </div>
          </div>
          <div v-else class="modifiers-section neutral">
            <h4>属性调整</h4>
            <p class="no-modifiers">标准数值，无特殊加成</p>
          </div>
        </div>
      </div>

      <div class="modal-footer">
        <div v-if="forceMode" class="force-mode-hint">
          ⚠️ 请选择一个开局以开始游戏
        </div>
        <button
          v-if="!forceMode"
          class="btn btn-secondary"
          @click="handleClose"
        >取消</button>
        <button class="btn btn-primary" @click="confirmStart">
          🚀 以「{{ selectedPreset.name }}」开始
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
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
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px;
  border-bottom: 1px solid var(--border-color);
}

.modal-header h2 {
  font-size: 22px;
  font-weight: 700;
  margin: 0;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.close-btn {
  width: 36px;
  height: 36px;
  border-radius: var(--radius-md);
  background: var(--bg-tertiary);
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
}

.close-btn:hover {
  background: var(--accent-light);
  transform: scale(1.05);
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.preset-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 12px;
}

.preset-card {
  background: var(--bg-secondary);
  border: 2px solid var(--border-color);
  border-radius: var(--radius-md);
  padding: 16px 12px;
  text-align: center;
  cursor: pointer;
  transition: all 0.25s;
}

.preset-card:hover {
  transform: translateY(-2px);
  border-color: var(--accent-primary);
  background: var(--bg-tertiary);
}

.preset-card.selected {
  border-color: var(--accent-primary);
  background: var(--accent-light);
  box-shadow: 0 0 0 3px rgba(var(--accent-primary-rgb, 236, 72, 153), 0.2);
}

.preset-icon {
  font-size: 36px;
  margin-bottom: 8px;
}

.preset-name {
  font-size: 15px;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 4px;
}

.preset-tagline {
  font-size: 11px;
  color: var(--text-secondary);
  line-height: 1.4;
}

.preset-detail.card {
  background: var(--bg-secondary);
  border-radius: var(--radius-md);
  padding: 20px;
}

.detail-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 12px;
}

.detail-icon {
  font-size: 48px;
  width: 72px;
  height: 72px;
  background: var(--accent-light);
  border-radius: var(--radius-md);
  display: flex;
  align-items: center;
  justify-content: center;
}

.detail-header h3 {
  font-size: 20px;
  font-weight: 700;
  color: var(--text-primary);
  margin: 0 0 4px 0;
}

.tagline {
  font-size: 13px;
  color: var(--text-secondary);
  margin: 0;
  font-style: italic;
}

.preset-description {
  color: var(--text-primary);
  line-height: 1.7;
  margin: 12px 0 20px 0;
  padding: 12px 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-sm);
  border-left: 3px solid var(--accent-primary);
}

.modifiers-section h4 {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin: 0 0 12px 0;
}

.modifier-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 8px;
}

.modifier-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 12px;
  border-radius: var(--radius-sm);
  background: var(--bg-tertiary);
}

.modifier-item.positive {
  background: rgba(34, 197, 94, 0.12);
  border: 1px solid rgba(34, 197, 94, 0.3);
}

.modifier-item.negative {
  background: rgba(239, 68, 68, 0.12);
  border: 1px solid rgba(239, 68, 68, 0.3);
}

.mod-label {
  font-size: 13px;
  color: var(--text-secondary);
}

.mod-value {
  font-size: 13px;
  font-weight: 700;
}

.positive .mod-value {
  color: #22c55e;
}

.negative .mod-value {
  color: #ef4444;
}

.modifiers-section.neutral {
  text-align: center;
  padding: 20px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-sm);
}

.no-modifiers {
  color: var(--text-secondary);
  font-size: 14px;
  margin: 0;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 20px 24px;
  border-top: 1px solid var(--border-color);
}

.btn {
  padding: 10px 20px;
  border-radius: var(--radius-md);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
}

.btn-secondary {
  background: var(--bg-tertiary);
  color: var(--text-primary);
}

.btn-secondary:hover {
  background: var(--border-color);
}

.btn-primary {
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  color: white;
}

.btn-primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(var(--accent-primary-rgb, 236, 72, 153), 0.4);
}

.force-mode-hint {
  flex: 1;
  color: #f59e0b;
  font-size: 13px;
  font-weight: 500;
  display: flex;
  align-items: center;
}

@media (max-width: 640px) {
  .preset-list {
    grid-template-columns: repeat(2, 1fr);
  }

  .modal-body {
    padding: 16px;
  }

  .modal-header {
    padding: 16px 20px;
  }

  .modal-footer {
    padding: 16px 20px;
    flex-direction: column-reverse;
  }

  .force-mode-hint {
    justify-content: center;
    margin-bottom: 8px;
  }

  .btn {
    width: 100%;
  }
}
</style>
