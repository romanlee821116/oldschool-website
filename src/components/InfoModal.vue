<script setup lang="ts">
  import Modal from './Modal.vue'
  import { onMounted, onUnmounted, ref } from 'vue'

  const props = defineProps<{
    visible: boolean
  }>()

  const emit = defineEmits<{
    close: []
  }>()

  const initialX = ref(0)
  const initialY = ref(100)

  const updatePosition = () => {
    // 計算右邊位置：視窗寬度 - modal寬度(600) - 邊距(50)
    initialX.value = window.innerWidth - 650
    initialY.value = 100
  }

  onMounted(() => {
    updatePosition()
    window.addEventListener('resize', updatePosition)
  })

  onUnmounted(() => {
    window.removeEventListener('resize', updatePosition)
  })

  const handleClose = () => {
    emit('close')
  }
</script>

<template>
  <Modal :visible="props.visible" :initialX="initialX" :initialY="initialY" title="Info" @close="handleClose">
    <div>
      <div class="info-modal-content">
        I'M A CAT!
      </div>
    </div>
  </Modal>
</template>

<style scoped lang="scss">

</style>