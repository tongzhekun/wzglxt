<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row type="flex" justify="center" style="margin-top: -15px; text-align: left">
        <el-col :span="9">
          <el-form-item label="申报年份：" prop="year">
            <el-date-picker
              style="height: 40px"
              v-model="form.year"
              @change="yearChange"
              format="YYYY"
              value-format="YYYY"
              type="year"
              size="small"
            />
          </el-form-item>
          <el-form-item label="发起人名称：" prop="user_name">
            <el-input
              v-model="form.user_name"
              :disabled="true"
              style="width: 85%; height: 40px"
              placeholder="请输入发起人名称"
            />
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="时间：" prop="dateRange">
            <el-date-picker
              style="height: 40px"
              v-model="form.dateRange"
              type="daterange"
              range-separator="至"
              start-placeholder="开始时间"
              end-placeholder="结束时间"
              size="small"
            />
          </el-form-item>
          <el-form-item label="发起人所属机构：" prop="inst_name">
            <el-input
              v-model="form.inst_name"
              :disabled="true"
              style="height: 40px"
              placeholder="请输入发起人所属机构"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center">
        <el-col :span="24" style="text-align: center">
          <el-button type="primary" @click="searchClick">查询</el-button>
          <el-button type="success" @click="submitClick">提交流程</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24">
          <el-table
            :data="form.loanData"
            border
            v-loading="loading"
            element-loading-text="加载中"
            element-loading-svg-view-box="-10, -10, 50, 50"
            element-loading-background="rgba(122, 122, 122, 0.8)"
            :header-cell-style="{ color: '#212121' }"
            style="width: 97%; height: 310px; margin-top: 5px"
          >
            <el-table-column
              prop="material_name"
              header-align="center"
              fixed
              align="center"
              label="物料名称"
              width="auto"
            />
            <el-table-column
              prop="material_type"
              header-align="center"
              label="物料类型"
              align="center"
              width="130px"
            />
            <el-table-column
              prop="consumable"
              header-align="center"
              label="是否易耗品(1是0否)"
              align="center"
              width="180px"
            />
            <el-table-column
              prop="num"
              header-align="center"
              align="center"
              label="需求数量"
              width="130px"
            />
          </el-table>
        </el-col>
      </el-row>
      <div v-if="dialogVisibleApproval">
        <el-dialog
          v-model="dialogVisibleApproval"
          title="选择审批人"
          width="500px"
          style="height: 230px; overflow: auto"
          class="centered-dialog custom-width-dialog"
        >
          <el-row type="flex" justify="center" style="margin-top: 5px">
            <el-col :span="24">
              <el-form-item label="审批人：" prop="next_approval_id">
                <el-select
                  v-model="form.next_approval_id"
                  clearable
                  @change="nextApprovalIdChange"
                  placeholder="请选择审批人"
                  style="width: 85%; height: 30px"
                >
                  <el-option
                    v-for="item in approveOptions"
                    :key="item.employee_code"
                    :label="item.employee_name + '(' + item.employee_code + ')'"
                    :value="item.employee_code"
                  />
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24" style="margin-top: 5px; text-align: center">
              <el-button type="primary" @click="confirmApprovalClick">提交</el-button>
            </el-col>
          </el-row>
        </el-dialog>
      </div>
    </el-form>
  </div>
</template>

