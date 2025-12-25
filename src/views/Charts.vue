<template>
  <div class="charts-container">
    <el-empty description="统计图表功能开发中，敬请期待..." :image-size="200">
      <template #image>
        <el-icon :size="100" color="#409EFF">
          <TrendCharts />
        </el-icon>
      </template>
    </el-empty>
  </div>
</template>

<script setup>
import { TrendCharts } from '@element-plus/icons-vue'
</script>

<style scoped>
.charts-container {
  padding: 20px;
  padding-top: 110px; /* 为固定的导航栏留出空间 */
  padding-bottom: 120px; /* 为底部统计留出空间 */
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 230px);
}
</style>
