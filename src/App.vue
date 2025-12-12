<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import InfoModal from './components/InfoModal.vue'
import PhotoModal from './components/PhotoModal.vue'
import JpegModal from './components/JpegModal.vue'
// @ts-ignore
import fileImg from './assets/file.png'
// @ts-ignore
import infoImg from './assets/info.png'
// @ts-ignore
import photoImg from './assets/photo.png'
// @ts-ignore
import wallpaper1 from './assets/wallpaper-1.jpeg'
// @ts-ignore
import wallpaper2 from './assets/wallpaper-2.jpeg'
// @ts-ignore
import wallpaper3 from './assets/wallpaper-3.jpeg'

interface ImagePosition {
  id: string
  src: string
  alt: string
  x: number
  y: number
}

const images = ref<ImagePosition[]>([
  { id: 'file', src: fileImg, alt: 'Project', x: 0, y: 0 },
  { id: 'info', src: infoImg, alt: 'Info', x: 0, y: 100 },
  { id: 'photo', src: photoImg, alt: 'Photo', x: 0, y: 200 }
])

const imagesContainer = ref<HTMLElement | null>(null)
let draggedImage: ImagePosition | null = null
let dragOffset = { x: 0, y: 0 }

const isShowInfoModal = ref(true)
const isShowPhotoModal = ref(true)
const imageSourceList = ref<string[]>([])

const closeJpegModal = (imageSrc: string) => {
  imageSourceList.value = imageSourceList.value.filter(src => src !== imageSrc)
}

const photos = ref([
  { src: wallpaper1, alt: 'photo1' },
  { src: wallpaper2, alt: 'photo2' },
  { src: wallpaper3, alt: 'photo3' }
])

const handleMouseDown = (event: MouseEvent, image: ImagePosition) => {
  draggedImage = image
  const target = event.target as HTMLElement
  const rect = target.getBoundingClientRect()
  dragOffset.x = event.clientX - rect.left
  dragOffset.y = event.clientY - rect.top
  event.preventDefault()
}

const handleMouseMove = (event: MouseEvent) => {
  if (draggedImage && imagesContainer.value) {
    const containerRect = imagesContainer.value.getBoundingClientRect()
    draggedImage.x = event.clientX - containerRect.left - dragOffset.x
    draggedImage.y = event.clientY - containerRect.top - dragOffset.y
  }
}

const handleMouseUp = () => {
  draggedImage = null
}

const showModal = (type: string) => {
  console.log(type)
  switch (type) {
    case 'info':
      isShowInfoModal.value = true
      break
    case 'photo':
      isShowPhotoModal.value = true
      break
    default:
      break
  }
}

const handleImageClick = (src: string) => {
  imageSourceList.value.push(src)
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
  <div id="app">
    <div class="container">
      <div class="name blocky">Sandy.c</div>
      <div class="images" ref="imagesContainer">
        <div
          v-for="image in images"
          :key="image.id"
          class="image"
          :style="{ left: image.x + 'px', top: image.y + 'px' }"
          @mousedown="handleMouseDown($event, image)"
          @dblclick="showModal(image.id)"
        >
          <img
            :src="image.src"
            :alt="image.alt"
          />
          <div class="image-name">
            {{ image.alt }}
          </div>
        </div>
      </div>
    </div>

    <InfoModal
      :visible="isShowInfoModal"
      @close="isShowInfoModal = false"
    />
    <PhotoModal
      :visible="isShowPhotoModal"
      :photos="photos"
      @close="isShowPhotoModal = false"
      @imageClick="handleImageClick"
    />
    <JpegModal
      v-for="imageSrc in imageSourceList"
      :key="imageSrc"
      :visible="imageSourceList.includes(imageSrc)"
      :src="imageSrc"
      @close="closeJpegModal(imageSrc)"
    />
  </div>
</template>

<style scoped lang="scss">
#app {
  margin: 0;
  padding: 0;
  width: 100vw;
  min-height: 100vh;
  background-image: url('./assets/bg.png');
  background-repeat: repeat;
  background-size: 15%;
  position: relative;
}

.blocky{
  font-weight: 900;
  font-size: 48px;
  color: white;
  position: relative;
  /* 以多個偏移量形成方塊陰影 (示範) */
  text-shadow:
    2px 0 0 #000,
    -2px 0 0 #000,
    0 2px 0 #000,
    0 -2px 0 #000,
    3px 0 0 #000,
    -3px 0 0 #000;
  /* 你可以繼續添加更多偏移來加粗邊緣，或把字變成方塊狀 */
  display: inline-block;
  padding: 6px;
}

.container {
  position: relative;
  z-index: 1;
  padding: 40px;
  max-width: 1200px;
  margin: 0 auto;
  
  .name {
    text-rendering: optimizeSpeed;
    -webkit-font-smoothing: none;
    -moz-osx-font-smoothing: auto;
    image-rendering: pixelated;
    font-size: 120px;
    font-weight: 900;
    color: #000;
    letter-spacing: -1px;
  }

  .images {
    position: relative;
    width: 100%;
    min-height: 200px;
    margin-top: 40px;

    .image {
      width: 100px;
      height: 100px;
      object-fit: contain;
      position: absolute;
      cursor: move;
      image-rendering: pixelated;
      user-select: none;
      pointer-events: auto;
      text-align: center;

      &:active {
        cursor: grabbing;
      }

      img {
        width: 100%;
        height: 100%;
        display: block;
        object-fit: contain;
      }

      .image-name {
        background-color: white;
        font-size: 12px;
        width: fit-content;
        margin: -24px auto 0 auto;
        padding: 2px 4px;
      }
    }
  }
}

.modal-file-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 20px;
  padding: 10px;
}

.modal-file-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  image-rendering: pixelated;
  text-rendering: optimizeSpeed;
  -webkit-font-smoothing: none;
  -moz-osx-font-smoothing: auto;

  &:hover {
    opacity: 0.8;
  }
}

.modal-file-thumbnail {
  width: 80px;
  height: 80px;
  object-fit: contain;
  border: 1px solid #ccc;
  background: #fff;
  margin-bottom: 8px;
  image-rendering: pixelated;
}

.modal-file-name {
  font-size: 11px;
  color: #000;
  text-align: center;
  word-break: break-all;
  max-width: 100px;
}

@media (max-width: 768px) {
  .container {
    padding: 20px;
    
    .name {
      font-size: 36px;
      margin-bottom: 40px;
    }
  }
}
</style>
