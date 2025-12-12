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
      <h1>Hi,</h1>
      <div class="info-modal-content">
        I am a dedicated CRM Specialist at BMW, driven by a passion for delivering premium customer experiences that reflect the brand’s reputation for excellence. With a background in customer relationship management and data-driven marketing, I focus on understanding customer behavior, identifying engagement opportunities, and ensuring every interaction aligns with BMW’s standard of precision and luxury. My daily work involves analyzing customer feedback, monitoring journey touchpoints, and coordinating with sales and service teams to enhance satisfaction and retention. I take pride in transforming raw data into actionable insights and building communication strategies that strengthen long-term loyalty. For me, CRM is not just about systems—it’s about creating meaningful and personalized experiences for every customer who chooses the BMW brand.
      </div>
    </div>
  </Modal>
</template>

<style scoped lang="scss">

</style>