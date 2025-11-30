<template>
  <div class="app-container">
    <!-- 页面头部 -->
    <div class="header">
      <div class="header-left">
        <h1>CS饰品租赁收益</h1>
        <el-button type="primary" @click="showAddItemDialog">添加饰品</el-button>
      </div>
      <div class="header-right">
        <el-button @click="importData">导入</el-button>
        <el-button @click="exportData">导出</el-button>
      </div>
    </div>

    <!-- 饰品列表 -->
    <div class="content">
      <el-table
        :data="items"
        style="width: 100%"
        @expand-change="handleExpandChange"
      >
        <el-table-column type="expand">
          <template #default="{ row }">
            <div class="rental-records">
              <el-table
                :data="getRentalRecordsPage(row)"
                style="width: 100%; margin-left: 50px;"
              >
                <el-table-column label="出租时间段" prop="period" width="300">
                  <template #default="{ row: rental }">
                    {{ formatDate(rental.startDate) }} ~ {{ formatDate(rental.endDate) }}
                    ({{ rental.days }}天)
                  </template>
                </el-table-column>
                <el-table-column label="日租金" prop="dailyRent" width="120">
                  <template #default="{ row: rental }">
                    ¥{{ rental.dailyRent.toFixed(2) }}
                  </template>
                </el-table-column>
                <el-table-column label="手续费率" prop="feeRate" width="100">
                  <template #default="{ row: rental }">
                    {{ rental.feeRate.toFixed(2) }}%
                  </template>
                </el-table-column>
                <el-table-column label="总租金" prop="totalRent" width="120">
                  <template #default="{ row: rental }">
                    ¥{{ rental.totalRent.toFixed(2) }}
                  </template>
                </el-table-column>
                <el-table-column label="扣除手续费后收益" prop="netProfit" width="150">
                  <template #default="{ row: rental }">
                    ¥{{ rental.netProfit.toFixed(2) }}
                  </template>
                </el-table-column>
                <el-table-column label="操作" width="100">
                  <template #default="{ row: rental, $index }">
                    <el-button
                      type="danger"
                      size="small"
                      @click="deleteRentalRecord(row, $index)"
                    >
                      删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
              <el-pagination
                v-if="row.rentalRecords.length > 10"
                style="margin-top: 10px; margin-left: 50px;"
                :current-page="rentalPagination[row.id]?.currentPage || 1"
                :page-size="10"
                layout="prev, pager, next"
                :total="row.rentalRecords.length"
                @current-change="(page) => handleRentalPageChange(row, page)"
              />
            </div>
          </template>
        </el-table-column>
        <el-table-column label="饰品名称" prop="name" width="200" />
        <el-table-column label="购入价格" prop="purchasePrice" width="120">
          <template #default="{ row }">
            ¥{{ row.purchasePrice.toFixed(2) }}
          </template>
        </el-table-column>
        <el-table-column label="购入时间" prop="purchaseDate" width="150">
          <template #default="{ row }">
            {{ formatDate(row.purchaseDate) }}
          </template>
        </el-table-column>
        <el-table-column label="总租金" prop="totalRentalIncome" width="120">
          <template #default="{ row }">
            ¥{{ calculateTotalRentalIncome(row).toFixed(2) }}
          </template>
        </el-table-column>
        <el-table-column label="收益率" prop="profitRate" width="100">
          <template #default="{ row }">
            {{ calculateProfitRate(row).toFixed(2) }}%
          </template>
        </el-table-column>
        <el-table-column label="年化收益率" prop="annualizedRate" width="120">
          <template #default="{ row }">
            {{ calculateAnnualizedRate(row).toFixed(2) }}%
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template #default="{ row }">
            <el-button type="primary" size="small" @click="showAddRentalDialog(row)">
              添加租赁记录
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 底部统计 -->
    <div class="footer-stats">
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

    <!-- 添加饰品对话框 -->
    <el-dialog
      v-model="addItemDialogVisible"
      title="添加饰品"
      width="500px"
    >
      <el-form :model="newItem" label-width="100px">
        <el-form-item label="饰品名称">
          <el-input v-model="newItem.name" placeholder="请输入饰品名称" />
        </el-form-item>
        <el-form-item label="购入价格">
          <el-input
            v-model.number="newItem.purchasePrice"
            placeholder="请输入购入价格"
            type="number"
            step="0.01"
          >
            <template #prepend>¥</template>
          </el-input>
        </el-form-item>
        <el-form-item label="购入日期">
          <el-date-picker
            v-model="newItem.purchaseDate"
            type="date"
            placeholder="选择购入日期"
            style="width: 100%"
            value-format="YYYY-MM-DD"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="addItemDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addItem">确定</el-button>
      </template>
    </el-dialog>

    <!-- 添加租赁记录对话框 -->
    <el-dialog
      v-model="addRentalDialogVisible"
      title="添加租赁记录"
      width="500px"
    >
      <el-form :model="newRental" label-width="100px">
        <el-form-item label="日租金">
          <el-input
            v-model.number="newRental.dailyRent"
            placeholder="请输入日租金"
            type="number"
            step="0.01"
          >
            <template #prepend>¥</template>
          </el-input>
        </el-form-item>
        <el-form-item label="手续费率">
          <el-input
            v-model.number="newRental.feeRate"
            placeholder="请输入手续费率"
            type="number"
            step="0.01"
          >
            <template #append>%</template>
          </el-input>
        </el-form-item>
        <el-form-item label="出租时间段">
          <el-date-picker
            v-model="rentalDateRange"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            style="width: 100%"
            value-format="YYYY-MM-DD"
            @change="calculateRentalDays"
          />
        </el-form-item>
        <el-form-item v-if="rentalDays > 0" label="出租天数">
          <el-tag type="success">{{ rentalDays }} 天</el-tag>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="addRentalDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addRentalRecord">确定</el-button>
      </template>
    </el-dialog>

    <!-- 隐藏的文件输入 -->
    <input
      ref="fileInput"
      type="file"
      accept=".json"
      style="display: none"
      @change="handleFileImport"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'

