<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row type="flex" justify="center" style="text-align: left">
        <el-col :span="9">
          <el-form-item label="事项名称：" prop="flow_title">
            <el-input style="width: 85%; height: 35px" v-model="form.flow_title" />
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="申请人：" prop="apply_name">
            <el-input style="width: 85%; height: 35px" v-model="form.apply_name" />
          </el-form-item>
        </el-col>
        <el-col :span="2">
          <el-button type="primary" @click="searchClick">查询</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-table
            :data="form.loanData"
            :header-cell-style="{ color: '#212121' }"
            @row-click="rowClick"
            :row-class-name="tableRowClassName"
            style="width: 97%; height: 350px; margin-top: 5px"
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
    </el-form>
  </div>
</template>

<script>
import { searchDone, searchDemand, searchWzApply, searchDemandApplyTotal } from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'Done',
  data() {
    return {
      userId: '',
      total: 0, // 总记录数
      currentPage: 1, // 当前页码
      pageSize: 10, // 每页记录数
      form: {
        loanData: [],
        flow_title: '',
        apply_name: ''
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
      const responseDone = await searchDone({
        user_id: this.userId,
        page: this.currentPage,
        pageSize: this.pageSize,
        flow_title: this.form.flow_title,
        apply_name: this.form.apply_name
      })
      this.total = responseDone.data.total
      this.form.loanData = []
      // 创建一个数组来存储所有的异步操作Promise
      // const promises = responseDone.data.data.map(async (item) => {
      //   // 1和2是需求预估
      //   if (item.flow_no == '1' || item.flow_no == '2') {
      //     const responseDemand = await searchDemand({ busi_id: item.busi_id })
      //     item.apply_id = responseDemand.data.data[0].user_id
      //     item.apply_name = responseDemand.data.data[0].user_name
      //     item.time = responseDemand.data.data[0].time
      //     // 1和2是需求预估
      //     if (item.flow_no === '1' || item.flow_no === '2') {
      //       item.url = '/materialIssuance/demandForecastApprove'
      //     }
      //     return item
      //   } else if (item.flow_no == '3' || item.flow_no == '4') {
      //     const responseDemand = await searchWzApply({ busi_id: item.busi_id })
      //     item.apply_id = responseDemand.data.data[0].user_id
      //     item.apply_name = responseDemand.data.data[0].user_name
      //     item.time = responseDemand.data.data[0].time
      //     item.url = '/materialIssuance/reviewApprove'
      //     return item
      //   } else if (item.flow_no == '5') {
      //     const responseDemand = await searchDemandApplyTotal({ busi_id: item.busi_id })
      //     item.apply_id = responseDemand.data.data[0].user_id
      //     item.apply_name = responseDemand.data.data[0].user_name
      //     item.time = responseDemand.data.data[0].time
      //     item.url = '/materialIssuance/demandForecastTotalApplyApprove'
      //     return item
      //   }
      // })
      // const processedItems = await Promise.all(promises)
      // this.form.loanData = processedItems
      this.form.loanData = responseDone.data.data
      this.form.loanData = this.form.loanData.filter((item) =>
        item.apply_name.includes(this.form.apply_name)
      )
      // 对this.form.loanData进行排序
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
