<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

interface Props {
  title?: string
  visible?: boolean
  width?: number
  isPicture?: boolean
  initialX?: number
  initialY?: number
}

const props = withDefaults(defineProps<Props>(), {
  title: 'Window',
  visible: false,
  width: 600,
  isPicture: false,
  initialX: undefined,
  initialY: undefined,
})

const emit = defineEmits<{
  close: []
  minimize: []
  maximize: []
}>()

const modalOverlayRef = ref<HTMLElement | null>(null)
const modalWindowRef = ref<HTMLElement | null>(null)
const isDragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })
const isMaximized = ref(false)
const isShowContent = ref(true)

// 計算初始位置
const calculateInitialPosition = () => {
  if (props.initialX !== undefined && props.initialY !== undefined) {
    return { x: props.initialX, y: props.initialY }
  }
  // 如果指定了 initialX 或 initialY，使用計算值
  if (props.initialX !== undefined || props.initialY !== undefined) {
    const x = props.initialX !== undefined ? props.initialX : 100
    const y = props.initialY !== undefined ? props.initialY : 100
    return { x, y }
  }
  return { x: 100, y: 100 }
}

const position = ref(calculateInitialPosition())

// 當 visible 變為 true 時，重置位置
watch(() => props.visible, (newVal) => {
  if (newVal) {
    const initialPos = calculateInitialPosition()
    position.value = initialPos
  }
})

// 監聽 initialX 和 initialY 的變化
watch([() => props.initialX, () => props.initialY], () => {
  if (props.visible) {
    const initialPos = calculateInitialPosition()
    position.value = initialPos
  }
})

const handleTitleMouseDown = (event: MouseEvent) => {
  if (isMaximized.value) return
  isDragging.value = true
  if (modalWindowRef.value) {
    const rect = modalWindowRef.value.getBoundingClientRect()
    dragOffset.value.x = event.clientX - rect.left
    dragOffset.value.y = event.clientY - rect.top
  }
  event.preventDefault()
  event.stopPropagation()
}

const handleMouseMove = (event: MouseEvent) => {
  if (isDragging.value && !isMaximized.value && modalOverlayRef.value && modalWindowRef.value) {
    const overlayRect = modalOverlayRef.value.getBoundingClientRect()
    const windowRect = modalWindowRef.value.getBoundingClientRect()
    position.value.x = event.clientX - overlayRect.left - dragOffset.value.x
    position.value.y = event.clientY - overlayRect.top - dragOffset.value.y
    
    // 限制在視窗範圍內
    const maxX = overlayRect.width - windowRect.width
    const maxY = overlayRect.height - windowRect.height
    position.value.x = Math.max(0, Math.min(position.value.x, maxX))
    position.value.y = Math.max(0, Math.min(position.value.y, maxY))
  }
}

const handleMouseUp = () => {
  isDragging.value = false
}

const handleClose = () => {
  emit('close')
}

onMounted(() => {
  window.addEventListener('mousemove', handleMouseMove)
  window.addEventListener('mouseup', handleMouseUp)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('mouseup', handleMouseUp)
})
</script>

<template>
  <Teleport to="body">
    <div
      v-if="visible"
      ref="modalOverlayRef"
      class="modal-overlay"
    >
      <div
        ref="modalWindowRef"
        class="modal-window"
        :style="{ left: position.x + 'px', top: position.y + 'px'}"
      >
        <div
          class="modal-titlebar"
          @mousedown="handleTitleMouseDown"
        >
          <span class="modal-title">{{ title }}</span>
          <div class="modal-buttons" @mousedown.stop>
            <template v-if="!isPicture">
              <div class="modal-button minimize" @click.stop="isShowContent = false" title="最小化">
                <img src="../assets/minsize-btn.svg" alt="minimize" />
              </div>
              <div class="modal-button maximize" @click.stop="isShowContent = true" :title="isMaximized ? '還原' : '最大化'">
                <img src="../assets/toggle-btn.svg" alt="maximize" />
              </div>
            </template>
            <div class="modal-button close" @click.stop="handleClose" title="關閉">
              <img src="../assets/close-btn.svg" alt="close" />
            </div>
          </div>
        </div>
        <div
          v-show="isShowContent"
          class="modal-content"
          :class="{ 'modal-content-picture': isPicture }"
        >
          <slot></slot>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style scoped lang="scss">
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1000;
  pointer-events: none;
}

.modal-window {
  position: absolute;
  height: fit-content;
  max-width: 600px;
  background: #fff;
  border: 2px solid #dfdfdf;
  box-shadow: -2px -2px 0px 0px #0A0A0A inset, 2px 2px 0px 0px #D1D3D4 inset, -4px -4px 0px 0px rgba(65, 65, 67, 0.75) inset, 4px 4px 0px 0px #FFF inset;
  display: flex;
  flex-direction: column;
  image-rendering: pixelated;
  text-rendering: optimizeSpeed;
  -webkit-font-smoothing: none;
  -moz-osx-font-smoothing: auto;
  pointer-events: auto;

  &.maximized {
    width: 90vw !important;
    height: 90vh !important;
    left: 5vw !important;
    top: 5vh !important;
  }
}

.modal-titlebar {
  background: linear-gradient(90deg, #2B6991 0%, #1E93D0 40%, #1E93D0 60%, #5BBCDB 100%);;
  color: #fff;
  padding: 4px 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: move;
  user-select: none;
  border-bottom: 2px solid #000;
  font-size: 12px;
  font-weight: bold;
}

.modal-title {
  flex: 1;
  text-shadow: 1px 1px 0px rgba(0, 0, 0, 0.5);
}

.modal-buttons {
  display: flex;
  align-items: center;
  cursor: pointer;
  height: 16px;

  .modal-button {
    height: 16px
  }
}


.modal-content {
  flex: 1;
  padding: 24px;
  overflow: auto;
  background: #fff;
  image-rendering: pixelated;
  text-rendering: optimizeSpeed;
  -webkit-font-smoothing: none;
  -moz-osx-font-smoothing: auto;
  max-height: 400px;
  overflow-y: scroll;

  &.modal-content-picture {
    padding: 0;
    max-height: none;
    overflow: visible;
  }
}
</style>

