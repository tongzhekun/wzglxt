<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px" v-if="showAll">
      <el-row type="flex" justify="center" style="text-align: left">
        <el-col :span="9">
          <el-form-item label="客户经理：" prop="user_name">
            <el-input :disabled="true" v-model="form.user_name" style="width: 85%; height: 30px" />
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="客户姓名：" prop="user_id">
            <el-select
              filterable
              v-model="form.custom_license"
              clearable
              placeholder="请选择客户姓名"
              style="width: 85%; height: 30px"
            >
              <el-option
                v-for="item in custOptions"
                :key="item.custom_license"
                :label="item.custom_name"
                :value="item.custom_license"
              />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center">
        <el-col :span="24" style="text-align: center">
          <el-button type="primary" @click="searchClick">查询</el-button>
        </el-col>
      </el-row>
      <el-row style="margin-top: 10px">
        <el-col :span="24">
          <el-table
            :data="form.loanData"
            border
            v-loading="loading"
            element-loading-text="加载中"
            element-loading-svg-view-box="-10, -10, 50, 50"
            element-loading-background="rgba(122, 122, 122, 0.8)"
            :header-cell-style="{ color: '#212121' }"
            style="width: 97%; height: 330px; margin-top: 5px"
          >
            <el-table-column
              prop="custom_name"
              header-align="center"
              fixed
              align="center"
              label="客户名称"
              width="auto"
            />
            <el-table-column
              prop="done_amount"
              header-align="center"
              label="已发放金额"
              align="center"
              width="100px"
            />
            <el-table-column
              prop="is_consumable"
              header-align="center"
              label="易耗品金额"
              align="center"
              width="100px"
            />
            <el-table-column
              prop="no_consumable"
              header-align="center"
              label="非易耗品金额"
              align="center"
              width="110px"
            />
            <el-table-column
              prop="quota_standard"
              header-align="center"
              label="定额标准"
              align="center"
              width="110px"
            />
            <el-table-column
              prop="remain_standard"
              header-align="center"
              label="剩余定额标准"
              align="center"
              width="110px"
            />
          </el-table>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script>
import {
  searchCustList,
  searchReviewProcessKh,
  userRole,
  treeSc,
  wzType,
  submitDemand,
  userMessage,
  searchNextApproval,
  searchDemand
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'IssuanceProgressKh',
  data() {
    return {
      level: '0', //0是各市场部，1是市场部各客户经理，2是客户经理各客户
      instDisable: false,
      showAll: true,
      roleKeyArray: [],
      total: 0, // 总记录数
      currentPage: 1, // 当前页码
      pageSize: 10, // 每页记录数
      loading: false,
      custOptions: [],
      form: {
        custom_license: '',
        instCode: '',
        procurement_time: '',
        procurement_time1: '',
        loanData: [],
        user_name: '',
        user_id: ''
      },
      formRules: {
        // procurement_time: [{ required: true, message: '请选择年份', trigger: 'blur' }]
      }
    }
  },
  async created() {
    this.form.procurement_time1 = this.$route.query.procurement_time1
    this.form.procurement_time = this.$route.query.procurement_time
    this.form.user_id = this.$route.query.user_id
    this.form.user_name = this.$route.query.user_name
    this.form.instCode = this.$route.query.inst_code
    const response1 = await searchCustList({
      custom_name: '',
      user_id: this.form.user_id
    })
    this.custOptions = response1.data.data
    this.searchClick()
  },
  methods: {
    procurementChange() {
      this.form.procurement_time = this.form.procurement_time1.getFullYear()
    },
    async searchClick() {
      let validatestat = false
      this.$refs['formRef'].validate((valid) => {
        if (valid) {
          validatestat = true
        } else {
          return false
        }
      })
      setTimeout(async () => {
        if (validatestat) {
          if (this.level === '0') {
            this.loading = true
            const payload = {
              procurement_time: this.form.procurement_time,
              user_id: this.form.user_id,
              custom_license: this.form.custom_license
            }
            const response = await searchReviewProcessKh(payload) // 调用 upload 函数并传入 payload
            if (response.data.code == 200) {
              this.form.loanData = response.data.data
              this.total = response.data.total
            } else {
              this.$message.error(response.data.data.message)
            }
            this.loading = false
          } else if (this.level === '1') {
            this.loading = true
            const payload = {
              procurement_time: this.form.procurement_time,
              instCode: this.form.instCode,
              user_id: this.form.user_id
            }
            const response = await searchReviewProcessKh(payload) // 调用 upload 函数并传入 payload
            if (response.data.code == 200) {
              this.form.loanData = response.data.data
              this.total = response.data.total
            } else {
              this.$message.error(response.data.data.message)
            }
            this.loading = false
          }
        }
      }, 300)
      this.$forceUpdate()
    },
    calculateColumnWidths(data) {
      const columnWidths = []
      for (let colIndex = 0; colIndex < data[0].length; colIndex++) {
        let maxLength = 0
        data.forEach((row) => {
          const cellValue = row[colIndex] ? row[colIndex].toString() : '' // Ensure it's a string
          maxLength = Math.max(maxLength, cellValue.length)
        })
        columnWidths.push({ wch: maxLength + 7 }) // Adding extra padding for clarity
      }
      return columnWidths
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
:deep(.el-input.is-disabled .el-input__wrapper) {
  background-color: #dcdcdc;
  box-shadow: 0 0 0 1px var(--el-disabled-border-color) inset;
}
:deep(.el-form-item__label) {
  align-items: flex-start;
  box-sizing: border-box;
  color: #000000 !important;
  display: inline-flex;
  flex: 0 0 auto;
  font-size: var(--el-form-label-font-size);
  height: 32px;
  justify-content: flex-end;
  line-height: 32px;
  padding: 0 12px 0 0;
}
</style>
