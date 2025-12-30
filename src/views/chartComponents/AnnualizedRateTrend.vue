<template>
  <div class="annualized-rate-trend">
    <h2 class="chart-title">综合年利率趋势</h2>
    <div class="chart-wrapper">
      <v-chart 
        class="chart" 
        :option="chartOption" 
        autoresize
        :loading="loading"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, inject, onMounted } from 'vue'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  LegendComponent
} from 'echarts/components'
import VChart from 'vue-echarts'

// 注册必要的 echarts 组件
use([
  CanvasRenderer,
  LineChart,
  TitleComponent,
  TooltipComponent,
  GridComponent,
  LegendComponent
])

// 从父组件注入数据
const items = inject('items')

// 加载状态
const loading = ref(true)

// 计算某个饰品在特定日期的拥有天数
const calculateOwnershipDaysAtDate = (item, date) => {
  const purchaseDate = new Date(item.purchaseDate)
  if (date < purchaseDate) return 0
  
  // 筛选出在指定日期之前或当天已开始的租赁记录
  const relevantRecords = item.rentalRecords.filter(record => {
    return new Date(record.startDate) <= date
  })
  
  // 如果没有相关租赁记录，拥有日数为购买日期至指定日期
  if (!relevantRecords || relevantRecords.length === 0) {
    const diffTime = Math.abs(date - purchaseDate)
    return Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  }
  
  // 找出这些记录中结束日期最晚的那个
  const sortedRecords = [...relevantRecords].sort((a, b) => {
    return new Date(b.endDate) - new Date(a.endDate)
  })
  const latestRecord = sortedRecords[0]
  const latestEndDate = new Date(latestRecord.endDate)
  
  // 如果最新租赁记录的到期时间早于或等于指定日期（已归还）
  if (latestEndDate <= date) {
    // 拥有日数为购买日期至指定日期
    const diffTime = Math.abs(date - purchaseDate)
    return Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  } else {
    // 如果最新租赁记录的到期时间晚于指定日期（未归还）
    // 拥有日数为购买日期至最终归还日期
    const diffTime = Math.abs(latestEndDate - purchaseDate)
    return Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  }
}

// 计算某个饰品在特定日期的总租金收入
const calculateTotalRentalIncomeAtDate = (item, date) => {
  // 只计算开始日期在指定日期之前或当天的租赁记录
  return item.rentalRecords
    .filter(record => new Date(record.startDate) <= date)
    .reduce((sum, rental) => sum + rental.netProfit, 0)
}

// 计算某个饰品在特定日期的年化收益率
const calculateAnnualizedRateAtDate = (item, date) => {
  const totalIncome = calculateTotalRentalIncomeAtDate(item, date)
  const ownershipDays = calculateOwnershipDaysAtDate(item, date)
  
  if (ownershipDays === 0 || item.purchasePrice === 0) return 0
  
  const profitRate = (totalIncome / item.purchasePrice)
  const annualizedRate = (profitRate / ownershipDays) * 365 * 100
  
  return annualizedRate
}

// 计算总年化收益率在特定日期
const calculateTotalAnnualizedRateAtDate = (date) => {
  const relevantItems = items.value.filter(item => {
    const purchaseDate = new Date(item.purchaseDate)
    return purchaseDate <= date
  })
  
  if (relevantItems.length === 0) return 0
  
  let weightedSum = 0
  let totalWeight = 0
  
  relevantItems.forEach(item => {
    const weight = item.purchasePrice
    const annualizedRate = calculateAnnualizedRateAtDate(item, date)
    weightedSum += annualizedRate * weight
    totalWeight += weight
  })
  
  if (totalWeight === 0) return 0
  return weightedSum / totalWeight
}

// 生成所有需要计算的日期点
const generateDatePoints = () => {
  if (!items.value || items.value.length === 0) return []
  
  const dateSet = new Set()
  const today = new Date()
  today.setHours(0, 0, 0, 0)
  
  // 收集所有购入日期和租赁结束日期
  items.value.forEach(item => {
    const purchaseDate = new Date(item.purchaseDate)
    purchaseDate.setHours(0, 0, 0, 0)
    dateSet.add(purchaseDate.getTime())
    
    item.rentalRecords.forEach(record => {
      const endDate = new Date(record.endDate)
      endDate.setHours(0, 0, 0, 0)
      if (endDate <= today) {
        dateSet.add(endDate.getTime())
      }
    })
  })
  
  // 添加今天
  dateSet.add(today.getTime())
  
  // 转换为数组并排序
  const dates = Array.from(dateSet)
    .map(timestamp => new Date(timestamp))
    .sort((a, b) => a - b)
  
  return dates
}

// 生成图表数据
const generateChartData = () => {
  const datePoints = generateDatePoints()
  
  const dates = []
  const rates = []
  
  datePoints.forEach(date => {
    const rate = calculateTotalAnnualizedRateAtDate(date)
    dates.push(formatDate(date))
    rates.push(rate.toFixed(2))
  })
  
  return { dates, rates }
}

// 格式化日期
const formatDate = (date) => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

// 图表配置
const chartOption = computed(() => {
  const { dates, rates } = generateChartData()
  
  return {
    tooltip: {
      trigger: 'axis',
      formatter: (params) => {
        const date = params[0].name
        const rate = params[0].value
        return `日期: ${date}<br/>总年化收益率: ${rate}%`
      }
    },
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      top: '10%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: dates,
      boundaryGap: false,
      axisLabel: {
        rotate: 45,
        formatter: (value) => {
          // 只显示月-日
          const parts = value.split('-')
          return `${parts[1]}-${parts[2]}`
        }
      }
    },
    yAxis: {
      type: 'value',
      name: '总年化收益率 (%)',
      axisLabel: {
        formatter: '{value}%'
      }
    },
    series: [
      {
        name: '总年化收益率',
        type: 'line',
        data: rates,
        smooth: true,
        lineStyle: {
          width: 2,
          color: '#409EFF'
        },
        itemStyle: {
          color: '#409EFF'
        },
        areaStyle: {
          color: {
            type: 'linear',
            x: 0,
            y: 0,
            x2: 0,
            y2: 1,
            colorStops: [
              {
                offset: 0,
                color: 'rgba(64, 158, 255, 0.3)'
              },
              {
                offset: 1,
                color: 'rgba(64, 158, 255, 0.05)'
              }
            ]
          }
        }
      }
    ]
  }
})

onMounted(() => {
  // 模拟加载延迟
  setTimeout(() => {
    loading.value = false
  }, 300)
})
</script>

<style scoped>
.annualized-rate-trend {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart-title {
  font-size: 20px;
  font-weight: 600;
  color: #303133;
  margin: 0 0 20px 0;
  padding-left: 10px;
  border-left: 4px solid #409EFF;
}

.chart-wrapper {
  flex: 1;
  min-height: 400px;
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.chart {
  width: 100%;
  height: 100%;
  min-height: 400px;
}
</style>
