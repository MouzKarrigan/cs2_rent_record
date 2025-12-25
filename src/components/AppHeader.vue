<template>
  <div class="app-header">
    <div class="header-left">
      <h1>CS饰品租赁收益</h1>
      <el-button type="primary" @click="showAddItemDialog">添加饰品</el-button>
    </div>
    <div class="header-center">
      <div class="nav-tabs">
        <router-link to="/" class="nav-tab" :class="{ active: activeRoute === '/' }">
          首页
        </router-link>
        <router-link to="/charts" class="nav-tab" :class="{ active: activeRoute === '/charts' }">
          统计图表
        </router-link>
      </div>
    </div>
    <div class="header-right">
      <el-button @click="importData">导入</el-button>
      <el-button @click="exportData">导出</el-button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

// 定义emit事件
const emit = defineEmits(['add-item', 'import-data', 'export-data'])

const activeRoute = computed(() => route.path)

const handleSelect = (index) => {
  // 路由会自动处理导航
}

const showAddItemDialog = () => {
  emit('add-item')
}

const importData = () => {
  emit('import-data')
}

const exportData = () => {
  emit('export-data')
}
</script>

<style scoped>
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background-color: #ffffff;
  border-bottom: 2px solid #e4e7ed;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 20px;
  flex: 1;
}

.header-left h1 {
  margin: 0;
  font-size: 28px;
  color: #303133;
  white-space: nowrap;
}

.header-center {
  flex: 0 0 auto;
  display: flex;
  justify-content: center;
  margin: 0 40px;
}

.nav-tabs {
  display: flex;
  gap: 10px;
  align-items: center;
}

.nav-tab {
  padding: 10px 30px;
  font-size: 16px;
  font-weight: 500;
  color: #606266;
  text-decoration: none;
  border-radius: 4px;
  transition: all 0.3s;
  cursor: pointer;
  white-space: nowrap;
}

.nav-tab:hover {
  color: #409eff;
  background-color: #ecf5ff;
}

.nav-tab.active {
  color: #409eff;
  background-color: #ecf5ff;
  font-weight: 600;
}

.header-right {
  display: flex;
  gap: 10px;
  flex: 1;
  justify-content: flex-end;
}
</style>
