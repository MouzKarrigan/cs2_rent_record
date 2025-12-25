<template>
  <div class="app-footer">
    <el-card>
      <div class="stats-row">
        <div class="stat-item">
          <span class="stat-label">总购入价:</span>
          <span class="stat-value">¥{{ totalPurchasePrice.toFixed(2) }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">总租金:</span>
          <span class="stat-value">¥{{ totalRentalIncome.toFixed(2) }}</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">总收益率:</span>
          <span class="stat-value">{{ totalProfitRate.toFixed(2) }}%</span>
        </div>
        <div class="stat-item">
          <span class="stat-label">总年化收益率:</span>
          <span class="stat-value">{{ totalAnnualizedRate.toFixed(2) }}%</span>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'

defineProps({
  totalPurchasePrice: {
    type: Number,
    default: 0
  },
  totalRentalIncome: {
    type: Number,
    default: 0
  },
  totalProfitRate: {
    type: Number,
    default: 0
  },
  totalAnnualizedRate: {
    type: Number,
    default: 0
  }
})
</script>

<style scoped>
.app-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #ffffff;
  padding: 20px;
  box-shadow: 0 -2px 12px 0 rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

.stats-row {
  display: flex;
  justify-content: space-around;
  align-items: center;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.stat-label {
  font-size: 14px;
  color: #909399;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #409eff;
}
</style>
