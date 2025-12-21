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

  // 判斷是否為影片格式
  const isVideo = computed(() => {
    if (!props.src) return false
    const videoExtensions = ['.mp4', '.webm', '.ogg', '.mov', '.avi', '.wmv', '.flv', '.mkv']
    const lowerSrc = props.src.toLowerCase()
    return videoExtensions.some(ext => lowerSrc.endsWith(ext))
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
      <img v-if="imageSrc && !isVideo" :src="imageSrc" alt="media" />
      <video v-if="imageSrc && isVideo" :src="imageSrc" controls autoplay>
        您的瀏覽器不支援影片播放。
      </video>
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

  img,
  video {
    width: auto;
    height: 100%;
    max-height: 60vh;
    object-fit: contain;
    display: block;
  }

  img {
    image-rendering: pixelated;
  }

  video {
    max-width: 100%;
  }
}
</style>