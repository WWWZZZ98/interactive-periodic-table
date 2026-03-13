<script setup lang="ts">
import { ref } from 'vue'
import PeriodicTable from './components/periodicTable/index.vue'

const count = ref(0)
const periodicTableIsOpen = ref(true)
const showChemFormula = ref(false)

const handleInput = (value: string) => {
  console.log('输入:', value)
}
</script>

<template>
  <div class="app">
    <h1>元素周期表</h1>
    <div class="controls">
      <el-button type="primary" @click="periodicTableIsOpen = !periodicTableIsOpen">
        {{ periodicTableIsOpen ? '隐藏' : '显示' }}周期表
      </el-button>
      <el-button @click="showChemFormula = !showChemFormula">
        {{ showChemFormula ? '隐藏' : '显示' }}化学式输入
      </el-button>
    </div>
    <PeriodicTable
      :periodic-table-is-open="periodicTableIsOpen"
      :show-chem-formula="showChemFormula"
      @on-input="handleInput"
    />
  </div>
</template>

<style scoped>
.app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  padding: 20px;
}

h1 {
  margin-bottom: 20px;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  justify-content: center;
  gap: 10px;
}
</style>
