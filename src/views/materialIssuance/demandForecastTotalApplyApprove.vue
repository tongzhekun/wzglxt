<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row type="flex" justify="center" style="margin-top: -15px; text-align: left">
        <el-col :span="9">
          <el-form-item label="申报年份：" prop="year">
            <el-date-picker
              style="height: 40px"
              v-model="form.year"
              format="YYYY"
              value-format="YYYY"
              :disabled="true"
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
              :disabled="true"
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
            style="width: 97%; height: 290px; margin-top: 5px"
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
              width="110px"
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
              width="110px"
            />
          </el-table>
        </el-col>
      </el-row>
      <el-row
        type="flex"
        justify="center"
        style="margin-top: 8px"
        v-if="currentType === '1' && buttonShow"
      >
        <el-col :span="18" style="text-align: center">
          <el-form-item label="审核意见：" prop="approval_content">
            <el-input
              v-model="form.approval_content"
              style="width: 94%; height: 40px"
              placeholder="请输入审核意见"
              :rows="2"
              type="textarea"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 8px" v-if="buttonShow">
        <el-col :span="24" style="text-align: center">
          <el-button
            type="warning"
            @click="cancelClick"
            v-if="currentType === '2' && user_id === form.user_id"
            >取消</el-button
          >
          <el-button type="warning" @click="backfirstClick" v-if="currentType === '1'"
            >退回第一步</el-button
          >
          <el-button type="warning" @click="backlastClick" v-if="currentType === '1'"
            >退回上一步</el-button
          >
          <el-button type="danger" @click="denyClick" v-if="currentType === '1'">否决</el-button>
          <el-button type="primary" @click="submitClick" v-if="currentType === '1'">提交</el-button>
          <el-button type="success" @click="traceClick">流程跟踪</el-button>
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
            <el-col :span="24" style="margin-top: 5px; margin-bottom: 20px; text-align: center">
              <el-button type="primary" @click="confirmApprovalClick">提交</el-button>
            </el-col>
          </el-row>
        </el-dialog>
      </div>
      <el-dialog
        v-model="traceDialog"
        title="流程跟踪"
        width="800px"
        style="height: 500px; overflow: auto"
        class="centered-dialog custom-width-dialog"
      >
        <el-scrollbar style="height: 500px">
          <el-timeline>
            <el-timeline-item v-for="(item, index) in traceOptions" :key="index">
              <el-card :style="{ backgroundColor: item.show_type === '0' ? '#b6b6b6' : '#e9f4ff' }">
                <h4>{{
                  item.show_type === '0' ? item.flow_node_name + '(退回记录)' : item.flow_node_name
                }}</h4>
                <div style="margin-top: 5px">
                  <span>{{
                    item.flow_node != '1'
                      ? '审批人：' + item.approval_name
                      : '发起人：' + item.approval_name
                  }}</span
                  >&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                  <span>{{
                    item.flow_node != '1'
                      ? item.approval_time === '0000-00-00 00:00:00'
                        ? '审批时间：'
                        : '审批时间：' + item.approval_time
                      : item.approval_time === '0000-00-00 00:00:00'
                        ? '发起时间：'
                        : '发起时间：' + item.approval_time
                  }}</span>
                </div>
                <div style="margin-top: 5px" v-if="item.flow_node != '1'"
                  >审批意见：{{ item.approval_content }}</div
                >
              </el-card>
            </el-timeline-item>
          </el-timeline>
        </el-scrollbar>
        <div style="text-align: center">
          <el-button @click="traceDialog = false">取消</el-button>
        </div>
      </el-dialog>
    </el-form>
  </div>
</template>

