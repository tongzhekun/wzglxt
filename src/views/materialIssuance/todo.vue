<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row>
        <el-col :span="24">
          <el-table
            :data="form.loanData"
            :header-cell-style="{ color: '#212121' }"
            @row-click="rowClick"
            :row-class-name="tableRowClassName"
            style="width: 97%; height: 400px; margin-top: 5px"
          >
            <el-table-column
              prop="flow_title"
              header-align="center"
              fixed
              align="center"
              label="事项名称"
              width="auto"
            />
            <el-table-column
              prop="flow_status"
              align="center"
              header-align="center"
              label="状态"
              width="110px"
            >
              <template #default="scope">
                <span>{{
                  scope.row.flow_status === '1'
                    ? '待审核'
                    : scope.row.flow_status === '2'
                      ? '已退回'
                      : scope.row.flow_status === '3'
                        ? '已否决'
                        : scope.row.flow_status === '4'
                          ? '已取消'
                          : '已完成'
                }}</span>
              </template>
            </el-table-column>
            <el-table-column
              prop="apply_name"
              header-align="center"
              label="申请人"
              align="center"
              width="180px"
            />
            <el-table-column
              prop="time"
              header-align="center"
              align="center"
              label="申请时间"
              width="250px"
            />
          </el-table>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24" style="margin-top: 5px; text-align: center">
          <el-pagination
            background
            size="small"
            layout="total, prev, pager, next"
            :total="total"
            v-model:current-page="currentPage"
            :page-size="pageSize"
            @current-change="fetchData"
          /> </el-col
      ></el-row>
      <el-dialog
        v-model="dialogVisibleMessage"
        title="物料到期消息提示"
        width="1100px"
        style="text-align: center"
        class="centered-dialog custom-width-dialog"
      >
        <div style="margin-top: 20px">
          <el-row>
            <el-col :span="24">
              <el-table
                :data="loanDataMessage"
                border
                v-loading="loading"
                element-loading-text="加载中"
                element-loading-svg-view-box="-10, -10, 50, 50"
                element-loading-background="rgba(122, 122, 122, 0.8)"
                :header-cell-style="{ color: '#212121' }"
                style="width: 97%; height: 325px; margin-top: 5px"
              >
                <el-table-column
                  prop="material_name"
                  header-align="center"
                  align="center"
                  label="物料名称"
                  width="auto"
                />
                <el-table-column
                  prop="creation_time"
                  header-align="center"
                  label="入库时间"
                  align="center"
                  width="120px"
                />
                <el-table-column
                  prop="end_time"
                  header-align="center"
                  label="发放结束时间"
                  align="center"
                  width="120px"
                />
                <el-table-column
                  prop="delay_time"
                  header-align="center"
                  align="center"
                  label="延期时间"
                  width="120px"
                />
                <el-table-column
                  prop="inventory_quantity"
                  align="center"
                  header-align="center"
                  label="物料数量"
                  width="100px"
                />
                <el-table-column
                  prop="inst_name"
                  align="center"
                  header-align="center"
                  label="机构名称"
                  width="auto"
                />
              </el-table>
            </el-col>
          </el-row>
        </div>
      </el-dialog>
    </el-form>
  </div>
</template>

