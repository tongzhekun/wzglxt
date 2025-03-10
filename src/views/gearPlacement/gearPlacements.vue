<template>
  <div class="container">
    <el-form :model="formData" ref="formRef">
      <el-row type="flex" justify="center">
        <el-col :span="6" style="text-align: left">
          <span style="font-size: 20px; color: #000">档位投放明细</span>
        </el-col>
        <el-col :span="4" style="text-align: left">
          <el-form-item label="周期数据" prop="tier">
            <el-select
              v-model="formData.serialId"
              placeholder="请选择周期数据"
              @change="getperiodDataById"
            >
              <!-- <el-option >
                   2024-10-21至2024-10-27卷烟拟合数据
                </el-option>
                <el-option >
                   2024-10-28至2024-11-03卷烟拟合数据
                </el-option> -->
              <el-option v-for="item in periodData" :key="item.serialId" :value="item.serialId">
                {{ item.periodName }}
              </el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="4" style="text-align: left" />
        <el-col :span="10" style="text-align: left">
          <el-button type="info" round @click="paramDialog">卷烟参数设置</el-button>
          <el-button type="danger" round @click="paramSelectDialog">投放规则</el-button>
          <el-button type="success" round @click="calculateDistribution">定量拟合</el-button>
          <el-button type="warning" round @click="exportToCSV">导出为CSV</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="24" style="margin-top: 8px; margin-bottom: 20px; text-align: center">
          <el-table
            :data="allocationResults"
            v-loading="loading"
            element-loading-text="加载中"
            element-loading-svg-view-box="-10, -10, 50, 50"
            element-loading-background="rgba(122,122,122,0.8)"
            style="width: 100%; height: 330px"
          >
            <!-- 烟草名称列，添加点击事件 -->
            <el-table-column prop="name" fixed label="卷烟名称" width="100px">
              <template v-slot="scope">
                <div @click="showTobaccoDetail(scope.row)" style="cursor: pointer">
                  {{ scope.row.name }}
                </div>
              </template>
            </el-table-column>
            <el-table-column prop="stockOld" label="投放量" />
            <el-table-column prop="isSale" label="是否投放" width="100px" />
            <el-table-column prop="remaining" label="预估销量" width="100px" />
            <el-table-column v-for="level in reversedLevels" :key="level" :label="level + '档'">
              <template v-slot="scope">
                <el-input
                  v-if="scope.row.edit"
                  v-model="scope.row.allocations[reversedLevels.indexOf(level)]"
                  size="small"
                  @blur="handleInputBlur(scope.row)"
                />
                <span v-else> {{ scope.row.allocations[reversedLevels.indexOf(level)] }}</span>
              </template>
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
      <el-dialog
        v-model="dialogVisibleParam"
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
                @click.prevent="downloadTemplateParam"
                style="color: #409eff; text-decoration: underline"
                >投放参数表</a
              >
            </el-col>
          </el-row>
          <el-row style="margin-top: 15px">
            <el-col :span="4">
              <span>上传文件(xlsx格式):</span>
            </el-col>
            <el-col :span="5">
              <input
                type="file"
                id="file-input"
                ref="fileInputParam"
                @change="handleFileUploadParam"
                accept=".xlsx"
              />
            </el-col>
          </el-row>
        </div>
        <el-row>
          <el-col :span="24" style="margin-top: 20px; text-align: center">
            <el-button @click="uploadParamClick" type="primary">提交</el-button>
            <el-button @click="dialogVisibleParam = false">取消</el-button>
          </el-col>
        </el-row>
      </el-dialog>

      <el-dialog
        v-model="dialogVisibleParamSelect"
        title="投放规则界面"
        width="550px"
        class="centered-dialog custom-width-dialog"
      >
        <div style="margin-top: 20px">
          <el-row :gutter="20">
            <!-- 档位间差距大小 -->
            <el-col :span="24">
              <el-form-item label="档位间差距大小" prop="tier">
                <el-select v-model="formData.selectedLevel" placeholder="请选择档位">
                  <el-option label="高" :value="1" />
                  <el-option label="中" :value="2" />
                  <el-option label="低" :value="3" />
                </el-select>
              </el-form-item>
            </el-col>

            <!-- 投放范围 -->
            <el-col :span="24">
              <el-form-item label="高价烟投放范围" prop="range">
                <el-select v-model="formData.selectedRange" placeholder="请选择档位">
                  <el-option label="30-25档" :value="5" />
                  <el-option label="30-20档" :value="4" />
                  <el-option label="30-15档" :value="3" />
                  <el-option label="30-1档" :value="2" />
                </el-select>
              </el-form-item>
            </el-col>

            <!-- 投放量 -->
            <el-col :span="24">
              <el-form-item label="客户高档位投放量大于低档位投放量" prop="amount">
                <el-select v-model="formData.selectedWeekMonth" placeholder="请选择">
                  <el-option label="月高于" :value="5" />
                  <el-option label="周高于" :value="4" />
                </el-select>
              </el-form-item>
            </el-col>

            <!-- 差异性 -->
            <el-col :span="24">
              <el-form-item label="客户之间本周存销比差异性" prop="difference">
                <el-select v-model="formData.selectedweekDifference" placeholder="请选择">
                  <el-option label="等于上轮" :value="5" />
                  <el-option label="小于上轮" :value="4" />
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
        </div>
        <el-row>
          <el-col :span="24" style="margin-top: 20px; text-align: center">
            <el-button @click="uploadParamSelectClick" type="primary">确认</el-button>
            <el-button @click="dialogVisibleParamSelect = false">取消</el-button>
          </el-col>
        </el-row>
      </el-dialog>

      <!-- 弹出框，显示烟草详情 -->
      <div v-if="dialogDetailVisible">
        <el-dialog
          v-model="dialogDetailVisible"
          width="550px"
          :title="tobaccoName"
          class="centered-dialog custom-width-dialog"
        >
          <div style="margin-top: 20px">
            <el-row :gutter="20">
              <!-- 预测批发量 -->
              <el-col :span="24">
                <el-form-item label="预测批发量：" prop="tier">
                  <el-input v-model="formData.preWholesale" disabled />
                </el-form-item>
              </el-col>

              <!-- 预测动销率 -->
              <el-col :span="24">
                <el-form-item label="预测动销率：" prop="range">
                  <el-input v-model="formData.preFixRate" disabled />
                </el-form-item>
              </el-col>

              <!-- 预测社会销量 -->
              <el-col :span="24">
                <el-form-item label="预测社会销量：" prop="amount">
                  <el-input v-model="formData.preSocitySale" disabled />
                </el-form-item>
              </el-col>

              <!-- 预测社会存销比 -->
              <el-col :span="24">
                <el-form-item label="预测社会存销比：" prop="difference">
                  <el-input v-model="formData.preSocityStockSaleRate" disabled />
                </el-form-item>
              </el-col>
            </el-row>
          </div>
        </el-dialog>
      </div>
    </el-form>
  </div>