// 饰品列表
const items = ref([])

// 添加饰品对话框
const addItemDialogVisible = ref(false)
const newItem = ref({
  name: '',
  purchasePrice: 0,
  purchaseDate: ''
})

// 添加租赁记录对话框
const addRentalDialogVisible = ref(false)
const currentItem = ref(null)
const newRental = ref({
  dailyRent: 0,
  feeRate: 0
})
const rentalDateRange = ref([])
const rentalDays = ref(0)

// 租赁记录分页
const rentalPagination = ref({})

// ID生成器
let nextItemId = 1
let nextRentalId = 1

// 显示添加饰品对话框
const showAddItemDialog = () => {
  newItem.value = {
    name: '',
    purchasePrice: 0,
    purchaseDate: ''
  }
  addItemDialogVisible.value = true
}

// 添加饰品
const addItem = () => {
  if (!newItem.value.name) {
    ElMessage.warning('请输入饰品名称')
    return
  }
  if (newItem.value.purchasePrice <= 0) {
    ElMessage.warning('请输入有效的购入价格')
    return
  }
  if (!newItem.value.purchaseDate) {
    ElMessage.warning('请选择购入日期')
    return
  }

  items.value.push({
    id: nextItemId++,
    name: newItem.value.name,
    purchasePrice: Number(newItem.value.purchasePrice),
    purchaseDate: newItem.value.purchaseDate,
    rentalRecords: []
  })

  addItemDialogVisible.value = false
  ElMessage.success('饰品添加成功')
}

// 显示添加租赁记录对话框
const showAddRentalDialog = (item) => {
  currentItem.value = item
  newRental.value = {
    dailyRent: 0,
    feeRate: 0
  }
  rentalDateRange.value = []
  rentalDays.value = 0
  addRentalDialogVisible.value = true
}

// 计算租赁天数
const calculateRentalDays = () => {
  if (rentalDateRange.value && rentalDateRange.value.length === 2) {
    const start = new Date(rentalDateRange.value[0])
    const end = new Date(rentalDateRange.value[1])
    const diffTime = Math.abs(end - start)
    rentalDays.value = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  } else {
    rentalDays.value = 0
  }
}

// 添加租赁记录
const addRentalRecord = () => {
  if (newRental.value.dailyRent <= 0) {
    ElMessage.warning('请输入有效的日租金')
    return
  }
  if (newRental.value.feeRate < 0 || newRental.value.feeRate > 100) {
    ElMessage.warning('请输入有效的手续费率 (0-100)')
    return
  }
  if (!rentalDateRange.value || rentalDateRange.value.length !== 2) {
    ElMessage.warning('请选择出租时间段')
    return
  }
  if (rentalDays.value < 8) {
    ElMessage.warning('根据CS2交易规则，出租时间不能少于8天')
    return
  }

  const totalRent = newRental.value.dailyRent * rentalDays.value
  const fee = totalRent * (newRental.value.feeRate / 100)
  const netProfit = totalRent - fee

  currentItem.value.rentalRecords.push({
    id: nextRentalId++,
    dailyRent: Number(newRental.value.dailyRent),
    feeRate: Number(newRental.value.feeRate),
    startDate: rentalDateRange.value[0],
    endDate: rentalDateRange.value[1],
    days: rentalDays.value,
    totalRent: totalRent,
    netProfit: netProfit
  })

  // 初始化该饰品的分页
  if (!rentalPagination.value[currentItem.value.id]) {
    rentalPagination.value[currentItem.value.id] = { currentPage: 1 }
  }

  addRentalDialogVisible.value = false
  ElMessage.success('租赁记录添加成功')
}

