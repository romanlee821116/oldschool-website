<script setup lang="ts">
  import Modal from './Modal.vue'
  import { computed } from 'vue'
  
  const props = defineProps<{
    src: string
    visible: boolean
  }>()

  const emit = defineEmits<{
    close: []
  }>()

  const handleClose = () => {
    emit('close')
  }

  // 確保圖片路徑正確處理
  const imageSrc = computed(() => {
    if (!props.src) return ''
    // 如果已經是完整的 URL 或 data URL，直接返回
    if (props.src.startsWith('http') || props.src.startsWith('data:')) {
      return props.src
    }
    // 如果是相對路徑，確保以 ./ 或 / 開頭
    if (props.src.startsWith('./') || props.src.startsWith('/')) {
      return props.src
    }
    // 否則添加 ./ 前綴
    return './' + props.src.replace(/^\.\//, '')
  })
</script>

<template>
  <Modal
    :isPicture="true"
    :visible="props.visible"
    title="Image"
    @close="handleClose"
  >
    <div class="jpeg-modal-content">
      <img v-if="imageSrc" :src="imageSrc" alt="jpeg" />
    </div>
  </Modal>
</template>

<style scoped lang="scss">
.jpeg-modal-content {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  margin: 0;
  padding: 0;

  img {
    width: auto;
    height: 100%;
    object-fit: contain;
    image-rendering: pixelated;
    display: block;
  }
}
</style>