</template>

<script>
import {
  uploadTobacco,
  downloadTobacco,
  queryTobacco,
  uploadCust,
  downloadCust,
  queryCust,
  uploadDecayRule,
  downloadDecayRule,
  queryDecayRules,
  saveCalculateDistribution,
  queryTobaccoBySku,
  savePeriod,
  queryPeriod,
  queryPeriodBySerialId,
  queryCalculateDistributionBySerialId
} from '@/api/login'
import { useForm } from '@/hooks/web/useForm'
import * as XLSX from 'xlsx'
const { formRegister, formMethods } = useForm()
const { getFormData, getElFormExpose, setValues } = formMethods
export default {
  name: 'GearPlacementDemo',
  data() {
    return {
      tobaccoName: '卷烟详情',
      totalSumFinal: Number,
      loading: false,
      Reduceratio: Number,
      stock30: '',
      stock1: '',
      serialId: '',
      serialIdNew: '',
      societySalesDate: '',
      allocations: [],
      downTobaccoArray: [],
      downCustArray: [],
      downDecayRuleArray: [],
      uploadArrayTobacco: [],
      uploadArrayCust: [],
      uploadArrayDecayRule: [],
      sheets: [],
      PATH_URL: import.meta.env.VITE_API_BASE_PATH,
      dialogVisibleParam: false,
      dialogVisibleParamSelect: false,
      dialogDetailVisible: false,
      formData: {
        ratio: 5,
        selectedDate: '',
        selectedLevel: '',
        selectedRange: '',
        selectedWeekMonth: '',
        selectedweekDifference: ''
      },
      detailData: {
        a: 1
      },
      formRules: {
        selectedDate: [{ required: true, message: '日期不能为空', trigger: 'blur' }]
      },
      selectedDate: '',
      reversedLevels: '',
      levels: Array.from({ length: 30 }, (_, index) => index + 1),
      customerLevels: 30,
      customers: Array(30).fill(100),
      skus: [],
      decayRuleArray: [],
      allocationResults: [],
      newAllocationResults: [],
      period: {},
      periodData: []
    }
  },
  created() {
    this.reversedLevels = this.levels.slice().reverse()
    // 在组件创建时调用后端 API 获取数据
    this.getPriodData()
  },
  methods: {
    async getPriodData() {
      //查询所有的周期
      const payloadPeriod = {}
      const responsePeriod = await queryPeriod(payloadPeriod)
      if (responsePeriod.data.code == 200) {
        const periodArray = responsePeriod.data.data.list
        this.periodData = periodArray
      } else {
        this.$message.error(responsePeriod.data.data.message)
      }
    },

    async getperiodDataById() {
      //查询所有的周期
      const payloadCDById = {
        serialId: this.formData.serialId
      }
      const responseCDById = await queryCalculateDistributionBySerialId(payloadCDById)
      if (responseCDById.data.code == 200) {
        const CDByIdArray = responseCDById.data.data.list
        const allocationTemp = []
        for (let i = 0; i < CDByIdArray.length; i++) {
          this.allocationResults.push({
            sku: CDByIdArray[i].sku,
            name: CDByIdArray[i].cigName,
            stockOld: CDByIdArray[i].stocksSale,
            isSale: CDByIdArray[i].isSale == 0 ? '否' : '是',
            remaining: CDByIdArray[i].stocksPre,
            allocations: CDByIdArray[i].allocations
          })
        }
      } else {
        this.$message.error(responseCDById.data.data.message)
      }
    },

    //上传附件把投放的参数表存入临时表内
    async uploadParamClick() {
      try {
        //烟草参数上传
        const payloadTobacco = {
          skus: this.uploadArrayTobacco
        }
        const responseTobacco = await uploadTobacco(payloadTobacco) //
        //客户参数上传
        const payloadCust = {
          customers: this.uploadArrayCust
        }
        const responseCust = await uploadCust(payloadCust) //
        //衰减速率参数上传
        const payloadDecayRule = {
          decayRules: this.uploadArrayDecayRule
        }
        const responseDecayRule = await uploadDecayRule(payloadDecayRule) //

        // this.uploadArrayDecayRule.push(rowObject)
        //保存批次号
        const payloadPeriod = {
          period: this.period
        }

        const responsePeriod = await savePeriod(payloadPeriod) //

        if (
          responseTobacco.data.code == 200 &&
          responseCust.data.code == 200 &&
          responseDecayRule.data.code == 200 &&
          responsePeriod.data.code == 200
        ) {
          this.$message.success('上传成功！')
          this.dialogVisibleParam = false // 关闭弹窗
        } else {
          this.$message.error('上传失败')
        }
      } catch (error) {
        this.$message.error('上传失败')
      }
    },

    //上传附件把投放的参数表存入临时表内
    async uploadParamSelectClick() {
      if (this.formData.selectedLevel === '1') {
        this.formData.ratio = 5
      } else if (this.formData.selectedLevel === '2') {
        this.formData.ratio = 3
      } else if (this.formData.selectedLevel === '3') {
        this.formData.ratio = 2
      } else {
        this.formData.ratio = 5
      }
      this.dialogVisibleParamSelect = false // 关闭弹窗
    },

    //烟草参数弹窗
    paramDialog() {
      this.$nextTick(() => {
        if (this.$refs.fileInputParam !== undefined) {
          this.$refs.fileInputParam.value = null
        }
      })
      this.dialogVisibleParam = true
    },

    //投放规则弹窗
    paramSelectDialog() {
      this.$nextTick(() => {
        if (this.$refs.fileInputParam !== undefined) {
          this.$refs.fileInputParam.value = null
        }
      })
      this.dialogVisibleParamSelect = true
    },

    //香烟详情弹窗
    // 点击烟草名称时显示详情
    async showTobaccoDetail(tobacco) {
      this.dialogDetailVisible = true
      this.$forceUpdate()
      this.tobaccoName = '下轮预测情况——' + tobacco.name

      //查询所有的香烟
      const payloadTobaccoBySku = {
        sku: tobacco.sku
      }
      const responseTobaccoBySku = await queryTobaccoBySku(payloadTobaccoBySku) // 调用 upload 函数并传入 payload
      if (responseTobaccoBySku.data.code == 200) {
        const tobaccoBySku = responseTobaccoBySku.data.data.list[0]
        this.societySalesDate = tobaccoBySku.societySalesDate
        const inputDateString = `${this.societySalesDate.slice(0, 4)}-${this.societySalesDate.slice(4, 6)}-${this.societySalesDate.slice(6, 8)}`
        // 1. 将字符串转换为日期对象
        const inputDate = new Date(inputDateString)
        // 2. 获取当年1月1日的日期对象
        const startOfYear = new Date(inputDate.getFullYear(), 0, 1)
        // 3. 计算两者之间的天数差
        const timeDiff = inputDate - startOfYear
        const daysDiff = timeDiff / (1000 * 3600 * 24) // 毫秒转天数
        this.formData.preWholesale = tobaccoBySku.weekSalesNum
        this.formData.preFixRate = (tobaccoBySku.fixSaleRate / 100).toFixed(2)
        this.formData.preSocitySale = (
          tobaccoBySku.weekSalesNum *
          (tobaccoBySku.fixSaleRate / 100)
        ).toFixed(2)
        const a = parseFloat(tobaccoBySku.weekSalesNum + tobaccoBySku.societyStock)
        const b = parseFloat(tobaccoBySku.weekSalesNum) * parseFloat(tobaccoBySku.fixSaleRate / 100)
        const c = tobaccoBySku.societySales / daysDiff
        this.formData.preSocityStockSaleRate = ((a - b) / c).toFixed(2)
      } else {
        this.$message.error('不存在该香烟')
      }
    },

    // 输入框失去焦点时的处理（根据你的需求进行修改）
    handleInputBlur(row) {
      row.edit = false
    },

    //上传参数
    handleFileUploadParam(event) {
      const fileInput = document.getElementById('file-input')
      const file = fileInput.files[0] // 获取用户选择的文件
      if (!file || !file.name.endsWith('.xlsx')) {
        alert('请先选择一个文件')
        return
      }
      const reader = new FileReader() // 创建一个 FileReader 实例
      return new Promise((resolve, reject) => {
        // 读取文件内容
        reader.onload = (e) => {
          const data = e.target.result
          // 使用 xlsx 库解析文件内容
          const workbook = XLSX.read(data, { type: 'array' })
          // 获取名为 'Sheet1' 的工作表
          const sheetTobacco = workbook.Sheets['卷烟参数表']
          const sheetCust = workbook.Sheets['客户参数表']
          const sheetDecayRule = workbook.Sheets['衰减速率参数表']
          // 如果 'Sheet1' 不存在，提醒用户
          if (!sheetTobacco || !sheetCust || !sheetDecayRule) {
            alert('没有找到名为 "烟草参数表" 、"客户参数表"和"衰减速率参数表"的工作表')
            reject('没有找到工作表')
            return
          }
          /**********烟草参数表******** */

          // 使用 aoa_to_sheet 方法将工作表转化为数据数组
          const jsonDataTobacco = XLSX.utils.sheet_to_json(sheetTobacco, { header: 1 }) // header: 1 表示第一行是表头
          // 初始化一个空数组，存储处理后的数据对象
          this.uploadArrayTobacco = []
          // 获取表头数据
          const headersTobacco = jsonDataTobacco[0] // 第一行是表头
          //表头中文映射
          const headerMapTobacco = {
            编号: 'sku',
            卷烟品名: 'name',
            周销量计划: 'weekSalesNum',
            本轮订足率: 'fixRate',
            商业库存: 'businessStock',
            累计社会存销比: 'stockSaleRate',
            动销率: 'fixSaleRate',
            社会库存: 'societyStock',
            累计社会销量: 'societySales',
            累计社会销量日期: 'societySalesDate'
          }
          // 从第二行开始处理每一行数据

          this.serialId = jsonDataTobacco[1][9]
          const serialId = this.serialId.toString()
          this.serialIdNew = this.serialId.toString()
          const dateResult = this.getWeekRangeFromString(serialId)
          const periodValue = dateResult.monday + '至' + dateResult.sunday + '卷烟拟合数据'
          this.period = {
            serialId: serialId,
            periodName: periodValue
          }

          for (let i = 1; i < jsonDataTobacco.length; i++) {
            const row = jsonDataTobacco[i]
            // 创建一个对象，将每列的数据与表头对应起来
            const rowObject = headersTobacco.reduce((acc, header, index) => {
              const tobaccoHeader = headerMapTobacco[header] || header // 使用映射表查找英文字段名
              acc[tobaccoHeader] = row[index] // 将每一列的值映射到对应的英文字段名
              return acc
            }, {})
            this.uploadArrayTobacco.push(rowObject)
          }

          /**********客户参数表******** */
          // 使用 aoa_to_sheet 方法将工作表转化为数据数组
          const jsonDataCust = XLSX.utils.sheet_to_json(sheetCust, { header: 1 }) // header: 1 表示第一行是表头
          // 初始化一个空数组，存储处理后的数据对象
          this.uploadArrayCust = []
          // 获取表头数据
          const headersCust = jsonDataCust[0] // 第一行是表头

          //表头中文映射
          const headerMapCust = {
            档位: 'gear',
            客户数: 'num'
          }
          // 从第二行开始处理每一行数据
          for (let i = 1; i < jsonDataCust.length; i++) {
            const row = jsonDataCust[i]
            // 创建一个对象，将每列的数据与表头对应起来
            const rowObject = headersCust.reduce((acc, header, index) => {
              const custHeader = headerMapCust[header] || header // 使用映射表查找英文字段名
              acc[custHeader] = row[index] // 将每一列的值映射到对应的英文字段名
              return acc
            }, {})
            this.uploadArrayCust.push(rowObject)
          }

          /*********衰减速率参数表******** */

          // 使用 aoa_to_sheet 方法将工作表转化为数据数组
          const jsonDataDecayRule = XLSX.utils.sheet_to_json(sheetDecayRule, { header: 1 }) // header: 1 表示第一行是表头
          // 初始化一个空数组，存储处理后的数据对象
          this.uploadArrayDecayRule = []
          // 获取表头数据
          const headersDecayRule = jsonDataDecayRule[0] // 第一行是表头
          //表头中文映射
          const headerMapDecayRule = {
            '可供量（箱）': 'stocks',
            '30档定量': 'lev30Num',
            衰减速度: 'decayRate',
            备注: 'remark'
          }
          // 从第二行开始处理每一行数据
          for (let i = 1; i < jsonDataDecayRule.length; i++) {
            const row = jsonDataDecayRule[i]
            // 创建一个对象，将每列的数据与表头对应起来
            const rowObject = headersDecayRule.reduce((acc, header, index) => {
              const decayRuleHeader = headerMapDecayRule[header] || header // 使用映射表查找英文字段名
              acc[decayRuleHeader] = row[index] // 将每一列的值映射到对应的英文字段名
              return acc
            }, {})
            this.uploadArrayDecayRule.push(rowObject)
          }

          resolve() // 确保数据已更新
        }
        reader.onerror = (err) => {
          reject(err)
        }
        // 读取文件
        reader.readAsArrayBuffer(file)
      })
    },

    //下载参数模板
    async downloadTemplateParam() {
      try {
        //卷烟参数表
        const payloadTobacco = {}
        const responseTobacco = await downloadTobacco(payloadTobacco)
        //客户参数表
        const payloadCust = {}
        const responseCust = await downloadCust(payloadCust)

        //衰减速率参数表
        const payloadDecayRule = {}
        const responseDecayRule = await downloadDecayRule(payloadDecayRule)

        if (
          responseTobacco.data.code == 200 &&
          responseCust.data.code == 200 &&
          responseDecayRule.data.code == 200
        ) {
          // 烟草参数
          this.downTobaccoArray = responseTobacco.data.data.list || [] // Ensure it's an array
          const dataTobacco = []
          dataTobacco.push([
            '编号',
            '卷烟品名',
            '周销售计划',
            '本轮定足率',
            '商业库存',
            '累计社会存销比',
            '动销率',
            '社会库存',
            '累计社会销量',
            '累计社会销量日期'
          ])
          this.downTobaccoArray.forEach((result) => {
            dataTobacco.push([
              result.sku,
              result.name,
              result.weekSalesNum.toString(),
              result.fixRate.toString(),
              result.businessStock.toString(),
              result.stockSaleRate.toString(),
              result.fixSaleRate.toString(),
              result.societyStock.toString(),
              result.societySales.toString(),
              result.societySalesDate.toString()
            ])
          })

          // 客户参数
          this.downCustArray = responseCust.data.data.list || [] // Ensure it's an array
          const dataCust = []
          dataCust.push(['档位', '客户数'])
          this.downCustArray.forEach((result) => {
            dataCust.push([result.gear, result.num.toString()])
          })

          // 衰减速率参数
          this.downDecayRuleArray = responseDecayRule.data.data.list || [] // Ensure it's an array
          const dataDecayRule = []
          dataDecayRule.push(['可供量（箱）', '30档定量', '衰减速度', '备注'])
          this.downDecayRuleArray.forEach((result) => {
            dataDecayRule.push([
              result.stocks,
              result.lev30Num.toString(),
              result.decayRate.toString(),
              result.remark.toString()
            ])
          })
          const wb = XLSX.utils.book_new()

          const wsTobacco = XLSX.utils.aoa_to_sheet(dataTobacco) // Convert array of arrays to sheet
          const wsCust = XLSX.utils.aoa_to_sheet(dataCust) // Convert array of arrays to sheet
          const wsDecayRule = XLSX.utils.aoa_to_sheet(dataDecayRule) // Convert array of arrays to sheet

          XLSX.utils.book_append_sheet(wb, wsTobacco, '卷烟参数表') // Append data to Sheet2
          XLSX.utils.book_append_sheet(wb, wsCust, '客户参数表') // Append data to Sheet2
          XLSX.utils.book_append_sheet(wb, wsDecayRule, '衰减速率参数表') // Append data to Sheet2
          const excelBlob = new Blob([XLSX.write(wb, { bookType: 'xlsx', type: 'array' })], {
            type: 'application/octet-stream'
          })

          const link = document.createElement('a')
          link.href = URL.createObjectURL(excelBlob) // Create blob URL
          link.setAttribute('download', '投放参数表.xlsx') // Set file name
          document.body.appendChild(link)
          link.click() // Trigger download
          document.body.removeChild(link) // Clean up the link
          this.$message.success('下载成功!')
          this.dialogVisibleParam = false // Close dialog
        } else {
          this.$message.error('下载失败!')
        }
      } catch (error) {
        this.$message.error('下载失败')
      }
    },

    //按照挡位进行分配
    async calculateDistribution() {
      this.loading = true
      this.allocationResults = []
      ;(this.newAllocationResults = []),
        setTimeout(async () => {
          //查询所有的香烟
          const payloadTobacco = {}
          const responseTobacco = await queryTobacco(payloadTobacco) // 调用 upload 函数并传入 payload
          if (responseTobacco.data.code == 200) {
            this.skus = responseTobacco.data.data.list
            this.skus.forEach((item) => {
              item.stockOld = (item.weekSalesNum / item.fixRate).toFixed(2)
              item.stock = (parseInt(item.stockOld, 10) || 0) * 250
            })
          } else {
            this.$message.error(responseTobacco.data.data.message)
          }

          //查询所有的客户参数
          const payloadCust = {}
          const responseCust = await queryCust(payloadCust)
          if (responseCust.data.code == 200) {
            const customeArray = responseCust.data.data.list
            this.customers = []
            customeArray.forEach((item) => {
              this.customers.push(item.num)
            })
          } else {
            this.$message.error(responseCust.data.data.message)
          }

          //查询所有的速率数据
          const payloadDecayRule = {}
          const responseDecayRule = await queryDecayRules(payloadDecayRule)
          if (responseDecayRule.data.code == 200) {
            this.decayRuleArray = responseDecayRule.data.data.list
          } else {
            this.$message.error(responseDecayRule.data.data.message)
          }

          const payloadPeriod = {}
          const responsePeriod = await queryPeriod(payloadPeriod)

          if (responsePeriod.data.code == 200) {
            const periodArray = responsePeriod.data.data.list
            this.serialIdNew = periodArray.reduce((max, current) => {
              return parseInt(current.serialId) > parseInt(max.serialId) ? current : max
            }).serialId
          } else {
            this.$message.error(responsePeriod.data.data.message)
          }
          //得到总客户数
          const totalCustomers = this.customers.reduce((sum, item) => {
            return sum + item.num // 累加每个 item 的 num 值
          }, 0)
          this.allocationResults = []
          this.skus.forEach((sku) => {
            // 烟的条数
            // 初始化数组含30个0
            this.allocations = Array(this.customerLevels).fill(0)
            let remainingStock = sku.stock
            const remainingStockBox = sku.stock / 250 //可供量（箱数）

            const entry = this.decayRuleArray.find(
              (item) => remainingStockBox >= item.stocks[0] && remainingStockBox <= item.stocks[1]
            )

            this.Reduceratio = 1 - entry.decayRate

            this.stock30 = entry.lev30Num
            this.stock1 = 0
            if (this.stock30 >= this.formData.ratio) {
              this.stock1 = Math.round(this.stock30 / this.formData.ratio)
            }
            for (let level = this.customerLevels - 1; level >= 0 && remainingStock > 0; level--) {
              //从高到低每一档的客户数
              const levelCustomers = this.customers[level] //客户数
              if (level == 29) {
                this.allocations[level] = this.stock30
                remainingStock -= this.allocations[level] * levelCustomers
              } else {
                if (this.stock1 > 0) {
                  //得到当前档位之后的数组
                  const customersWithout = this.customers.filter((_, index) => index < level + 1)
                  // 计算剩下档位的客户数依次乘以固定值后相加
                  const totalSum = customersWithout.reduce(
                    (sum, customer) => sum + customer * this.stock1,
                    0
                  )
                  //当前档位总数乘和大于成剩余的档位数，重复循坏，除非等于最小档数，退出循环
                  if (totalSum >= remainingStock) {
                    remainingStock -= this.allocations[level] * levelCustomers
                    for (let i = level; i >= 0; i--) {
                      this.allocations[i] = this.stock1 // 赋值为固定值
                    }
                    break // 退出循环
                  } else {
                    for (let j = 1; j <= 999; j++) {
                      this.allocations[level] = Math.round(
                        this.allocations[29] * (this.Reduceratio ** (29 - level)) ** j
                      )
                      const remainNext =
                        remainingStock - this.allocations[level] * this.customers[level]
                      const customersWithoutNext = this.customers.filter(
                        (_, index) => index < level
                      )
                      // 计算剩下的客户数依次乘以固定值后相加
                      const totalSumNext = customersWithoutNext.reduce(
                        (sum, customer) => sum + customer * this.stock1,
                        0
                      )
                      //分配完后剩余档数和最小条数乘和大于烟的剩余，重复循坏，除非等于最小档数，退出循环
                      if (totalSumNext >= remainNext) {
                        if (this.allocations[level] <= this.stock1) {
                          remainingStock -= this.allocations[level] * levelCustomers
                          for (let i = level; i >= 0; i--) {
                            this.allocations[i] = this.stock1 // 赋值为固定值
                          }
                          break
                        }
                      } else {
                        if (this.allocations[level] < this.stock1) {
                          this.allocations[level] = this.stock1
                        }
                        remainingStock -= this.allocations[level] * levelCustomers
                        break
                      }
                    }
                  }
                } else {
                  if (remainingStock > 0) {
                    this.allocations[level] = Math.round(
                      this.allocations[29] * this.Reduceratio ** (29 - level)
                    )
                    remainingStock -= this.allocations[level] * levelCustomers
                  } else {
                    for (let i = level; i >= 0; i--) {
                      this.allocations[i] = this.stock1 // 赋值为固定值
                    }
                  }
                }
              }
            }
            this.totalSumFinal = this.allocations.reduce((sum, value, index) => {
              return sum + value * this.customers[index] // 相乘并累加
            }, 0)
            if (sku.sku != '') {
              this.allocationResults.push({
                sku: sku.sku,
                stockOld: sku.stockOld,
                stock: sku.stock,
                name: sku.name,
                isSale: this.allocations.every((allocation) => allocation === 0) ? '否' : '是',
                allocations: this.allocations,
                remaining: (this.totalSumFinal / 250).toFixed(2)
              })

              this.newAllocationResults.push({
                serialId: this.serialIdNew,
                sku: sku.sku,
                cigName: sku.name,
                stocksSale: sku.stockOld,
                isSale: this.allocations.every((allocation) => allocation === 0) ? '否' : '是',
                allocations: this.allocations,
                stocksPre: parseFloat((this.totalSumFinal / 250).toFixed(2))
              })
            }
          })
          this.allocationResults.forEach((item) => {
            item.allocations = item.allocations.slice().reverse()
          })

          //保存所有的拟合数据
          const payloadCaculation = {
            distributions: this.newAllocationResults
          }
          const responseCaculation = await saveCalculateDistribution(payloadCaculation)
          if (responseCaculation.data.code == 200) {
            this.$message.success('数据加载成功！')
          } else {
            this.$message.error('数据加载失败！')
            this.allocationResults = []
          }
          this.loading = false
          this.$forceUpdate()
        }, 1000)
    },

    //导出分配结果
    exportToCSV() {
      let csv =
        '编码,名称,可供量,是否投放,预测值,' +
        Array.from({ length: this.customerLevels }, (_, i) => `${30 - i}档`).join(',') + // 从 30 开始递减到 1
        '\n' // 继续添加表头，最后添加换行符
      this.allocationResults.forEach((result) => {
        csv += `${result.sku},${result.name},${result.stockOld},${result.isSale},${result.remaining},${result.allocations.join(',')}\n`
      })
      const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.setAttribute('download', '档位投放表.csv')
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
    },

    getWeekRangeFromString(dateStr) {
      // 1. 将字符串转换为日期对象
      const inputDateString = `${this.societySalesDate.slice(0, 4)}-${this.societySalesDate.slice(4, 6)}-${this.societySalesDate.slice(6, 8)}`

      // 将输入的日期字符串 (yyyyMMdd) 转换为 Date 对象
      const givenDate = new Date(
        dateStr.substring(0, 4),
        dateStr.substring(4, 6) - 1,
        dateStr.substring(6, 8)
      )

      // 获取当前日期的星期几 (0-6), 0是周日，1是周一，...，6是周六
      const dayOfWeek = givenDate.getDay()

      // 计算周一的日期 (通过减去当前日期的星期数，调整到周一)
      const monday = new Date(givenDate)
      monday.setDate(givenDate.getDate() - dayOfWeek + 1) // 周一是当前日期减去星期几的天数再加1

      // 计算周日的日期 (通过加上周日剩余的天数)
      const sunday = new Date(givenDate)
      sunday.setDate(givenDate.getDate() - dayOfWeek + 7) // 周日是当前日期减去星期几的天数再加7

      // 返回格式化后的周一和周日
      return {
        monday: this.formatDate(monday),
        sunday: this.formatDate(sunday)
      }
    },

    // 格式化日期为 YYYY-MM-DD
    formatDate(date) {
      const year = date.getFullYear()
      const month = String(date.getMonth() + 1).padStart(2, '0') // 月份从 0 开始
      const day = String(date.getDate()).padStart(2, '0') // 保证是两位数

      return `${year}-${month}-${day}`
    }
  }
}
</script>

<style scoped>
.container {
  margin: 5px;
}

table {
  width: 100%;
  margin-bottom: 10px;
  border-collapse: collapse;
}

table,
th,
td {
  border: 1px solid black;
}

th,
td {
  padding: 5px;
  text-align: center;
}

.el-table thead th {
  font-weight: 600;
  color: #000;
}

.custom-form-item::v-deep.el-form-item__label {
  font-size: 18px;
  color: #000 !important;
}

.el-table__header-wrapper {
  width: 100%;
  height: 40px;
  overflow: hidden;
}

.el-dialog__header {
  height: 54px;
  padding: 0;
  margin-right: 0 !important;
  text-align: center;
  border-bottom: 1px solid var(--el-border-color);
}
</style>
