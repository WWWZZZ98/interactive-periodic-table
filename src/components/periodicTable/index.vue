<script setup>
import { ref, computed } from 'vue'
import elements from '@/data/elements.json'
import { RefreshLeft } from '@element-plus/icons-vue'

const emit = defineEmits(['onInput'])
const props = defineProps({
  periodicTableIsOpen: {
    type: Boolean,
    default: true
  },
  showChemFormula: {
    type: Boolean,
    default: false
  }
})

const searchTerm = ref('')
const selectedElements = ref([])
// const periodicTableIsOpen = ref(true)
const selectedType = ref('仅含有所选元素')

const numberButtons = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '(', ')', '*']

// const isOpenPeriodicTableShow = () => {
//   periodicTableIsOpen.value = !periodicTableIsOpen.value
// }

const goSearch = () => {
  if (!search.value) return
  $router.push({
    name: 'search',
    query: {
    },
  })
}
// 主表元素（不包括镧系和锕系）
const mainTableElements = computed(() => {
  return elements.filter(el =>
    !(el.atomicNumber >= 57 && el.atomicNumber <= 71) &&
    !(el.atomicNumber >= 89 && el.atomicNumber <= 103)
  )
})

// 镧系元素
const lanthanideElements = computed(() => {
  return elements.filter(el => el.atomicNumber >= 57 && el.atomicNumber <= 71)
})

// 锕系元素
const actinideElements = computed(() => {
  return elements.filter(el => el.atomicNumber >= 89 && el.atomicNumber <= 103)
})

function handleElementClick(element) {
  searchTerm.value += element.symbol.toString()
  emit('onInput', element.symbol)
}

function appendToFormula(value) {
  searchTerm.value += value.toString()
  emit('onInput', value)
}

function resetSearch() {
  searchTerm.value = ''
  selectedElements.value = []
}

function getGridPosition(el) {
  return {
    gridColumn: el.group,
    gridRow: el.period,
  }
}

// 添加浮框位置计算函数
function getTooltipPosition(element) {
  // 默认位置在元素右侧
  let position = { top: '-25px', left: '60px', right: 'auto', bottom: 'auto' };

  // 根据元素在周期表中的位置调整浮框位置
  if (element.group > 14) {  // 如果元素在右侧
    position.left = 'auto';
    position.right = '60px';
  }

  // 处理上下边缘
  if (element.period < 2) {  // 如果元素在顶部
    position.top = '0px';
  } else if (element.period > 6) {  // 如果元素在底部
    position.top = 'auto';
    position.bottom = '0px';
  }

  // 处理镧系和锕系元素的特殊情况
  if (element.atomicNumber >= 57 && element.atomicNumber <= 71) {
    // 镧系元素
    const index = element.atomicNumber - 57;
    if (index > 10) { // 右侧的镧系元素
      position.left = 'auto';
      position.right = '60px';
    }
    position.top = '-25px'; // 镧系元素通常显示在上方
  } else if (element.atomicNumber >= 89 && element.atomicNumber <= 103) {
    // 锕系元素
    const index = element.atomicNumber - 89;
    if (index > 10) { // 右侧的锕系元素
      position.left = 'auto';
      position.right = '60px';
    }
    position.bottom = '0'; // 锕系元素通常显示在上方
  }

  return position;
}

defineExpose({
  mainTableElements
})
</script>


