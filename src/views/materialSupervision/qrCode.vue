<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="150px">
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
          <el-form-item label="终端层级：" prop="terminal_level">
            <el-input
              class="inputClass"
              v-model="form.terminal_level"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="物料名称：" prop="id">
            <el-select
              @change="materialChange"
              v-model="form.id"
              placeholder="请选择物料名称"
              style="width: 85%; height: 30px"
            >
              <el-option
                v-for="item in materialNameOptions"
                :key="item.id"
                :label="item.material_name"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="附件上传：" prop="fileList">
            <el-upload
              ref="upload"
              v-model:file-list="form.fileList"
              :action="upLoadUrl"
              class="upload-demo"
              multiple
              :on-remove="handleRemove"
              :limit="5"
              :data="{ name: file_id }"
            >
              <el-button type="primary">选择附件</el-button>
            </el-upload>
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
          <el-form-item label="原二维码：" prop="qr_code">
            <el-input
              class="inputClass"
              v-model="form.qr_code"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 5px">
        <el-col :span="18" style="text-align: center">
          <el-form-item label="变更原因：" prop="material_reason">
            <el-input
              :rows="2"
              type="textarea"
              class="inputClass"
              v-model="form.material_reason"
              style="width: 94%; height: 40px"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 10px; text-align: left">
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
      <el-row type="flex" justify="center" style="margin-top: 8px; margin-bottom: 10px">
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
import axios from 'axios'
import {
  searchQrCode,
  searchCustList,
  userMessage,
  searchNextApproval,
  submitQrCodeChangeApply,
  searchQrCodeChangeApply
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'QrCode',
  data() {
    return {
      filesArray: [],
      files: [],
      materialCodeArray: [],
      materialNameArray: [],
      qrCodeArray: [],
      materialCodeString: '',
      materialNameString: '',
      qrCodeString: '',
      PATH_URL: import.meta.env.VITE_API_BASE_PATH,
      upLoadUrl: import.meta.env.VITE_API_BASE_PATH + '/uploads/uploads.php',
      file_id: '',
      customArray: [],
      wzArray: [],
      busi_id: '',
      flow_no: '8',
      userId: '',
      dialogVisibleApproval: false,
      approveOptions: [],
      materialNameOptions: [],
      form: {
        id: '',
        qr_code: '',
        inst_code: '',
        inst_name: '',
        user_id: '',
        user_name: '',
        telephone: '',
        custom_name: '',
        terminal_level: '',
        material_reason: '',
        fileList: [],
        custom_license: '',
        operator_telephone: '',
        material_code: '',
        next_approval_id: '',
        next_approval_name: ''
      },
      formRules: {
        next_approval_id: [{ required: true, message: '请选择审批人', trigger: 'blur' }],
        custom_name: [{ required: true, message: '请输入客户名称', trigger: 'blur' }],
        custom_license: [{ required: true, message: '选择客户名称后返显', trigger: 'blur' }],
        material_reason: [{ required: true, message: '请输入变更原因', trigger: 'blur' }]
        // fileList: [{ required: true, message: '请上传附件', trigger: 'blur' }]
      }
    }
  },
  async created() {
    const userStore = useUserStore()
    const loginInfo = userStore.getUserInfo
    this.userId = loginInfo.userId
    this.form.user_id = loginInfo.userId
    this.busi_id = this.$route.query.busi_id
    this.file_id = loginInfo.userId + Date.now()
    if (this.busi_id != undefined && this.busi_id != '') {
      const responseCheck = await searchQrCodeChangeApply({ busi_id: this.busi_id })
      this.form.custom_name = responseCheck.data.data[0].custom_name
      this.file_id = responseCheck.data.data[0].file_id
      const responseFile = await axios.get(
        this.PATH_URL + `/uploads/list_files.php?name=${this.file_id}`
      )
      if (responseFile.data.files) {
        this.filesArray = []
        this.files = responseFile.data.files // 将返回的文件列表保存到 data 中
        this.files.forEach((item) => {
          this.filesArray.push({
            name: item,
            url: this.PATH_URL + '/uploads/' + this.file_id + '/' + item,
            response: {
              file: 'uploads/' + this.file_id + '/' + item
            }
          })
        })
        this.form.fileList = this.filesArray
      } else {
        console.error(responseFile.data.message || '无法获取文件列表')
      }
      this.form.terminal_level = responseCheck.data.data[0].terminal_level
      this.form.material_reason = responseCheck.data.data[0].material_reason
      this.form.custom_license = responseCheck.data.data[0].custom_license
      this.form.operator_telephone = responseCheck.data.data[0].operator_telephone
      this.form.material_code = responseCheck.data.data[0].material_code
      this.form.material_name = responseCheck.data.data[0].material_name
      this.form.qr_code = responseCheck.data.data[0].qr_code
      this.form.id = responseCheck.data.data[0].qr_id
      const qrResponse = await searchQrCode({ custom_license: this.form.custom_license })
      this.materialNameOptions = qrResponse.data.data
    } else {
      this.busi_id = ''
    }
    const response = await userMessage({ userId: this.userId })
    this.form.user_name = response.data.data[0].employee_name
    this.form.inst_code = response.data.data[0].inst_code
    this.form.inst_name = response.data.data[0].inst_name
    this.form.telephone = response.data.data[0].telephone
    if (this.form.inst_code.substring(5, 6) === '1') {
      this.flow_no = '9'
    } else {
      this.flow_no = '8'
    }
  },
  unmounted() {
    if (this.busi_id == undefined || this.busi_id == '') {
      axios
        .post(import.meta.env.VITE_API_BASE_PATH + '/uploads/deleteFile.php', {
          fileId: this.file_id // 假设 file.url 是文件的完整路径
        })
        .then((response) => {})
        .catch((error) => {
          this.$message.error('网络错误：' + error.message)
        })
    }
  },
  methods: {
    materialChange() {
      this.materialNameOptions.forEach((item) => {
        if (this.form.id === item.id) {
          this.form.qr_code = item.qr_code
          this.form.material_code = item.material_code
          this.form.material_name = item.material_name
        }
      })
    },
    handleRemove(file, uploadFiles) {
      axios
        .post(import.meta.env.VITE_API_BASE_PATH + '/uploads/delete.php', {
          filePath: file.response.file // 假设 file.url 是文件的完整路径,
        })
        .then((response) => {
          if (response.data.success) {
            this.$message.success('文件删除成功')
          } else {
            this.$message.error('文件删除失败：' + response.data.message)
          }
        })
        .catch((error) => {
          this.$message.error('网络错误：' + error.message)
        })
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
          terminal_level: item.terminal_level,
          custom_license: item.custom_license,
          operator_telephone: item.operator_telephone
        })
      })
      cb(this.customArray)
    },
    async handleSelect(item) {
      this.form.custom_name = item.custom_name
      this.form.terminal_level = item.terminal_level
      this.form.custom_license = item.custom_license
      this.form.operator_telephone = item.operator_telephone
      const qrResponse = await searchQrCode({ custom_license: item.custom_license })
      this.materialNameOptions = qrResponse.data.data
      this.$forceUpdate()
    },
    inputName() {
      this.form.terminal_level = ''
      this.form.custom_license = ''
      this.form.operator_telephone = ''
      this.form.id = ''
      this.form.qr_code = ''
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
        if (this.form.fileList.length == 0) {
          this.file_id = ''
        }
        const responseResult = await submitQrCodeChangeApply({
          busi_id: this.busi_id,
          flow_no: this.flow_no,
          flow_node: '1',
          flow_title: this.form.custom_name + '的二维码变更申请',
          approval_content: '',
          flow_node_name: '客户经理发起',
          approval_name: this.form.user_name,
          user_id: this.userId,
          material_code: this.form.material_code,
          material_name: this.form.material_name,
          qr_code: this.form.qr_code,
          qr_id: this.form.id,
          user_name: this.form.user_name,
          inst_code: this.form.inst_code,
          inst_name: this.form.inst_name,
          telephone: this.form.telephone,
          next_approval_id: this.form.next_approval_id,
          next_approval_name: this.form.next_approval_name,
          custom_name: this.form.custom_name,
          terminal_level: this.form.terminal_level,
          material_reason: this.form.material_reason,
          file_id: this.file_id,
          custom_license: this.form.custom_license,
          operator_telephone: this.form.operator_telephone
        })
        this.dialogVisibleApproval = false
        this.busi_id = responseResult.data.busi_id
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
            this.$message.warning('只有市场部的客户经理能发起二维码变更申请')
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
</style>