<script>
import {
  wzType,
  submitDemandTotal,
  userMessage,
  searchNextApproval,
  searchDemandApplyTotal,
  searchTodo,
  searchMaxFlowNode,
  cancel,
  deny,
  backfirst,
  backlast,
  trace
} from '@/api/login'
import { ElMessageBox } from 'element-plus'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
import { color } from 'echarts'
export default {
  name: 'DemandForecastTotalApplyApprove',
  data() {
    return {
      traceDialog: false,
      buttonShow: false,
      currentType: '1', //1是本人审批，2是别人审批自己看审批过的已办，3是已完成
      busi_id: '',
      flow_no: '',
      flow_node: '',
      flow_title: '',
      flow_node_name: '',
      flow_node_max: '',
      material_name: '',
      materialNameShow: true,
      uploadArray: [],
      loading: false,
      user_id: '',
      inst_code: '',
      inst_name: '',
      user_name: '',
      telephone: '',
      dialogVisible: false,
      dialogVisibleApproval: false,
      dialogVisibleMessage: false,
      approveOptions: [],
      traceOptions: [],
      form: {
        approval_content: '',
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
        approval_content: [{ required: true, message: '请输入审批意见', trigger: 'blur' }]
      }
    }
  },
  async created() {
    const userStore = useUserStore()
    const loginInfo = userStore.getUserInfo
    this.user_id = loginInfo.userId
    this.busi_id = this.$route.query.busi_id
    const responseDemand = await searchDemandApplyTotal({ busi_id: this.busi_id })
    this.form.loanData = responseDemand.data.data
    this.form.user_id = responseDemand.data.data[0].user_id
    this.form.year = responseDemand.data.data[0].year
    const array = []
    array.push(responseDemand.data.data[0].time_start)
    array.push(responseDemand.data.data[0].time_end)
    this.form.dateRange = array
    //发起人信息
    const response = await userMessage({ userId: this.form.user_id })
    this.form.user_name = response.data.data[0].employee_name
    this.form.inst_code = response.data.data[0].inst_code
    this.form.inst_name = response.data.data[0].inst_name
    this.form.telephone = response.data.data[0].telephone
    //处理人信息
    const response1 = await userMessage({ userId: this.user_id })
    this.user_name = response1.data.data[0].employee_name
    this.inst_code = response1.data.data[0].inst_code
    this.inst_name = response1.data.data[0].inst_name
    this.telephone = response1.data.data[0].telephone
    const responseTodo = await searchTodo({
      user_id: this.user_id,
      page: this.currentPage,
      pageSize: this.pageSize,
      busi_id: this.busi_id,
      approve_status: '1'
    })
    //大于零说明本人处理的，小于0说明不是本人处理的
    if (responseTodo.data.data.length > 0) {
      this.flow_no = responseTodo.data.data[0].flow_no
      this.flow_node = responseTodo.data.data[0].flow_node
      this.flow_title = responseTodo.data.data[0].flow_title
      this.flow_node_name = responseTodo.data.data[0].flow_node_name
      const responseMaxFlowNode = await searchMaxFlowNode({
        flow_no: this.flow_no
      })
      this.flow_node_max = responseMaxFlowNode.data.data
      this.buttonShow = true
    } else {
      const responseTodo1 = await searchTodo({
        page: this.currentPage,
        pageSize: this.pageSize,
        busi_id: this.busi_id,
        approve_status: '1'
      })
      //大于零说明是不是在本人处理流程，但有人处理
      if (responseTodo1.data.data.length > 0) {
        this.flow_no = responseTodo1.data.data[0].flow_no
        this.flow_node = responseTodo1.data.data[0].flow_node
        this.flow_title = responseTodo1.data.data[0].flow_title
        this.flow_node_name = responseTodo1.data.data[0].flow_node_name
        const responseMaxFlowNode = await searchMaxFlowNode({
          flow_no: this.flow_no
        })
        this.flow_node_max = responseMaxFlowNode.data.data
        this.currentType = '2'
        this.buttonShow = true
        //流程结束，看流程状态
      } else {
        const responseTodo2 = await searchTodo({
          page: this.currentPage,
          pageSize: this.pageSize,
          busi_id: this.busi_id
        })
        this.flow_no = responseTodo2.data.data[0].flow_no
        this.flow_title = responseTodo2.data.data[0].flow_title
        const responseMaxFlowNode = await searchMaxFlowNode({
          flow_no: this.flow_no
        })
        this.flow_node_max = responseMaxFlowNode.data.data
        this.flow_node = this.flow_node_max
        this.currentType = '3'
        this.buttonShow = true
      }
    }
  },
  methods: {
    backfirstClick() {
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
          this.$forceUpdate()
          ElMessageBox.confirm('是否退回第一步?', '提示', {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning'
          })
            .then(async () => {
              const payload = {
                busi_id: this.busi_id,
                flow_node: this.flow_node,
                approval_content: this.form.approval_content
              }
              const response = await backfirst(payload) // 调用 upload 函数并传入 payload
              if (response.data.code == 200) {
                this.$message.success(response.data.message)
                this.$router.push('/materialIssuance/todo')
              } else {
                this.$message.error(response.data.message)
              }
            })
            .catch(() => {})
        }
      }, 300)
    },
    backlastClick() {
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
          this.$forceUpdate()
          ElMessageBox.confirm('是否退回上一步?', '提示', {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning'
          })
            .then(async () => {
              const payload = {
                busi_id: this.busi_id,
                flow_node: this.flow_node,
                approval_content: this.form.approval_content
              }
              const response = await backlast(payload) // 调用 upload 函数并传入 payload
              if (response.data.code == 200) {
                this.$message.success(response.data.message)
                this.$router.push('/materialIssuance/todo')
              } else {
                this.$message.error(response.data.message)
              }
            })
            .catch(() => {})
        }
      }, 300)
    },
    denyClick() {
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
          this.$forceUpdate()
          ElMessageBox.confirm('是否否决该笔流程?', '提示', {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning'
          })
            .then(async () => {
              const payload = {
                busi_id: this.busi_id,
                flow_node: this.flow_node,
                approval_content: this.form.approval_content
              }
              const response = await deny(payload) // 调用 upload 函数并传入 payload
              if (response.data.code == 200) {
                this.$message.success(response.data.message)
                this.$router.push('/materialIssuance/todo')
              } else {
                this.$message.error(response.data.message)
              }
            })
            .catch(() => {})
        }
      }, 300)
    },
    cancelClick() {
      ElMessageBox.confirm('是否取消该笔流程?', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const payload = { busi_id: this.busi_id }
          const response = await cancel(payload) // 调用 upload 函数并传入 payload
          if (response.data.code == 200) {
            this.$message.success(response.data.message)
            this.$router.push('/materialIssuance/todo')
          } else {
            this.$message.error(response.data.message)
          }
        })
        .catch(() => {})
    },
    async traceClick() {
      const responseResult = await trace({
        busi_id: this.busi_id
      })
      this.traceOptions = responseResult.data.data
      this.traceDialog = true
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
              flow_no: this.flow_no,
              flow_node: this.flow_node,
              flow_title: this.flow_title,
              approval_content: this.form.approval_content,
              flow_node_name: this.flow_node_name,
              approval_name: this.form.user_name,
              loanData: this.form.loanData,
              user_id: this.user_id,
              user_name: this.user_name,
              inst_code: this.inst_code,
              inst_name: this.inst_name,
              dateRange: this.form.dateRange,
              year: this.form.year,
              next_approval_id: this.form.next_approval_id,
              next_approval_name: this.form.next_approval_name,
              busi_id: this.busi_id
            })
            if (responseResult.data.code === 300) {
              this.$message.error(responseResult.data.message)
            } else {
              this.dialogVisibleApproval = false
              this.$message.success(responseResult.data.message)
              this.$router.push('/materialIssuance/todo')
            }
          } catch (error) {
            this.$message.error('审批失败')
          }
        }
      }, 300)
      this.$forceUpdate()
    },
    async submitClick() {
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
          if (Number(this.flow_node) === this.flow_node_max) {
            this.confirmApprovalClick()
          } else {
            const responseResult = await searchNextApproval({
              inst_code: this.inst_code,
              flow_no: this.flow_no,
              flow_node: this.flow_node
            })
            this.approveOptions = responseResult.data.data
            this.dialogVisibleApproval = true
          }
        }
      }, 300)
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  margin-left: 20px !important;
  padding-bottom: 20px;
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
:deep(.el-input.is-disabled .el-input__wrapper) {
  background-color: #dcdcdc;
  box-shadow: 0 0 0 1px var(--el-disabled-border-color) inset;
}
</style>