<template>
  <div class="table-container">
    <div class="periodic-table-home" v-if="periodicTableIsOpen">
      <div class="filter-buttons" v-if="showChemFormula">
        <div>
          <el-radio-group class="el-radio-group-style" v-model="selectedType" size="large">
            <!-- <el-radio-button class="el-radio-button-style" label="仅含有所选元素" value="仅含有所选元素" /> -->
            <!-- <el-radio-button class="el-radio-button-style" label="至少含有所选元素" value="至少含有所选元素" /> -->
            <el-radio-button class="el-radio-button-style" label="当前化学式" value="当前化学式" />
          </el-radio-group>
        </div>
        <!-- 数字按钮组 -->
          <div class="number-buttons" v-if="selectedType === '当前化学式'">
          <el-button class="num-btn" v-for="btn in numberButtons" :key="btn" @click="appendToFormula(btn)">{{ btn }}</el-button>
        </div>
      </div>
      <!-- 元素周期表 -->
      <div class="periodic-table">
        <div
          v-for="element in mainTableElements"
          :key="element.atomicNumber"
          class="element-box"
          :style="getGridPosition(element)"
          @click="handleElementClick(element)"
          :class="{
          [`category-${element.category}`]: true
        }"
        >
          <div class="element-content">
            <div class="element-symbol" :style="{ color: element.atomicNumber >= 84 ? '#A83234' : ''}">{{ element.symbol }}</div>
            <div class="element-name">{{ element.zhName }}<span v-if="element.atomicNumber >= 99">*</span></div>
          </div>
          <div class="element-tooltip" style="display: none;" :style="getTooltipPosition(element)">
            <!--          <div class="element-tooltip-arrow"></div>-->
            <div class="tooltip-symbol">{{ element.symbol }}</div>
            <div class="tooltip-number">{{ element.atomicNumber }}</div>
            <div class="tooltip-zh-name">{{ element.zhName }}</div>
            <div class="tooltip-en-name">{{ element.enName }}</div>
            <div class="tooltip-mass">{{ element.atomicWeight }}</div>
          </div>
        </div>

        <!-- 镧系和锕系标识 -->
        <div class="lanthanide-label" :style="{ gridColumn: '3', gridRow: '6' }">
          <div class="element-content">
            <div class="element-symbol">La-Lu</div>
            <div class="element-name">镧系</div>
          </div>
        </div>
        <div class="actinide-label" :style="{ gridColumn: '3', gridRow: '7' }">
          <div class="element-content">
            <div class="element-symbol" style="color: #A83234">Ac-Lr</div>
            <div class="element-name">锕系</div>
          </div>
        </div>
      </div>

      <!-- 镧系元素 -->
      <div class="lanthanide-series">
        <div></div>
        <div
          v-for="element in lanthanideElements"
          :key="element.atomicNumber"
          class="element-box"
          @click="handleElementClick(element)"
          :class="{
          'category-镧系': true
        }"
        >
          <div class="element-content">
            <div class="element-symbol">{{ element.symbol }}</div>
            <div class="element-name">{{ element.zhName }}</div>
          </div>
          <div class="element-tooltip" style="display: none;" :style="getTooltipPosition(element)">
            <!--          <div class="element-tooltip-arrow"></div>-->
            <div class="tooltip-symbol">{{ element.symbol }}</div>
            <div class="tooltip-number">{{ element.atomicNumber }}</div>
            <div class="tooltip-zh-name">{{ element.zhName }}</div>
            <div class="tooltip-en-name">{{ element.enName }}</div>
            <div class="tooltip-mass">{{ element.atomicWeight }}</div>
          </div>
        </div>
      </div>

      <!-- 锕系元素 -->
      <div class="actinide-series">
        <div></div>
        <div
          v-for="element in actinideElements"
          :key="element.atomicNumber"
          class="element-box"
          @click="handleElementClick(element)"
          :class="{
          'category-锕系': true
        }"
        >
          <div class="element-content">
            <div class="element-symbol" style="color: #A83234">{{ element.symbol }}</div>
            <div class="element-name">{{ element.zhName }}<span v-if="element.atomicNumber >= 99">*</span></div>
          </div>
          <div class="element-tooltip" style="display: none;" :style="getTooltipPosition(element)">
            <!--          <div class="element-tooltip-arrow"></div>-->
            <div class="tooltip-symbol">{{ element.symbol }}</div>
            <div class="tooltip-number">{{ element.atomicNumber }}</div>
            <div class="tooltip-zh-name">{{ element.zhName }}</div>
            <div class="tooltip-en-name">{{ element.enName }}</div>
            <div class="tooltip-mass">{{ element.atomicWeight }}</div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped lang="scss">
.table-container {
  padding: 0 0 20px 0;
  background-color: white;
  font-family: MicrosoftYaHei sans-serif;
  position: relative;
  max-width: 960px;
  // width: 100%;
  //min-height: calc(100vh - 290px);
  margin: 0 auto;
  box-sizing:border-box;
}

