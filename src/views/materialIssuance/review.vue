<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row type="flex" justify="center" style="margin-top: -15px; text-align: left">
        <el-col :span="9">
          <el-form-item label="客户名称：" prop="custom_name">
            <el-autocomplete
              @change="inputName"
              v-model="form.custom_name"
              :fetch-suggestions="querySearchAsync"
              placeholder="请输入客户名称"
              style="width: 85%; height: 40px"
              @select="handleSelect"
            />
          </el-form-item>
          <el-form-item label="负责人：" prop="operator_name">
            <el-input
              class="inputClass"
              v-model="form.operator_name"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="许可证号：" prop="custom_license">
            <el-input
              class="inputClass"
              v-model="form.custom_license"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="经营者电话：" prop="operator_telephone">
            <el-input
              class="inputClass"
              v-model="form.operator_telephone"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center">
        <el-col :span="18" style="text-align: center">
          <el-form-item label="经营地址：" prop="custom_address">
            <el-input
              class="inputClass"
              v-model="form.custom_address"
              :disabled="true"
              style="width: 94%; height: 40px"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="text-align: left">
        <el-col :span="9">
          <el-form-item label="客户经理：" prop="employee_name">
            <el-input
              class="inputClass"
              v-model="form.employee_name"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="申请物料名称：" prop="material_name">
            <el-autocomplete
              v-model="form.material_name"
              @change="inputName1"
              :fetch-suggestions="querySearchAsync1"
              placeholder="请输入申请物料名称"
              style="width: 85%; height: 40px"
              @select="handleSelect1"
            />
          </el-form-item>
          <el-form-item label="物料数量：" prop="num">
            <el-input
              v-model="form.num"
              @input="handleInput"
              style="width: 85%; height: 40px"
              placeholder="请选择物料数量"
            />
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="终端层级：" prop="terminal_level">
            <el-input
              class="inputClass"
              v-model="form.terminal_level"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="是否易耗品：" prop="consumable">
            <el-radio-group v-model="form.consumable">
              <el-radio value="1" :disabled="true">是</el-radio>
              <el-radio value="0" :disabled="true">否</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            label="延期申请流程编号："
            class="item-label"
            v-if="delayType == '1'"
            prop="delay_busi_id"
          >
            <el-input
              v-model="form.delay_busi_id"
              style="width: 85%; height: 40px"
              placeholder="请输入延期申请流程编号"
            />
            <el-link @click="reviewDelayClick" style="color: #29d; text-decoration: underline"
              >延期申请链接</el-link
            >
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="text-align: left">
        <el-col :span="9">
          <el-form-item label="发起人编号：" prop="user_id">
            <el-input
              class="inputClass"
              v-model="form.user_id"
              :disabled="true"
              style="width: 85%; height: 40px"
              placeholder="请输入发起人编号"
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
          <el-form-item label="发起人所属机构：" prop="inst_name">
            <el-input
              v-model="form.inst_name"
              :disabled="true"
              style="width: 85%; height: 40px"
              placeholder="请输入发起人所属机构"
            />
          </el-form-item>
          <el-form-item label="联系电话：" prop="telephone">
            <el-input
              v-model="form.telephone"
              :disabled="true"
              style="width: 85%; height: 40px"
              placeholder="请输入联系电话"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 8px">
        <el-col :span="24" style="text-align: center">
          <el-button type="primary" @click="submitClick">提交流程</el-button>
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
  searchDepartWz,
  searchCustList,
  userMessage,
  searchNextApproval,
  submitWzApply,
  searchWzApply,
  isDelayMaterial,
  searchWzDelayApply,
  searchFlowStatus
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'Review',
  data() {
    return {
      delayType: '0', //该物料是否是延期物料
      customArray: [],
      wzArray: [],
      busi_id: '',
      flow_no: '3',
      material_name: '',
      materialNameShow: true,
      userId: '',
      dialogVisibleApproval: false,
      approveOptions: [],
      form: {
        inst_code: '',
        inst_name: '',
        user_id: '',
        user_name: '',
        telephone: '',
        custom_name: '',
        operator_name: '',
        custom_license: '',
        operator_telephone: '',
        custom_address: '',
        employee_name: '',
        material_code: '',
        material_name: '',
        terminal_level: '',
        consumable: '',
        num: '',
        next_approval_id: '',
        next_approval_name: '',
        delay_busi_id: ''
      },
      formRules: {
        next_approval_id: [{ required: true, message: '请选择审批人', trigger: 'blur' }],
        num: [{ required: true, message: '请输入物料数量', trigger: 'blur' }],
        custom_name: [{ required: true, message: '请输入客户名称', trigger: 'blur' }],
        material_name: [{ required: true, message: '请输入申请物料名称', trigger: 'blur' }],
        custom_license: [{ required: true, message: '请选择客户名称后返显', trigger: 'blur' }],
        consumable: [{ required: true, message: '请选择物料种类后返显', trigger: 'blur' }],
        delay_busi_id: [{ required: true, message: '请输入延期申请流程编号', trigger: 'blur' }]
      }
    }
  },
  async created() {
    this.busi_id = this.$route.query.busi_id
    if (this.busi_id != undefined && this.busi_id != '') {
      const responseDemand = await searchWzApply({ busi_id: this.busi_id })
      this.form.custom_name = responseDemand.data.data[0].custom_name
      this.form.operator_name = responseDemand.data.data[0].operator_name
      this.form.custom_license = responseDemand.data.data[0].custom_license
      this.form.operator_telephone = responseDemand.data.data[0].operator_telephone
      this.form.custom_address = responseDemand.data.data[0].custom_address
      this.form.employee_name = responseDemand.data.data[0].employee_name
      this.form.material_code = responseDemand.data.data[0].material_code
      this.form.material_name = responseDemand.data.data[0].material_name
      this.form.terminal_level = responseDemand.data.data[0].terminal_level
      this.form.consumable = responseDemand.data.data[0].consumable
      this.form.num = responseDemand.data.data[0].num
    } else {
      this.busi_id = ''
    }
    const userStore = useUserStore()
    const loginInfo = userStore.getUserInfo
    this.userId = loginInfo.userId
    this.form.user_id = loginInfo.userId
    const response = await userMessage({ userId: this.userId })
    this.form.user_name = response.data.data[0].employee_name
    this.form.inst_code = response.data.data[0].inst_code
    this.form.inst_name = response.data.data[0].inst_name
    this.form.telephone = response.data.data[0].telephone
    if (this.form.inst_code.substring(5, 6) === '1') {
      this.flow_no = '4'
    } else {
      this.flow_no = '3'
    }
  },
  methods: {
    reviewDelayClick() {
      this.$router.push('/materialIssuance/reviewDelay')
    },
    async querySearchAsync(queryString, cb) {
      const responseResult = await searchCustList({
        custom_name: queryString,
        user_id: this.form.user_id
      })
      this.customArray = []
      responseResult.data.data.forEach((item) => {
        this.customArray.push({
          value: item.custom_name,
          custom_name: item.custom_name,
          operator_name: item.operator_name,
          custom_license: item.custom_license,
          operator_telephone: item.operator_telephone,
          custom_address: item.custom_address,
          employee_name: item.employee_name,
          terminal_level: item.terminal_level
        })
      })
      cb(this.customArray)
    },
    handleSelect(item) {
      this.form.custom_name = item.custom_name
      this.form.operator_name = item.operator_name
      this.form.custom_license = item.custom_license
      this.form.operator_telephone = item.operator_telephone
      this.form.custom_address = item.custom_address
      this.form.employee_name = item.employee_name
      this.form.terminal_level = item.terminal_level
      this.$forceUpdate()
    },
    inputName() {
      this.form.operator_name = ''
      this.form.custom_license = ''
      this.form.operator_telephone = ''
      this.form.custom_address = ''
      this.form.employee_name = ''
      this.form.terminal_level = ''
    },
    inputName1() {
      this.form.consumable = ''
    },
    handleInput() {
      let value = this.form.num
      if (value !== null && value !== '') {
        // 使用正则表达式匹配，只保留数字部分
        value = value.toString().replace(/\D/g, '')
        this.form.num = value
      } else {
        this.form.num = null
      }
    },
    async querySearchAsync1(queryString, cb) {
      const responseResult = await searchDepartWz({
        material_name: queryString,
        inst_code: this.form.inst_code
      })
      this.wzArray = []
      responseResult.data.data.forEach((item) => {
        this.wzArray.push({
          value: item.material_name,
          material_name: item.material_name,
          material_code: item.material_code,
          consumable: item.consumable
        })
      })
      cb(this.wzArray)
    },
    async handleSelect1(item) {
      this.form.material_name = item.material_name
      this.form.consumable = item.consumable
      this.form.material_code = item.material_code
      this.form.delay_busi_id = ''
      this.delayType = '0'
      const responseResult = await isDelayMaterial({
        inst_code: this.form.inst_code,
        material_code: this.form.material_code
      })
      this.delayType = responseResult.data.data[0].delayType
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
      try {
        const responseResult = await submitWzApply({
          busi_id: this.busi_id,
          flow_no: this.flow_no,
          flow_node: '1',
          flow_title: this.form.custom_name + '的' + this.form.material_name + '的物料申请',
          approval_content: '',
          flow_node_name: '客户经理发起',
          approval_name: this.form.user_name,
          user_id: this.userId,
          user_name: this.form.user_name,
          inst_code: this.form.inst_code,
          inst_name: this.form.inst_name,
          telephone: this.form.telephone,
          next_approval_id: this.form.next_approval_id,
          next_approval_name: this.form.next_approval_name,
          custom_name: this.form.custom_name,
          operator_name: this.form.operator_name,
          custom_license: this.form.custom_license,
          operator_telephone: this.form.operator_telephone,
          custom_address: this.form.custom_address,
          employee_name: this.form.employee_name,
          material_code: this.form.material_code,
          material_name: this.form.material_name,
          terminal_level: this.form.terminal_level,
          consumable: this.form.consumable,
          num: this.form.num
        })
        this.dialogVisibleApproval = false
        this.$message.success(responseResult.data.message)
        this.$router.push('/materialIssuance/todo')
      } catch (error) {
        this.$message.error('流程发起失败')
      }
      this.$forceUpdate()
    },
    async submitClick() {
      let validatestat = false
      try {
        // 使用await等待验证完成，返回的结果就是验证是否通过（true或false）
        validatestat = await this.$refs['formRef'].validate()
      } catch (error) {
        // 这里可以处理验证过程中出现的异常，比如显示错误提示等
        console.log(error)
      }
      setTimeout(async () => {
        if (validatestat) {
          if (this.form.inst_code.length != 7) {
            this.$message.warning('只有市场部的客户经理能发起物料申请')
          } else {
            if (this.form.delay_busi_id == '') {
              const responseResult = await searchNextApproval({
                inst_code: this.form.inst_code,
                flow_no: this.flow_no,
                flow_node: '1'
              })
              this.approveOptions = responseResult.data.data
            } else {
              //判断流程状态，再判断是否客户名称、物料名称、物料数量一致
              const responseFlowResult = await searchFlowStatus({
                busi_id: this.form.delay_busi_id,
                flow_status: '5'
              })
              if (responseFlowResult.data.data.length == 0) {
                this.$message.warning('延期申请流程编号有误或流程未结束')
              } else {
                const responseDelay = await searchWzDelayApply({ busi_id: this.form.delay_busi_id })
                if (
                  this.form.custom_license == responseDelay.data.data[0].custom_license &&
                  this.form.material_code == responseDelay.data.data[0].material_code &&
                  this.form.num == responseDelay.data.data[0].num
                ) {
                  const responseResult = await searchNextApproval({
                    inst_code: this.form.inst_code,
                    flow_no: this.flow_no,
                    flow_node: '1'
                  })
                  this.approveOptions = responseResult.data.data
                } else {
                  this.$message.warning('申请与延期申请的信息不一致（客户姓名、物料名称和数量）')
                }
              }
            }
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
:deep(.el-input__wrapper) {
  height: 40px;
  align-items: center;
  background-color: var(--el-input-bg-color, var(--el-fill-color-blank));
  background-image: none;
  border-radius: var(--el-input-border-radius, var(--el-border-radius-base));
  box-shadow: 0 0 0 1px var(--el-input-border-color, var(--el-border-color)) inset;
  cursor: text;
  display: inline-flex;
  flex-grow: 1;
  justify-content: center;
  padding: 1px 11px;
  transform: translateZ(0);
  transition: var(--el-transition-box-shadow);
}
:deep(.item-label .el-form-item__label) {
  line-height: 22px !important;
}
</style>
