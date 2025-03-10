<template>
  <div class="container">
    <el-form
      :model="form"
      ref="formRef"
      :rules="formRules"
      label-width="130px"
      v-if="judgeType === '1'"
    >
      <el-row type="flex" justify="center" style="margin-top: -15px; text-align: left">
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
      <el-row type="flex" justify="center" style="margin-top: -23px">
        <el-col :span="24" style="text-align: left">
          <el-button type="primary" @click="addClick">新增</el-button>
          <el-button type="success" @click="importClick">导入</el-button>
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
            <el-table-column
              prop="project"
              align="center"
              header-align="center"
              label="操作"
              width="180px"
            >
              <template #default="scope">
                <el-button round type="primary" size="small" @click.prevent="editClick(scope.row)"
                  >修改
                </el-button>
                <el-button round type="danger" size="small" @click.prevent="deleteClick(scope.row)"
                  >删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 8px">
        <el-col :span="24" style="text-align: center">
          <el-button type="primary" @click="submitClick">提交流程</el-button>
        </el-col>
      </el-row>
      <div v-if="dialogVisible">
        <el-dialog
          v-model="dialogVisible"
          title="附件操作"
          width="550px"
          class="centered-dialog custom-width-dialog"
        >
          <div style="margin-top: 20px">
            <el-row>
              <el-col :span="4">
                <span>下载模版:</span>
              </el-col>
              <el-col :span="5">
                <a
                  href="#"
                  @click.prevent="downloadTemplate"
                  style="color: #409eff; text-decoration: underline"
                  >物料需求模板表</a
                >
              </el-col>
            </el-row>
            <el-row style="margin-top: 15px">
              <el-col :span="4">
                <span>上传文件（xlsx格式）:</span>
              </el-col>
              <el-col :span="5">
                <input type="file" ref="fileInput" id="file-input" accept=".xlsx" />
              </el-col>
            </el-row>
          </div>
          <el-row>
            <el-col :span="24" style="margin-top: 20px; text-align: center">
              <el-button @click="uploadClick" type="primary">提交</el-button>
              <el-button @click="dialogVisible = false">取消</el-button>
            </el-col>
          </el-row>
        </el-dialog>
      </div>
      <div v-if="dialogVisibleMessage">
        <el-dialog
          v-model="dialogVisibleMessage"
          title="需求信息"
          width="800px"
          style="height: 350px; overflow: auto"
          class="centered-dialog custom-width-dialog"
        >
          <el-row type="flex" justify="center" style="margin-top: 5px">
            <el-col :span="12">
              <el-form-item label="物料名称：" prop="material_name">
                <el-select
                  @change="materialNameChange"
                  v-model="form.material_name"
                  filterable
                  :disabled="materialNameShow"
                  placeholder="请选择物料名称"
                  style="width: 85%; height: 30px"
                >
                  <el-option
                    v-for="item in wzOptions"
                    :key="item.material_code"
                    :label="item.material_name"
                    :value="item.material_code"
                  />
                </el-select>
              </el-form-item>
              <el-form-item label="物料类型：" prop="material_type">
                <el-input
                  :disabled="true"
                  v-model="form.material_type"
                  style="width: 85%; height: 40px"
                  placeholder="请选择物料类型"
                />
              </el-form-item>
              <el-form-item label="需求数量：" prop="num">
                <el-input
                  v-model="form.num"
                  @input="handleInput"
                  style="width: 85%; height: 40px"
                  placeholder="请选择需求数量"
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="物料编码：" prop="material_code">
                <el-input
                  :disabled="true"
                  v-model="form.material_code"
                  style="width: 85%; height: 40px"
                  placeholder="请选择物料编码"
                />
              </el-form-item>
              <el-form-item label="是否易耗品：" prop="consumable">
                <el-radio-group v-model="form.consumable">
                  <el-radio value="1" :disabled="true">是</el-radio>
                  <el-radio value="0" :disabled="true">否</el-radio>
                </el-radio-group>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24" style="text-align: center">
              <el-button type="primary" @click="confirmRoleClick">确认</el-button>
            </el-col>
          </el-row>
        </el-dialog>
      </div>
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
    <div
      v-if="judgeType === '0'"
      style="
        display: flex;
        height: 300px;
        font-size: 30px;
        color: #e68802;
        align-items: center;
        justify-content: center;
      "
      >未到需求申报时间</div
    >
  </div>
</template>

