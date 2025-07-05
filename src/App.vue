<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue'
import { Leafer, Rect } from 'leafer-ui'
import lipsticks from './assets/lipstick.json'

const leafer = ref<Leafer | null>(null)
const mainRef = ref<HTMLElement | null>(null)

onMounted(() => {
  if (!mainRef.value) return

  // 确保容器有有效尺寸
  if (mainRef.value.clientWidth === 0 || mainRef.value.clientHeight === 0) {
    mainRef.value.style.width = '100vw'
    mainRef.value.style.height = '100vh'
  }

  // 初始化Leafer
  leafer.value = new Leafer({
    view: mainRef.value,
    fill: '#fff',
    width: mainRef.value.clientWidth,
    height: mainRef.value.clientHeight,
    // 关键：降低 Canvas 的 z-index
    style: { zIndex: 0 },
  })

  nextTick(() => {
    createGradientRect()
  })
})

function createGradientRect() {
  if (!leafer.value) return

  const width = Math.max(leafer.value.width || 0, 100)
  const height = Math.max(leafer.value.height || 0, 100)

  const lipstickRect = new Rect({
    x: 0,
    y: 0,
    width,
    height,
    fill: {
      type: 'linear',
      stops: ['#FF4B4B', '#FEB027'],
      from: { x: 0, y: 0 },
      to: { x: width, y: height },
    },
  })

  leafer.value.add(lipstickRect)
}

onBeforeUnmount(() => {
  if (leafer.value) {
    leafer.value.destroy()
  }
})
</script>

<template>
  <div class="container">
    <div ref="mainRef" class="main"></div>

    <!-- 浮动内容放在 Leafer 容器外部 -->
    <div class="lipsticks-list">123</div>
    <div class="author">霜序廿</div>
  </div>
</template>

<style scoped lang="scss">
.container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.main {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1; // 确保低于浮动元素
}

.lipsticks-list {
  z-index: 10; // 高于 Leafer
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(255, 255, 255, 0.8);
  padding: 10px;
}

.author {
  z-index: 10; // 高于 Leafer
  position: fixed;
  right: 20px;
  bottom: 20px;
  background: rgba(255, 255, 255, 0.8);
  padding: 5px 10px;
  border-radius: 4px;
}
</style>

<style>
/* 全局样式确保 Leafer 的 Canvas 层级正确 */
.leafer-view {
  position: absolute !important;
  z-index: 1 !important; /* 低于浮动元素 */
}
</style>