<script>
import {
  userMessage,
  userRole,
  searchTodo,
  searchDelayMessage,
  searchDemand,
  searchWzApply,
  searchDemandApplyTotal,
  searchInventoryCheckApply,
  searchQrCodeChangeApply,
  searchWzDelayApply
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'Todo',
  data() {
    return {
      role: '0',
      roleKeyArray: [],
      loading: false,
      loanDataMessage: [],
      dialogVisibleMessage: false,
      userId: '',
      total: 0, // 总记录数
      currentPage: 1, // 当前页码
      pageSize: 10, // 每页记录数
      form: {
        loanData: []
      },
      formRules: {}
    }
  },
  async created() {
    const userStore = useUserStore()
    const loginInfo = userStore.getUserInfo
    this.userId = loginInfo.userId
    this.form.user_id = loginInfo.userId
    this.searchClick()
    const response2 = await userRole({ userId: this.userId })
    const roleKey = response2.data.data
    this.roleKeyArray = []
    if (roleKey.length > 0) {
      roleKey.forEach((item) => {
        this.roleKeyArray.push(item.role_id)
      })
    }
    if (this.roleKeyArray.indexOf('yc001') > -1 || this.roleKeyArray.indexOf('yc002') > -1) {
      this.role = '1'
    }
    this.loanDataMessage = []
    if (this.role === '1') {
      const responseUser = await userMessage({ userId: this.userId })
      const inst_code = responseUser.data.data[0].inst_code
      const response = await searchDelayMessage({ instCode: inst_code }) // 调用 upload 函数并传入 payload
      if (response.data.code == 200) {
        this.loanDataMessage = response.data.data
        if (this.loanDataMessage.length > 0) {
          this.dialogVisibleMessage = true
        }
      } else {
        this.$message.error(response.data.data.message)
      }
    }
  },
  activated() {
    this.searchClick()
    this.$forceUpdate()
  },
  methods: {
    rowClick(row) {
      this.$router.push(row.url + '?busi_id=' + row.busi_id)
    },
    async fetchData() {
      const responseTodo = await searchTodo({
        user_id: this.userId,
        page: this.currentPage,
        pageSize: this.pageSize,
        approve_status: '1'
      })

      this.total = responseTodo.data.total
      this.form.loanData = []
      // 创建一个数组来存储所有的异步操作Promise
      const promises = responseTodo.data.data.map(async (item) => {
        // 1和2是需求预估
        if (item.flow_no == '1' || item.flow_no == '2') {
          const responseDemand = await searchDemand({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialIssuance/demandForecast'
          } else {
            item.url = '/materialIssuance/demandForecastApprove'
          }
          return item
          //物资申请
        } else if (item.flow_no == '3' || item.flow_no == '4') {
          const responseDemand = await searchWzApply({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialIssuance/review'
          } else {
            item.url = '/materialIssuance/reviewApprove'
          }
          return item
        } else if (item.flow_no == '5') {
          const responseDemand = await searchDemandApplyTotal({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialIssuance/demandForecastTotalApply'
          } else {
            item.url = '/materialIssuance/demandForecastTotalApplyApprove'
          }
          return item
        } else if (item.flow_no == '6' || item.flow_no == '7') {
          const responseDemand = await searchInventoryCheckApply({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialSupervision/inventoryCheck'
          } else {
            item.url = '/materialSupervision/inventoryCheckApprove'
          }
          return item
        } else if (item.flow_no == '8' || item.flow_no == '9') {
          const responseDemand = await searchQrCodeChangeApply({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialSupervision/qrCode'
          } else {
            item.url = '/materialSupervision/qrCodeApprove'
          }
          return item
        } else if (item.flow_no == '10' || item.flow_no == '11') {
          const responseDemand = await searchWzDelayApply({ busi_id: item.busi_id })
          item.apply_id = responseDemand.data.data[0].user_id
          item.apply_name = responseDemand.data.data[0].user_name
          item.time = responseDemand.data.data[0].time
          if (item.flow_node === '1') {
            item.url = '/materialIssuance/reviewDelay'
          } else {
            item.url = '/materialIssuance/reviewDelayApprove'
          }
          return item
        }
      })
      // 使用Promise.all等待所有异步操作完成
      const processedItems = await Promise.all(promises)
      // 将处理好的所有项一次性添加到form.loanData数组中
      this.form.loanData = processedItems
      // 对this.form.loanData按照time字段进行排序
      this.form.loanData.sort((a, b) => {
        return new Date(b.time).getTime() - new Date(a.time).getTime()
      })
    },
    async searchClick() {
      this.currentPage = 1
      this.fetchData()
    },
    tableRowClassName({ row, rowIndex }) {
      return 'rowClassName'
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  margin-left: 20px !important;
  margin-right: 20px !important;
}
:deep(.el-input.is-disabled .el-input__inner) {
  color: #000000 !important;
  -webkit-text-fill-color: #000000 !important;
}
table {
  width: 100%;
  margin-bottom: 20px;
  overflow: hidden; /* 防止内容溢出 */
  border: 2px solid #909399; /* 表格整体边框加厚 */
  border-collapse: separate; /* 使用 separate 避免边框重叠 */
  border-spacing: 0; /* 确保单元格间距为 0 */
}

.el-table--fit {
  border-right: 1px solid #909399;
  border-bottom: 0;
  border-bottom: 1px solid #909399;
  border-left: 1px solid #909399;
}

table,
th,
td {
  border: 1px solid #909399; /* 保持统一边框 */
}

.el-table.cell {
  padding: 0 12px;
  overflow: hidden;
  line-height: auto;
  text-overflow: ellipsis;
  white-space: normal;
  box-sizing: border-box;
  overflow-wrap: break-word;
}

/* 表头样式 */
.el-table__header-wrapper th {
  color: #000;
  vertical-align: top; /* 垂直对齐顶部 */
  border-bottom: none; /* 移除 th 自身的下边框 */
}

/* 表头容器样式 */
.el-table__header-wrapper {
  position: relative; /* 设置相对定位 */
  z-index: 1;
  height: 41px !important;
  overflow: hidden;
  background-color: #fff;
}

/* 表头下边框伪元素 */
.el-table__header-wrapper::after {
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 2; /* 确保伪元素覆盖表格内容 */
  height: 1px;
  background-color: #909399; /* 表头下边框颜色 */
  content: '';
}

/* 表体单元格样式 */
.el-table__body tr td {
  height: 12px; /* 单元格高度 */
  padding: 5px !important;
  color: #212121;
  border-top: 1px solid #909399; /* 确保中间行上边框存在 */
  border-right: 1px solid #909399;
  border-bottom: 1px solid #909399; /* 确保中间行下边框存在 */
}

/* 去除第一行单元格的上边框 */
.el-table__body tr:first-child td {
  border-top: none;
}

/* 去除最后单元格的右边框 */
.el-table__body tr td:last-child {
  border-right: none;
}

/* 去除最后一行的下边框 */
.el-table__body tr:last-child td {
  border-bottom: none;
}

/* 默认表格单元格样式 */
.el-table--default.el-table__cell {
  padding: 0; /* 调整内边距 */
}

.custom-form-item::v-deep.el-form-item__label {
  font-size: 18px;
  color: #000 !important;
}

.el-dialog__header {
  height: 54px;
  padding: 0;
  margin-right: 0 !important;
  text-align: center;
  border-bottom: 1px solid var(--el-border-color);
}
// :deep(.el-table.rowClassName) {
//   cursor: pointer !important;
// }
// :deep(.el-table__body tr.rowClassName) {
//   cursor: pointer !important;
// }
:deep(.rowClassName) {
  cursor: pointer !important;
}
.el-pagination {
  --el-pagination-font-size: 14px;
  --el-pagination-bg-color: var(--el-fill-color-blank);
  --el-pagination-text-color: var(--el-text-color-primary);
  --el-pagination-border-radius: 2px;
  --el-pagination-button-color: var(--el-text-color-primary);
  --el-pagination-button-width: 32px;
  --el-pagination-button-height: 32px;
  --el-pagination-button-disabled-color: var(--el-text-color-placeholder);
  --el-pagination-button-disabled-bg-color: var(--el-fill-color-blank);
  --el-pagination-button-bg-color: var(--el-fill-color);
  --el-pagination-hover-color: var(--el-color-primary);
  --el-pagination-font-size-small: 12px;
  --el-pagination-button-width-small: 24px;
  --el-pagination-button-height-small: 24px;
  --el-pagination-button-width-large: 40px;
  --el-pagination-button-height-large: 40px;
  --el-pagination-item-gap: 16px;
  align-items: center;
  color: var(--el-pagination-text-color);
  display: flex;
  font-size: var(--el-pagination-font-size);
  font-weight: normal;
  justify-content: center;
  white-space: nowrap;
}
</style>