/* 顶部导航栏样式 */
//.header {
//  display: flex;
//  justify-content: center;
//  align-items: center;
//  padding: 10px 20px;
//  border-bottom: 1px solid #eee;
//  background-color: #f8f8f8;
//}
.search-box {
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
  :deep(.el-input) {
    .el-input__wrapper {
      padding: 1px 5px 1px 20px;
      border: 1px solid rgba(229,229,229,1);
      border-radius: 8px;

      .el-input__suffix {
        .el-input__suffix-inner {
          .search-btn {
            font-size: 16px;
            color: #1760C2;
            letter-spacing: 0;
            font-weight: 400;
            .el-icon {
              margin-right: 10px;
            }
            .buttonImg {
              height: 36px;
              width: 71px;
            }
          }
        }
      }
    }
  }
}


/* 筛选按钮组样式 */
.filter-buttons {
  display: flex;
  padding: 14px 0 0;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: flex-start;
  align-items: center;
  width: 936px;
  margin: 0 auto;
}
//.el-radio-group-style {
//  background: #FFFFFF;
//  border: 1px solid rgba(23,96,194,1);
//  border-radius: 5px;
//}
//
//.el-radio-button-style:not(.is-active) {
//  :deep(.el-radio-button__inner:nth-of-type(1)) {
//    border-left: 1px solid rgba(23,96,194,1);
//  }
//  :deep(.el-radio-button__inner) {
//    border-top: 1px solid rgba(23,96,194,1);
//    border-bottom: 1px solid rgba(23,96,194,1);
//    border-right: 1px solid rgba(23,96,194,1);
//    font-family: MicrosoftYaHei, sans-serif;
//    font-size: 14px;
//    color: #1760C2 !important;
//    letter-spacing: 0;
//    font-weight: 400;
//    line-height: 19px;
//  }
//}
.el-radio-button-style:not(.is-active) {
  :deep(.el-radio-button__inner) {
    font-family: MicrosoftYaHei, sans-serif;
    font-size: 14px;
    color: #1760C2 !important;
    letter-spacing: 0;
    font-weight: 400;
    border: 1px solid #1760c2;
    border-left: 0;
  }
}
.el-radio-button-style:not(.is-active):nth-of-type(1) {
  :deep(.el-radio-button__inner) {
    border-left: 1px solid #1760c2;
  }
}

