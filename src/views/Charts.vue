<template>
  <div class="charts-container">
    <!-- 左侧导航栏 -->
    <div class="sidebar">
      <el-menu
        :default-active="activeTab"
        class="sidebar-menu"
        @select="handleTabSelect"
      >
        <el-menu-item index="annualizedRateTrend">
          <el-icon><TrendCharts /></el-icon>
          <span>综合年利率趋势</span>
        </el-menu-item>
      </el-menu>
    </div>

    <!-- 右侧内容区域 -->
    <div class="content">
      <component :is="currentComponent" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { TrendCharts } from '@element-plus/icons-vue'
import AnnualizedRateTrend from './chartComponents/AnnualizedRateTrend.vue'

// 当前激活的tab
const activeTab = ref('annualizedRateTrend')

// 组件映射
const componentMap = {
  annualizedRateTrend: AnnualizedRateTrend
}

// 当前显示的组件
const currentComponent = computed(() => {
  return componentMap[activeTab.value] || null
})

// 处理tab选择
const handleTabSelect = (index) => {
  activeTab.value = index
}
</script>

<style scoped>
.charts-container {
  display: flex;
  padding-top: 90px; /* 为固定的导航栏留出空间 */
  padding-bottom: 100px; /* 为底部统计留出空间 */
  min-height: calc(100vh - 190px);
  background-color: #f5f7fa;
}

.sidebar {
  width: 220px;
  background: white;
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.1);
  position: fixed;
  top: 90px;
  left: 0;
  bottom: 100px;
  overflow-y: auto;
  z-index: 10;
}

.sidebar-menu {
  border-right: none;
  height: 100%;
}

.sidebar-menu .el-menu-item {
  height: 56px;
  line-height: 56px;
  font-size: 15px;
}

.sidebar-menu .el-menu-item .el-icon {
  font-size: 18px;
  margin-right: 8px;
}

.sidebar-menu .el-menu-item:hover {
  background-color: #ecf5ff;
}

.sidebar-menu .el-menu-item.is-active {
  background-color: #e6f4ff;
  color: #409EFF;
  border-right: 3px solid #409EFF;
}

.content {
  flex: 1;
  margin-left: 220px;
  padding: 20px;
  min-height: 100%;
}
</style>