<script>
import {
  submitDemandTotal,
  searchDemandApplyTotal,
  searchDemandMx,
  wzType,
  userMessage,
  searchNextApproval
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'DemandForecastTotalApply',
  data() {
    return {
      busi_id: '',
      flow_no: '5',
      material_name: '',
      materialNameShow: true,
      uploadArray: [],
      wzOptions: [],
      loading: false,
      userId: '',
      dialogVisible: false,
      dialogVisibleApproval: false,
      dialogVisibleMessage: false,
      approveOptions: [],
      form: {
        year: '',
        dateRange: '',
        inst_code: '',
        inst_name: '',
        user_id: '',
        user_name: '',
        telephone: '',
        loanData: [],
        consumable: '',
        material_code: '',
        num: '',
        material_type: '',
        material_name: '',
        next_approval_id: '',
        next_approval_name: ''
      },
      formRules: {
        next_approval_id: [{ required: true, message: '请选择审批人', trigger: 'blur' }],
        dateRange: [{ required: true, message: '请选择时间', trigger: 'blur' }],
        year: [{ required: true, message: '请选择申报年份', trigger: 'blur' }]
      }
    }
  },
  async created() {
    this.busi_id = this.$route.query.busi_id
    if (this.busi_id != undefined && this.busi_id != '') {
      const responseDemand = await searchDemandApplyTotal({ busi_id: this.busi_id })
      this.form.loanData = responseDemand.data.data
      this.form.year = responseDemand.data.data[0].year
      const array = []
      array.push(responseDemand.data.data[0].time_start)
      array.push(responseDemand.data.data[0].time_end)
      this.form.dateRange = array
    } else {
      this.busi_id = ''
    }
    const userStore = useUserStore()
    const loginInfo = userStore.getUserInfo
    this.userId = loginInfo.userId
    this.form.user_id = loginInfo.userId
    const responseWzType = await wzType({})
    this.wzOptions = responseWzType.data.data
    const response = await userMessage({ userId: this.userId })
    this.form.user_name = response.data.data[0].employee_name
    this.form.inst_code = response.data.data[0].inst_code
    this.form.inst_name = response.data.data[0].inst_name
    this.form.telephone = response.data.data[0].telephone
    this.flow_no = '5'
  },
  methods: {
    yearChange() {},
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
          this.loading = true
          const payload = {
            page: this.currentPage,
            pageSize: this.pageSize,
            instCode: this.form.inst_code,
            time: this.form.dateRange,
            user_name: ''
          }
          const response = await searchDemandMx(payload) // 调用 upload 函数并传入 payload
          if (response.data.code == 200) {
            this.form.loanData = response.data.data
            this.total = response.data.total
          } else {
            this.$message.error(response.data.data.message)
          }
          this.loading = false
        }
      }, 300)
      this.$forceUpdate()
    },
    nextApprovalIdChange() {
      this.approveOptions.forEach((item) => {
        if (item.employee_code === this.form.next_approval_id) {
          this.form.next_approval_name = item.employee_name
        }
      })
    },
    async confirmApprovalClick() {
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
          try {
            const responseResult = await submitDemandTotal({
              busi_id: this.busi_id,
              flow_no: this.flow_no,
              flow_node: '1',
              flow_title: this.form.user_name + '的需求预估汇总申请',
              approval_content: '',
              flow_node_name: '客户经理发起',
              approval_name: this.form.user_name,
              loanData: this.form.loanData,
              user_id: this.userId,
              user_name: this.form.user_name,
              inst_code: this.form.inst_code,
              inst_name: this.form.inst_name,
              dateRange: this.form.dateRange,
              year: this.form.year,
              next_approval_id: this.form.next_approval_id,
              next_approval_name: this.form.next_approval_name
            })
            this.dialogVisibleApproval = false
            this.$message.success(responseResult.data.message)
            this.$router.push('/materialIssuance/todo')
          } catch (error) {
            this.$message.error('流程发起失败')
          }
        }
      }, 300)
      this.$forceUpdate()
    },
    async submitClick() {
      if (this.form.loanData === null || this.form.loanData.length === 0) {
        this.$message.warning('请先查询要提交的物料需求表数据')
      } else {
        const responseResult = await searchNextApproval({
          inst_code: this.form.inst_code,
          flow_no: this.flow_no,
          flow_node: '1'
        })
        this.approveOptions = responseResult.data.data
        this.dialogVisibleApproval = true
      }
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
