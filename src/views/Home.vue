<template>
  <div class="home-container">
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
        <el-table-column label="购入时间" prop="purchaseDate" width="120">
          <template #default="{ row }">
            {{ formatDate(row.purchaseDate) }}
          </template>
        </el-table-column>
        <el-table-column label="拥有日数" prop="ownershipDays" width="100">
          <template #default="{ row }">
            {{ calculateOwnershipDays(row) }}天
          </template>
        </el-table-column>
        <el-table-column label="出租天数" prop="totalRentalDays" width="100">
          <template #default="{ row }">
            {{ calculateTotalRentalDays(row) }}天
          </template>
        </el-table-column>
        <el-table-column label="出租时间占比" prop="rentalTimeRatio" width="140">
          <template #default="{ row }">
            {{ calculateRentalTimeRatio(row).toFixed(2) }}%
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
        <el-table-column label="操作" width="200">
          <template #default="{ row, $index }">
            <el-button type="primary" size="small" @click="showAddRentalDialog(row)">
              添加租赁记录
            </el-button>
            <el-button type="danger" size="small" @click="deleteItem($index)">
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
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
          <div style="margin-top: 10px; display: flex; gap: 8px; flex-wrap: wrap;">
            <el-button size="small" @click="applyFeeRate('youyou-discount')">
              悠悠有品
            </el-button>
            <el-button size="small" @click="applyFeeRate('youyou')">
              悠悠有品-减免
            </el-button>
            <el-button size="small" @click="applyFeeRate('igxe')">
              IGXE
            </el-button>
            <el-button size="small" @click="applyFeeRate('igxe-subsidy')">
              IGXE-补贴
            </el-button>
          </div>
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
  </div>
</template>

<script setup>
import { inject } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

// 从App.vue注入共享状态和方法
const items = inject('items')
const rentalPagination = inject('rentalPagination')
const addItemDialogVisible = inject('addItemDialogVisible')
const addRentalDialogVisible = inject('addRentalDialogVisible')
const newItem = inject('newItem')
const newRental = inject('newRental')
const rentalDateRange = inject('rentalDateRange')
const rentalDays = inject('rentalDays')
const currentItem = inject('currentItem')

// 注入方法
const showAddItemDialog = inject('showAddItemDialog')
const addItem = inject('addItem')
const deleteItem = inject('deleteItem')
const showAddRentalDialog = inject('showAddRentalDialog')
const applyFeeRate = inject('applyFeeRate')
const calculateRentalDays = inject('calculateRentalDays')
const addRentalRecord = inject('addRentalRecord')
const deleteRentalRecord = inject('deleteRentalRecord')
const getRentalRecordsPage = inject('getRentalRecordsPage')
const handleRentalPageChange = inject('handleRentalPageChange')
const handleExpandChange = inject('handleExpandChange')
const calculateOwnershipDays = inject('calculateOwnershipDays')
const calculateTotalRentalDays = inject('calculateTotalRentalDays')
const calculateRentalTimeRatio = inject('calculateRentalTimeRatio')
const calculateTotalRentalIncome = inject('calculateTotalRentalIncome')
const calculateProfitRate = inject('calculateProfitRate')
const calculateAnnualizedRate = inject('calculateAnnualizedRate')
const formatDate = inject('formatDate')
</script>

<style scoped>
.home-container {
  padding: 20px;
  padding-top: 110px; /* 为固定的导航栏留出空间 */
  padding-bottom: 120px; /* 为底部统计留出空间 */
}

.content {
  margin-bottom: 30px;
}

.rental-records {
  padding: 20px 0;
  background-color: #f5f7fa;
}
</style>