// 删除租赁记录
const deleteRentalRecord = (item, index) => {
  const pagination = rentalPagination.value[item.id] || { currentPage: 1 }
  const actualIndex = (pagination.currentPage - 1) * 10 + index
  item.rentalRecords.splice(actualIndex, 1)
  
  // 如果当前页没有数据了，回到上一页
  if (item.rentalRecords.length > 0) {
    const maxPage = Math.ceil(item.rentalRecords.length / 10)
    if (pagination.currentPage > maxPage) {
      pagination.currentPage = maxPage
    }
  }
  
  ElMessage.success('租赁记录已删除')
}

// 获取当前页的租赁记录
const getRentalRecordsPage = (item) => {
  const pagination = rentalPagination.value[item.id] || { currentPage: 1 }
  const start = (pagination.currentPage - 1) * 10
  const end = start + 10
  return item.rentalRecords.slice(start, end)
}

// 处理租赁记录分页变化
const handleRentalPageChange = (item, page) => {
  if (!rentalPagination.value[item.id]) {
    rentalPagination.value[item.id] = {}
  }
  rentalPagination.value[item.id].currentPage = page
}

// 处理展开变化
const handleExpandChange = (row, expandedRows) => {
  // 初始化分页
  if (!rentalPagination.value[row.id]) {
    rentalPagination.value[row.id] = { currentPage: 1 }
  }
}

// 计算总租金收入
const calculateTotalRentalIncome = (item) => {
  return item.rentalRecords.reduce((sum, rental) => sum + rental.netProfit, 0)
}

// 计算收益率
const calculateProfitRate = (item) => {
  const totalIncome = calculateTotalRentalIncome(item)
  if (item.purchasePrice === 0) return 0
  return (totalIncome / item.purchasePrice) * 100
}

// 计算年化收益率
const calculateAnnualizedRate = (item) => {
  const totalIncome = calculateTotalRentalIncome(item)
  const purchaseDate = new Date(item.purchaseDate)
  const today = new Date()
  const diffTime = Math.abs(today - purchaseDate)
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24))
  
  if (diffDays === 0 || item.purchasePrice === 0) return 0
  
  const profitRate = (totalIncome / item.purchasePrice)
  const annualizedRate = (profitRate / diffDays) * 365 * 100
  
  return annualizedRate
}

// 总购入价
const totalPurchasePrice = computed(() => {
  return items.value.reduce((sum, item) => sum + item.purchasePrice, 0)
})

// 总租金
const totalRentalIncome = computed(() => {
  return items.value.reduce((sum, item) => sum + calculateTotalRentalIncome(item), 0)
})

// 总收益率
const totalProfitRate = computed(() => {
  if (totalPurchasePrice.value === 0) return 0
  return (totalRentalIncome.value / totalPurchasePrice.value) * 100
})

// 总年化收益率
const totalAnnualizedRate = computed(() => {
  if (items.value.length === 0) return 0
  
  let weightedSum = 0
  let totalWeight = 0
  
  items.value.forEach(item => {
    const weight = item.purchasePrice
    const annualizedRate = calculateAnnualizedRate(item)
    weightedSum += annualizedRate * weight
    totalWeight += weight
  })
  
  if (totalWeight === 0) return 0
  return weightedSum / totalWeight
})

// 格式化日期
const formatDate = (dateStr) => {
  if (!dateStr) return ''
  return dateStr
}

// 导出数据
const exportData = () => {
  const data = {
    items: items.value,
    exportDate: new Date().toISOString()
  }
  
  const dataStr = JSON.stringify(data, null, 2)
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `cs2_rent_record_${new Date().getTime()}.json`
  link.click()
  URL.revokeObjectURL(url)
  
  ElMessage.success('数据导出成功')
}

// 导入数据
const fileInput = ref(null)

const importData = () => {
  fileInput.value.click()
}

const handleFileImport = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      if (data.items && Array.isArray(data.items)) {
        items.value = data.items
        
        // 重置分页
        rentalPagination.value = {}
        
        // 更新ID生成器
        let maxItemId = 0
        let maxRentalId = 0
        items.value.forEach(item => {
          if (item.id > maxItemId) maxItemId = item.id
          item.rentalRecords.forEach(rental => {
            if (rental.id > maxRentalId) maxRentalId = rental.id
          })
        })
        nextItemId = maxItemId + 1
        nextRentalId = maxRentalId + 1
        
        ElMessage.success('数据导入成功')
      } else {
        ElMessage.error('文件格式不正确')
      }
    } catch (error) {
      ElMessage.error('文件解析失败')
    }
  }
  reader.readAsText(file)
  
  // 重置文件输入
  event.target.value = ''
}
</script>

<style scoped>
.app-container {
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 2px solid #e4e7ed;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 20px;
}

.header-left h1 {
  margin: 0;
  font-size: 28px;
  color: #303133;
}

.header-right {
  display: flex;
  gap: 10px;
}

.content {
  margin-bottom: 30px;
}

.rental-records {
  padding: 20px 0;
  background-color: #f5f7fa;
}

.footer-stats {
  position: sticky;
  bottom: 0;
  background-color: #fff;
  padding: 20px 0;
  box-shadow: 0 -2px 12px 0 rgba(0, 0, 0, 0.1);
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