/* 数字按钮组样式 */
.number-buttons {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.num-btn {
  margin-left: 0;
  height: 40px;
  width: 34px;
  font-family: MicrosoftYaHei, sans-serif;
  font-size: 16px;
  color: #333333;
  letter-spacing: 0;
  text-align: center;
  font-weight: 400;
}

/* 化学式显示区域 */
.formula-display {
  display: flex;
  padding: 10px 15px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #eee;
  align-items: center;
}

.formula-text {
  flex: 1;
  font-size: 16px;
  font-family: monospace;
  padding: 5px 10px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  min-height: 20px;
}

.clear-btn {
  margin-left: 10px;
  padding: 5px 10px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.clear-btn:hover {
  background: #d32f2f;
}

.periodic-table-home {
  width: 960px;
  border: 1px solid rgba(229,229,229,1);
  border-radius: 8px;
}

/* 元素周期表样式 */
.periodic-table {
  display: grid;
  grid-template-columns: repeat(18, 48px);
  grid-auto-rows: 48px;
  gap: 4px;
  justify-content: center;
  padding: 14px 14px 0;
  background-color: #ffffff;
  position: relative;
}

.lanthanide-label, .actinide-label {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #333333;
  background-color: #A0D8F6;
  border: 1px dashed #ccc;
  border-radius: 4px;
  //padding: 2px;
  // height: 48px;
}

.lanthanide-label .element-content,
.actinide-label .element-content {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  padding: 5px;
  justify-content: center;
}

.lanthanide-label .element-symbol,
.actinide-label .element-symbol {
  font-size: 12px;
  font-weight: bold;
  line-height: 1;
  margin-bottom: 4px;
  text-align: left;
  align-self: flex-start;
}

.lanthanide-label .element-name,
.actinide-label .element-name {
  font-size: 16px;
  color: #333;
  line-height: 1.2;
  text-align: center;
  width: 100%;
}

.lanthanide-series {
  display: grid;
  grid-template-columns: repeat(16, 48px);
  gap: 4px;
  padding: 8px 14px 4px;
  justify-content: center;
}
.actinide-series {
  display: grid;
  grid-template-columns: repeat(16, 48px);
  gap: 4px;
  padding: 0 14px 14px;
  justify-content: center;
}

.element-box {
  border: 1px solid #ccc;
  border-radius: 4px;
  text-align: center;
  position: relative;
  padding: 2px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  // height: 48px;
}

.element-box:hover {
  z-index: 100;
  transform: scale(1.05);
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}
.element-box:active {
  transform: scale(0.95);
}

/* 移除之前的详细信息显示方式 */
.element-box:hover::before {
  content: none;
}

/* 添加浮框内容容器 */
.element-box:hover .element-tooltip {
  display: flex !important;
  position: absolute;
  width: 100px;
  height: 100px;
  background-color: #E1ECFC;
  border-radius: 4px;
  box-shadow: 0px 2px 8px 0px rgba(0,0,0,0.5);
  z-index: 101;
  flex-direction: column;
  padding: 5px;
  box-sizing: border-box;
  pointer-events: none;
}

/* 添加小三角形指示器 */
.element-tooltip-arrow {
  position: absolute;
  width: 0;
  height: 0;
  background-color: #E1ECFC;
  transform: rotate(45deg);
  box-shadow: 0px 2px 8px 0px rgba(0,0,0,0.2);
  z-index: 100;
}

/* 根据位置添加不同方向的箭头 - 使用更简单的选择器 */
.element-box:hover .element-tooltip .element-tooltip-arrow {
  width: 10px;
  height: 10px;
  top: 45px;
  left: -5px;
}

/* 浮框内元素符号样式 */
.element-box:hover .tooltip-symbol {
  position: absolute;
  top: 5px;
  left: 5px;
  font-size: 12px;
  font-weight: 400;
  color:  #333333;
  letter-spacing: 0;
  line-height: normal;
}

/* 浮框内元素序号样式 */
.element-box:hover .tooltip-number {
  position: absolute;
  top: 5px;
  right: 5px;
  font-size: 12px;
  font-weight: 400;
  color:  #333333;
  letter-spacing: 0;
  line-height: normal;
}

/* 浮框内中文名称样式 */
.element-box:hover .tooltip-zh-name {
  font-size: 29px;
  text-align: center;
  margin-top: 15px;
  font-weight: bold;
  color:  #333333;
  letter-spacing: 0;
  line-height: normal;
}

/* 浮框内英文名称样式 */
.element-box:hover .tooltip-en-name {
  font-size: 12px;
  text-align: center;
  letter-spacing: 0;
  line-height: normal;
  color:  #333333;
}

/* 浮框内相对原子质量样式 */
.element-box:hover .tooltip-mass {
  font-size: 12px;
  text-align: center;
  letter-spacing: 0;
  line-height: normal;
  color:  #333333;
}

/* 隐藏原来的元素序号 */
.element-box .element-number {
  display: none;
}

.element-content {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  padding: 5px;
  justify-content: center;
}

.element-symbol {
  font-size: 12px;
  font-weight: bold;
  line-height: 1;
  margin-bottom: 4px;
  text-align: left;
  align-self: flex-start;
}

.element-name {
  font-size: 16px;
  color: #333;
  line-height: 1.2;
  text-align: center;
  width: 100%;
}

/* 移除元素序号样式 */
.element-number {
  font-size: 10px;
  position: absolute;
  bottom: 3px;
  right: 5px;
  color: #666;
}

/* 元素类别颜色 - 与图片完全一致 */
.category-稀有气体 {
  background-color: #F5B4CF;
}

.category-碱金属 {
  background-color: #ccffcc;
}

.category-碱土金属 {
  background-color: #ccccff;
}

.category-过渡金属 {
  background-color: #ccf2ff;
}

.category-非金属 {
  background-color: #5CC2D0;
}

.category-卤素 {
  background-color: #ffccff;
}

.category-类金属 {
  background-color: #e6ffcc;
}

.category-镧系 {
  background-color: #A0D8F6;
}

.category-锕系 {
  background-color: #A0D8F6;
}
.category-金属 {
  background-color: #A0D8F6;
}

</style>