<template>
  <div class="container">
    <el-form :model="form" ref="formRef" :rules="formRules" label-width="130px">
      <el-row type="flex" justify="center" style="margin-top: -15px; text-align: left">
        <el-col :span="9">
          <el-form-item label="客户名称：" prop="custom_name">
            <el-input
              v-model="form.custom_name"
              placeholder="请输入客户名称"
              :disabled="true"
              style="width: 85%; height: 40px"
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
              :disabled="true"
              v-model="form.check_longitude"
              style="width: 85%; height: 40px"
            />
          </el-form-item>
          <el-form-item label="盘检时间：" prop="check_date">
            <el-date-picker
              :disabled="true"
              style="width: 85%; height: 40px"
              v-model="form.check_date"
              type="date"
              size="small"
            />
          </el-form-item>
          <el-form-item label="附件内容：" prop="file_id">
            <div v-if="files.length > 0">
              <ul>
                <li v-for="(file, index) in files" :key="index" style="cursor: pointer">
                  <span
                    style="text-decoration: underline; white-space: nowrap"
                    @click="downloadFile(file)"
                    >{{ file }}</span
                  >
                </li>
              </ul>
            </div>
          </el-form-item>
        </el-col>
        <el-col :span="9">
          <el-form-item label="盘检纬度：" prop="check_latitude">
            <el-input
              class="inputClass"
              :disabled="true"
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
              :disabled="true"
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
import axios from 'axios'
import {
  submitInventoryCheckApply,
  userMessage,
  searchNextApproval,
  searchInventoryCheckApply,
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
  name: 'InventoryCheckApprove',
  data() {
    return {
      files: [], // 存储文件列表
      materialCodeArray: [],
      materialNameArray: [],
      qrCodeArray: [],
      materialCodeString: '',
      materialNameString: '',
      qrCodeString: '',
      PATH_URL: import.meta.env.VITE_API_BASE_PATH,
      upLoadUrl: import.meta.env.VITE_API_BASE_PATH + '/uploads/uploads.php',
      file_id: '',
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
        fileList: [],
        custom_license: '',
        latitude: '',
        check_latitude: '',
        material_code: '',
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
    const responseCheck = await searchInventoryCheckApply({ busi_id: this.busi_id })
    this.form.custom_name = responseCheck.data.data[0].custom_name
    this.file_id = responseCheck.data.data[0].file_id
    this.form.longitude = responseCheck.data.data[0].longitude
    this.form.is_normal = responseCheck.data.data[0].is_normal
    this.form.check_longitude = responseCheck.data.data[0].check_longitude
    this.form.check_date = responseCheck.data.data[0].check_date
    this.form.material_message = responseCheck.data.data[0].material_message
    this.form.fileList = responseCheck.data.data[0].fileList
    this.form.custom_license = responseCheck.data.data[0].custom_license
    this.form.latitude = responseCheck.data.data[0].latitude
    this.form.check_latitude = responseCheck.data.data[0].check_latitude
    this.form.material_code = responseCheck.data.data[0].material_code
    this.form.user_id = responseCheck.data.data[0].user_id
    this.materialCodeString = responseCheck.data.data[0].material_code_string
    this.materialNameString = responseCheck.data.data[0].material_name_string
    this.form.telephone = responseCheck.data.data[0].telephone
    this.form.user_id = responseCheck.data.data[0].user_id
    this.form.user_name = responseCheck.data.data[0].user_name
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
    const responseFile = await axios.get(
      this.PATH_URL + `/uploads/list_files.php?name=${this.file_id}`
    )
    if (responseFile.data.files) {
      this.files = responseFile.data.files // 将返回的文件列表保存到 data 中
    } else {
      console.error(responseFile.data.message || '无法获取文件列表')
    }
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
    async downloadFile(file) {
      try {
        const response = await axios({
          url: this.PATH_URL + `/uploads/download.php?name=${this.file_id}&file=${file}`, // 请求下载文件
          method: 'GET',
          responseType: 'blob' // 设置响应为 blob 类型，表示文件
        })
        // 创建 URL 对象并触发下载
        const url = window.URL.createObjectURL(new Blob([response.data]))
        const link = document.createElement('a')
        link.href = url
        link.setAttribute('download', file) // 设置下载文件名
        document.body.appendChild(link)
        link.click() // 触发下载
      } catch (error) {
        console.error('文件下载失败', error)
        alert('文件下载失败')
      }
    },
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
            const responseResult = await submitInventoryCheckApply({
              busi_id: this.busi_id,
              flow_no: this.flow_no,
              flow_node: this.flow_node,
              flow_title: this.flow_title,
              approval_content: this.form.approval_content,
              flow_node_name: this.flow_node_name,
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
:deep(.el-textarea.is-disabled .el-textarea__inner) {
  background-color: var(--el-disabled-bg-color);
  box-shadow: 0 0 0 1px var(--el-disabled-border-color) inset;
  color: #000000;
  cursor: not-allowed;
}
</style>
