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
          <el-form-item label="客户经度：" prop="longitude">
            <el-input
              class="inputClass"
              v-model="form.longitude"
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
          <el-form-item label="客户纬度：" prop="latitude">
            <el-input
              class="inputClass"
              v-model="form.latitude"
              :disabled="true"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 5px">
        <el-col :span="12">
          <el-table
            :data="form.loanData"
            :header-cell-style="{ color: '#212121' }"
            style="width: 99%; height: 200px; margin-top: 5px; text-align: center"
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
              prop="qr_code"
              align="center"
              header-align="center"
              label="物料二维码编号"
              width="auto"
            />
          </el-table>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 10px">
        <el-col :span="9">
          <el-form-item label="盘检经度：" prop="check_longitude">
            <el-input
              class="inputClass"
              @change="longChange"
              v-model="form.check_longitude"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="盘检时间：" prop="check_date">
            <el-date-picker
              style="width: 85%; height: 40px"
              v-model="form.check_date"
              type="date"
              size="small"
            />
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
              <template #tip>
                <div class="el-upload__tip">请上传物料的二维码照片</div>
              </template>
            </el-upload>
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="盘检纬度：" prop="check_latitude">
            <el-input
              class="inputClass"
              @change="longChange"
              v-model="form.check_latitude"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="是否超过100米：" prop="is_normal">
            <el-radio-group v-model="form.is_normal">
              <el-radio value="1" :disabled="true">是</el-radio>
              <el-radio value="0" :disabled="true">否</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row type="flex" justify="center" style="margin-top: 5px">
        <el-col :span="18" style="text-align: center">
          <el-form-item label="物料信息：" prop="material_message">
            <el-input
              :rows="2"
              type="textarea"
              class="inputClass"
              v-model="form.material_message"
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
  searchDepartWz,
  searchCustList,
  userMessage,
  searchNextApproval,
  submitInventoryCheckApply,
  searchInventoryCheckApply
} from '@/api/login'
import * as XLSX from 'xlsx'
import { useUserStore } from '@/store/modules/user'
export default {
  name: 'InventoryCheck',
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
      flow_no: '6',
      materialNameShow: true,
      userId: '',
      dialogVisibleApproval: false,
      approveOptions: [],
      form: {
        loanData: [],
        inst_code: '',
        inst_name: '',
        user_id: '',
        user_name: '',
        telephone: '',
        custom_name: '',
        longitude: '',
        is_normal: '',
        check_longitude: '',
        check_date: '',
        material_message: '',
        fileList: [
          // {
          //   name: 'example1.png',
          //   url: import.meta.env.VITE_API_BASE_PATH + '/uploads/1906011736316854071/机构表 (1).xlsx'
          // },
          // {
          //   name: 'example2.jpg',
          //   url: import.meta.env.VITE_API_BASE_PATH + '/uploads/1906011736316854071/员工表.xlsx'
          // }
        ],
        custom_license: '',
        latitude: '',
        check_latitude: '',
        material_code: '',
        next_approval_id: '',
        next_approval_name: ''
      },
      formRules: {
        next_approval_id: [{ required: true, message: '请选择审批人', trigger: 'blur' }],
        custom_name: [{ required: true, message: '请输入客户名称', trigger: 'blur' }],
        custom_license: [{ required: true, message: '选择客户名称后返显', trigger: 'blur' }],
        is_normal: [{ required: true, message: '未自动返显，重新输入盘检经纬度', trigger: 'blur' }],
        check_longitude: [{ required: true, message: '请输入盘检经度', trigger: 'blur' }],
        check_latitude: [{ required: true, message: '请输入盘检纬度', trigger: 'blur' }],
        check_date: [{ required: true, message: '请输入盘检时间', trigger: 'blur' }],
        material_message: [{ required: true, message: '请输入盘检信息', trigger: 'blur' }],
        fileList: [{ required: true, message: '请上传附件', trigger: 'blur' }]
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
      const responseCheck = await searchInventoryCheckApply({ busi_id: this.busi_id })
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
      this.form.longitude = responseCheck.data.data[0].longitude
      this.form.is_normal = responseCheck.data.data[0].is_normal
      this.form.check_longitude = responseCheck.data.data[0].check_longitude
      this.form.check_date = responseCheck.data.data[0].check_date
      this.form.material_message = responseCheck.data.data[0].material_message
      this.form.custom_license = responseCheck.data.data[0].custom_license
      this.form.latitude = responseCheck.data.data[0].latitude
      this.form.check_latitude = responseCheck.data.data[0].check_latitude
      this.form.material_code = responseCheck.data.data[0].material_code
      this.materialCodeString = responseCheck.data.data[0].material_code_string
      this.materialNameString = responseCheck.data.data[0].material_name_string
      this.qrCodeString = responseCheck.data.data[0].qr_code_string
      if (this.materialCodeString.indexOf(',') > -1) {
        this.materialCodeArray = this.materialCodeString.split(',')
      } else {
        this.materialCodeArray.push(this.materialCodeString)
      }
      if (this.materialNameString.indexOf(',') > -1) {
        this.materialNameArray = this.materialNameString.split(',')
      } else {
        this.materialNameArray.push(this.materialNameString)
      }
      if (this.qrCodeString.indexOf(',') > -1) {
        this.qrCodeArray = this.qrCodeString.split(',')
      } else {
        this.qrCodeArray.push(this.qrCodeString)
      }
      this.form.loanData = []
      this.materialCodeArray.forEach((item, index) => {
        this.form.loanData.push({
          material_name: this.materialNameArray[index],
          material_code: this.materialCodeArray[index],
          qr_code: this.qrCodeArray.length > index ? this.qrCodeArray[index] : ''
        })
      })
    } else {
      this.busi_id = ''
    }
    const response = await userMessage({ userId: this.userId })
    this.form.user_name = response.data.data[0].employee_name
    this.form.inst_code = response.data.data[0].inst_code
    this.form.inst_name = response.data.data[0].inst_name
    this.form.telephone = response.data.data[0].telephone
    if (this.form.inst_code.substring(5, 6) === '1') {
      this.flow_no = '7'
    } else {
      this.flow_no = '6'
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
    longChange() {
      if (
        this.form.check_latitude != '' &&
        this.form.check_longitude != '' &&
        this.form.latitude != '' &&
        this.form.longitude != ''
      ) {
        const R = 6371000 // 地球平均半径（米）
        const dLat = (this.form.check_latitude - this.form.latitude) * (Math.PI / 180)
        const dLon = (this.form.check_longitude - this.form.longitude) * (Math.PI / 180)
        const a =
          Math.sin(dLat / 2) * Math.sin(dLat / 2) +
          Math.cos(this.form.latitude * (Math.PI / 180)) *
            Math.cos(this.form.check_latitude * (Math.PI / 180)) *
            Math.sin(dLon / 2) *
            Math.sin(dLon / 2)
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
        const distance = R * c
        if (distance > 100) {
          this.form.is_normal = '1'
        } else {
          this.form.is_normal = '0'
        }
      }
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
          longitude: item.longitude,
          custom_license: item.custom_license,
          latitude: item.latitude
        })
      })
      cb(this.customArray)
    },
    async handleSelect(item) {
      this.form.custom_name = item.custom_name
      this.form.longitude = item.longitude
      this.form.custom_license = item.custom_license
      this.form.latitude = item.latitude
      const qrResponse = await searchQrCode({ custom_license: item.custom_license })
      this.form.loanData = qrResponse.data.data
      this.$forceUpdate()
    },
    inputName() {
      this.form.longitude = ''
      this.form.custom_license = ''
      this.form.latitude = ''
    },
    nextApprovalIdChange() {
      this.approveOptions.forEach((item) => {
        if (item.employee_code === this.form.next_approval_id) {
          this.form.next_approval_name = item.employee_name
        }
      })
    },
    async confirmApprovalClick() {
      if (this.form.loanData == null || this.form.loanData.length == 0) {
        this.materialCodeString = ''
        this.materialNameString = ''
        this.qrCodeString = ''
      } else if (this.form.loanData.length == 1) {
        this.materialCodeString = this.form.loanData[0].material_code
        this.materialNameString = this.form.loanData[0].material_name
        this.qrCodeString = this.form.loanData[0].qr_code
      } else if (this.form.loanData.length > 1) {
        this.materialCodeArray = []
        this.materialNameArray = []
        this.qrCodeArray = []
        this.form.loanData.forEach((item) => {
          this.materialCodeArray.push(item.material_code)
          this.materialNameArray.push(item.material_name)
          this.qrCodeArray.push(item.qr_code)
        })
        this.materialCodeString = this.materialCodeArray.join(',')
        this.materialNameString = this.materialNameArray.join(',')
        this.qrCodeString = this.qrCodeArray.join(',')
      }
      try {
        const responseResult = await submitInventoryCheckApply({
          busi_id: this.busi_id,
          flow_no: this.flow_no,
          flow_node: '1',
          flow_title: this.form.custom_name + '的盘检申请',
          approval_content: '',
          flow_node_name: '客户经理发起',
          approval_name: this.form.user_name,
          user_id: this.userId,
          material_code_string: this.materialCodeString,
          material_name_string: this.materialNameString,
          qr_code_string: this.qrCodeString,
          user_name: this.form.user_name,
          inst_code: this.form.inst_code,
          inst_name: this.form.inst_name,
          telephone: this.form.telephone,
          next_approval_id: this.form.next_approval_id,
          next_approval_name: this.form.next_approval_name,
          custom_name: this.form.custom_name,
          longitude: this.form.longitude,
          is_normal: this.form.is_normal,
          check_date: this.form.check_date,
          material_message: this.form.material_message,
          file_id: this.file_id,
          check_longitude: this.form.check_longitude,
          custom_license: this.form.custom_license,
          latitude: this.form.latitude,
          check_latitude: this.form.check_latitude,
          material_code: this.form.material_code
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
            this.$message.warning('只有市场部的客户经理能发起物料盘检申请')
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
