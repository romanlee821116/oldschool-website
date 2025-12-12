<script setup lang="ts">
  import Modal from './Modal.vue'
  import { onMounted, onUnmounted, ref } from 'vue'

  interface PhotoItem {
    src: string
    alt: string
  }

  const props = defineProps<{
    visible: boolean
    photos?: PhotoItem[]
  }>()

  const emit = defineEmits<{
    close: []
    imageClick: [src: string]
  }>()

  const initialX = ref(0)
  const initialY = ref(0)

  const updatePosition = () => {
    // 計算中間下方位置：x = (視窗寬度 - modal寬度) / 2，y = 視窗高度 - modal高度估算 - 邊距
    initialX.value = (window.innerWidth - 600) / 2  - 100
    initialY.value = window.innerHeight - 400 // 假設 modal 高度約 400px，下方留 100px 邊距
  }

  onMounted(() => {
    updatePosition()
    window.addEventListener('resize', updatePosition)
  })

  onUnmounted(() => {
    window.removeEventListener('resize', updatePosition)
  })

  const handleImageClick = (src: string) => {
    emit('imageClick', src)
  }

  const handleClose = () => {
    emit('close')
  }
</script>

<template>
  <Modal :visible="props.visible" :initialX="initialX" :initialY="initialY" title="Photo" @close="handleClose">
    <div>
      <div class="photo-modal-content">
        <div v-for="photo in props.photos" :key="photo.src">
          <img 
            :src="photo.src" 
            :alt="photo.alt" 
            @click="handleImageClick(photo.src)"
          />
          <p>{{ photo.alt }}</p>
        </div>
      </div>
    </div>
  </Modal>
</template>

<style scoped lang="scss">
.photo-modal-content {
  display: flex;
  flex-wrap: wrap;
  gap: 32px;

  img {
    width: 120px;
    height: 120px;
    object-fit: cover;
    display: block;
    cursor: pointer;

    &:hover {
      border: 1px dashed #000;
    }
  }

  p {
    text-align: center;
    font-size: 12px;
    margin-top: 8px;
  }
}
</style>