<script>
import {
  wzType,
  submitDemand,
  userMessage,
  searchNextApproval,
  searchDemand,
  judgeDemandTime
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'DemandForecast',
  data() {
    return {
      judgeType: '0',
      busi_id: '',
      flow_no: '1',
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
        num: [{ required: true, message: '请输入需求数量', trigger: 'blur' }],
        material_name: [{ required: true, message: '请输入物料名称', trigger: 'blur' }]
      }
    }
  },
  async created() {
    const responseJudge = await judgeDemandTime({ name: 'demand' }) // 调用 upload 函数并传入 payload
    this.judgeType = responseJudge.data.data
    this.busi_id = this.$route.query.busi_id
    if (this.busi_id != undefined && this.busi_id != '') {
      const responseDemand = await searchDemand({ busi_id: this.busi_id })
      this.form.loanData = responseDemand.data.data
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
    if (this.form.inst_code.substring(5, 6) === '1') {
      this.flow_no = '2'
    } else {
      this.flow_no = '1'
    }
  },
  methods: {
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
            const responseResult = await submitDemand({
              busi_id: this.busi_id,
              flow_no: this.flow_no,
              flow_node: '1',
              flow_title: this.form.user_name + '的需求预估申请',
              approval_content: '',
              flow_node_name: '客户经理发起',
              approval_name: this.form.user_name,
              loanData: this.form.loanData,
              user_id: this.userId,
              user_name: this.form.user_name,
              inst_code: this.form.inst_code,
              inst_name: this.form.inst_name,
              telephone: this.form.telephone,
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
        this.$message.warning('请先新增物料需求表数据')
      } else {
        if (this.form.inst_code.length != 7) {
          this.$message.warning('只有市场部的客户经理能发起需求预估')
        } else {
          const responseResult = await searchNextApproval({
            inst_code: this.form.inst_code,
            flow_no: this.flow_no,
            flow_node: '1'
          })
          this.approveOptions = responseResult.data.data
          this.dialogVisibleApproval = true
        }
      }
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
    addClick() {
      this.form.material_code = ''
      this.form.material_name = ''
      this.material_name = ''
      this.form.material_type = ''
      this.form.consumable = ''
      this.form.num = ''
      this.dialogVisibleMessage = true
      this.materialNameShow = false
    },
    editClick(row) {
      this.dialogVisibleMessage = true
      this.materialNameShow = true
      this.form.material_code = row.material_code
      this.form.material_name = row.material_code
      this.form.material_type = row.material_type
      this.form.consumable = row.consumable
      this.form.num = row.num
    },
    deleteClick(row) {
      const indexToRemove = this.form.loanData.findIndex(
        (item) => item.material_code === row.material_code
      )
      this.form.loanData.splice(indexToRemove, 1)
    },
    confirmRoleClick() {
      //新增
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
          if (!this.materialNameShow) {
            //判断原来数据里面有没有，有的话报错
            let errorType = '1'
            this.form.loanData.forEach((item) => {
              if (item.material_code == this.form.material_code) {
                errorType = '2'
                return
              }
            })
            if (errorType == '1') {
              this.form.loanData.push({
                material_code: this.form.material_code,
                material_name: this.material_name,
                material_type: this.form.material_type,
                consumable: this.form.consumable,
                num: this.form.num
              })
              this.dialogVisibleMessage = false
            } else {
              this.$message.error('该物料已存在！')
            }
            // 修改
          } else {
            this.form.loanData.forEach((item) => {
              if ((item.material_code = this.form.material_code)) {
                item.num = this.form.num
              }
            })
            this.dialogVisibleMessage = false
          }
        }
      }, 300)
    },
    materialNameChange() {
      this.form.material_code = this.form.material_name
      this.wzOptions.forEach((item) => {
        if (item.material_code == this.form.material_code) {
          this.material_name = item.material_name
          this.form.consumable = item.consumable
          this.form.material_type = item.material_type
        }
      })
    },
    importClick() {
      this.$nextTick(() => {
        if (this.$refs.fileInput !== undefined) {
          this.$refs.fileInput.value = null
        }
      })
      this.dialogVisible = true
    },
    //下载物料需求模版表
    async downloadTemplate() {
      const responseWzType = await wzType({})
      this.wzOptions = responseWzType.data.data
      const data = []
      data.push(['物料编码', '物料名称', '物料类型', '是否易耗品(填1或0,1是0否)', '需求数量'])
      const wb = XLSX.utils.book_new()
      const ws = XLSX.utils.aoa_to_sheet(data) // Convert array of arrays to sheet
      ws['!cols'] = this.calculateColumnWidths(data)
      XLSX.utils.book_append_sheet(wb, ws, '物料需求模版表') // Append data to workbookk

      const dataWz = []
      dataWz.push(['物料编码', '物料名称', '物料类型', '是否易耗品(1是0否)'])
      this.wzOptions.forEach((result) => {
        dataWz.push([
          result.material_code,
          result.material_name,
          result.material_type,
          result.consumable
        ])
      })
      const wsWz = XLSX.utils.aoa_to_sheet(dataWz) // Convert array of arrays to sheet
      wsWz['!cols'] = this.calculateColumnWidths(dataWz)
      XLSX.utils.book_append_sheet(wb, wsWz, '物料基础表') // Append data to the same workbook

      // Export the workbook
      const excelBlob = new Blob([XLSX.write(wb, { bookType: 'xlsx', type: 'array' })], {
        type: 'application/octet-stream'
      })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(excelBlob) // Create blob URL
      link.setAttribute('download', '物料需求模版表.xlsx') // Set file name
      document.body.appendChild(link)
      link.click() // Trigger download
      document.body.removeChild(link) // Clean up the link
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
