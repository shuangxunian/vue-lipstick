<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'
import { Leafer } from 'leafer-ui'
import lipsticks from './assets/lipstick.json'

/** —— 1. Leafer 背景画布 —— **/
const leafer = ref<Leafer | null>(null)
const mainRef = ref<HTMLElement | null>(null)

onMounted(() => {
  if (!mainRef.value) return
  // 确保容器有尺寸
  const el = mainRef.value
  if (el.clientWidth === 0 || el.clientHeight === 0) {
    el.style.width = '100vw'
    el.style.height = '100vh'
  }
  leafer.value = new Leafer({
    view: el,
    width: el.clientWidth,
    height: el.clientHeight,
    fill: '#fff',
  })
})

onBeforeUnmount(() => {
  leafer.value?.destroy()
})

/** —— 2. 提取品牌 / 系列 / 色号数据 —— **/
type Lip = { color: string; id: string; name: string }
type Series = { name: string; lipsticks: Lip[] }
type Brand = { name: string; series: Series[] }

const brands = (lipsticks as { brands: Brand[] }).brands

/** 选中状态 **/
const selectedBrand = ref<string>('')
const selectedSeries = ref<string>('')

/** 品牌列表 **/
const brandOptions = computed(() => brands.map((b) => b.name))

/** 动态系列列表 **/
const seriesOptions = computed(() => {
  if (!selectedBrand.value) return []
  const b = brands.find((b) => b.name === selectedBrand.value)
  return b ? b.series.map((s) => s.name) : []
})

/** 根据选中品牌 + 系列 过滤色号 **/
const filteredLipsticks = computed<Lip[]>(() => {
  let list: Lip[] = []
  brands.forEach((b) => {
    if (selectedBrand.value && b.name !== selectedBrand.value) return
    b.series.forEach((s) => {
      if (selectedSeries.value && s.name !== selectedSeries.value) return
      list = list.concat(s.lipsticks)
    })
  })
  return list
})
</script>

<template>
  <!-- —— 筛选控件 —— -->
  <div class="filter-bar">
    <label>
      品牌：
      <select v-model="selectedBrand">
        <option value="">全部</option>
        <option v-for="b in brandOptions" :key="b" :value="b">{{ b }}</option>
      </select>
    </label>
    <label>
      系列：
      <select v-model="selectedSeries" :disabled="!selectedBrand">
        <option value="">全部</option>
        <option v-for="s in seriesOptions" :key="s" :value="s">{{ s }}</option>
      </select>
    </label>
  </div>

  <!-- —— Leafer 背景画布 —— -->
  <div ref="mainRef" class="main-canvas"></div>

  <!-- —— 色块网格 —— -->
  <div class="lipsticks-list">
    <div
      v-for="lip in filteredLipsticks"
      :key="lip.id"
      class="lip-block"
      :style="{ backgroundColor: lip.color }"
      :title="`${lip.color} — ${lip.name}`"
    ></div>
  </div>

  <div class="author">霜序廿</div>
</template>

<style scoped lang="scss">
/* 整体重置 & 布局 */
.main-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 0;
}

/* 筛选栏 */
.filter-bar {
  position: fixed;
  top: 1rem;
  left: 1rem;
  background: rgba(255, 255, 255, 0.8);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  z-index: 10;
  label {
    margin-right: 1rem;
    font-size: 0.9rem;
  }
  select {
    margin-left: 0.3rem;
  }
}

.lipsticks-list {
  position: fixed;
  top: 5rem; /* 留出筛选栏高度 */
  left: 1rem;
  right: 1rem;
  bottom: 1rem;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(40px, 1fr));
  grid-gap: 4px;
  padding: 4px;
  overflow: auto;
  z-index: 5;
}

.lip-block {
  width: 100%;
  padding-bottom: 100%; /* 保持正方形 */
  border: 1px solid #eee;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.1);
  cursor: default;
}

.author {
  position: fixed;
  right: 1rem;
  bottom: 1rem;
  font-size: 0.8rem;
  color: #666;
  z-index: 10;
}
</style>
