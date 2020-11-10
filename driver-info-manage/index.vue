<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="司机编号" prop="driverId">
        <el-input v-model="formData.driverId"></el-input>
      </el-form-item>
      <el-form-item label="司机名称" prop="driverName">
        <el-input v-model="formData.driverName"></el-input>
      </el-form-item>
      <el-form-item label="所在省份" prop="driverProvinceId" >
          <el-select v-model="formData.driverProvinceId" @focus="funProvinceId">
           <el-option
           v-for="(item, index) in province"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
         <el-form-item label="所在城市" prop="driverCityId" >
          <el-select v-model="formData.driverCityId" @focus="funCityId(formData.driverProvinceId)">
           <el-option
           v-for="(item, index) in city"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
          <el-form-item label="所在地区" prop="driverAreaId" >
          <el-select v-model="formData.driverAreaId" @focus="funAreaId(formData.driverCityId)">
           <el-option
           v-for="(item, index) in area"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
    </i-search>

    <i-table
     v-if="this.store === '0'"
    :toolbar="toolbar"
    :tableData="tableData"
    :pageInfo="pageInfo"
    :selectionShow='true'
    @handleSizeChange="handleSizeChange"
    @handleCurrentChange="handleCurrentChange"
    @handleSelectionChange='handleSelectionChange'>
      <el-table-column
      v-for="(item, index) in columnList"
      :key='index'
      :label="item.label"
      :prop="item.prop"
      :show-overflow-tooltip="true"
      align="center"
      >
      </el-table-column>
    </i-table>
    <i-table
    v-if="this.store === '1' || this.store === '3'"
    :toolbar="othertoolbar"
    :tableData="tableData"
    :pageInfo="pageInfo"
    :selectionShow='true'
    @handleSizeChange="handleSizeChange"
    @handleCurrentChange="handleCurrentChange"
    @handleSelectionChange='handleSelectionChange'>
      <el-table-column
      v-for="(item, index) in columnList"
      :key='index'
      :label="item.label"
      :prop="item.prop"
      :show-overflow-tooltip="true"
      align="center"
      >
      </el-table-column>
    </i-table>

    <i-dialog
    v-model="dialogvisible"
    :title='dialogTitle'
    @dialog-cancel="dialogCancel"
    @dialog-confirm="dialogConfirm"
    @dialog-before-close="dialogBeforeClose">
    <el-form ref="form" :model="dialogFormData" label-width="100px" :rules="formRules">
      <el-row >
        <el-col :span="12">
          <el-form-item label="司机名称" prop="driverName">
          <el-input v-model="dialogFormData.driverName"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="联系电话" prop="driverPhone">
          <el-input v-model.number="dialogFormData.driverPhone"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="司机账号" prop="driverAccount">
          <el-input v-model="dialogFormData.driverAccount"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="密码" prop="driverPassword">
          <el-input v-model="dialogFormData.driverPassword" type="password"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="身份证" prop="idCard">
          <el-input v-model.number="dialogFormData.idCard"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="所在省份" prop="driverProvinceId">
          <el-select v-model="dialogFormData.driverProvinceId" @focus="funProvinceId">
           <el-option
           v-for="(item, index) in province"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="所在城市" prop="driverCityId" >
          <el-select v-model="dialogFormData.driverCityId" @focus="funCityId(dialogFormData.driverProvinceId)">
           <el-option
           v-for="(item, index) in city"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="所在区" prop="driverAreaId" >
          <el-select v-model="dialogFormData.driverAreaId" @focus="funAreaId(dialogFormData.driverCityId)">
           <el-option
           v-for="(item, index) in area"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    </i-dialog>
  </div>
</template>
<script>
import ISearch from '../../components/common/i-search'
import ITable from '../../components/common/i-table'
import IDialog from '../../components/common/i-dialog'
import req from '@/api/driver-info-manage'
export default {
  name: 'driver-info-manage',
  components: {
    ISearch,
    ITable,
    IDialog
  },
  data () {
    return {
      tableSelectRows: [],
      province: [], // 省份
      city: [], // 城市
      area: [], // 省份
      formData: {},
      dialogtype: '',
      dialogvisible: false,
      dialogTitle: '新增门店',
      dialogFormData: {},

      driverProvinceId: '',
      driverCityId: '',
      driverAreaId: '',
      toolbar: [
        {
          btnName: '详情',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length < 1) {
              this.$message('未选择选项')
            } else if (this.tableSelectRows.length > 1) {
              this.$message.error('不能选择多于一条')
            } else {
              this.dialogvisible = true
              this.dialogtype = 'details'
              this.dialogTitle = '用户详情'
              req('findDriverById', {driverId: this.tableSelectRows[0].driverId}).then(data => {
                // console.log(data, '12321')
                this.dialogFormData = data.data
                this.driverCityId = this.dialogFormData.driverCityId
                this.driverProvinceId = this.dialogFormData.driverProvinceId
                this.driverAreaId = this.dialogFormData.driverAreaId
                this.dialogFormData.driverProvinceId = data.data.driverProvince
                this.dialogFormData.driverCityId = data.data.driverCity
                this.dialogFormData.driverAreaId = data.data.driverArea
              })
            }
          }
        },
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            this.dialogvisible = true
            this.dialogtype = 'create'
            this.dialogTitle = '新增司机'
          }
        },
        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length < 1) {
              this.$message('未选择选项')
            } else if (this.tableSelectRows.length > 1) {
              this.$message.error('不能选择多于一条')
            } else {
              this.dialogvisible = true
              this.dialogtype = 'uersup'
              this.dialogTitle = '修改司机信息'
              req('findDriverById', {driverId: this.tableSelectRows[0].driverId}).then(data => {
                this.dialogFormData = data.data
                this.driverCityId = this.dialogFormData.driverCityId
                this.driverProvinceId = this.dialogFormData.driverProvinceId
                this.driverAreaId = this.dialogFormData.driverAreaId
                // console.log(this.driverProvinceId, 'driverProvinceId')
                // console.log(this.driverCityId, 'driverCityId')
                // console.log(this.driverAreaId, 'driverAreaId')
                this.dialogFormData.driverProvinceId = data.data.driverProvince
                this.dialogFormData.driverCityId = data.data.driverCity
                this.dialogFormData.driverAreaId = data.data.driverArea
                // console.log(this.dialogFormData)
              })
            }
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows[0] === 0) {
              this.$message.info('请选择删除的数据')
              return
            }
            this.$confirm('是否确认删除').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.userId
              }).toString()
              req('deleteDriver', {userId: ids}).then(data => {
                if (data.code === 0) {
                  this.$message.success(data.msg)
                  this.fetch()
                } else {
                  this.$message.error(data.msg)
                }
              })
            }).catch(() => {
            })
          }
        }
      ],
      othertoolbar: [
        {
          btnName: '详情',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length < 1) {
              this.$message('未选择选项')
            } else if (this.tableSelectRows.length > 1) {
              this.$message.error('不能选择多于一条')
            } else {
              this.dialogvisible = true
              this.dialogtype = 'details'
              this.dialogTitle = '用户详情'
              req('findDriverById', {driverId: this.tableSelectRows[0].driverId}).then(data => {
                // console.log(data, '12321')
                this.dialogFormData = data.data
                this.driverCityId = this.dialogFormData.driverCityId
                this.driverProvinceId = this.dialogFormData.driverProvinceId
                this.driverAreaId = this.dialogFormData.driverAreaId
                this.dialogFormData.driverProvinceId = data.data.driverProvince
                this.dialogFormData.driverCityId = data.data.driverCity
                this.dialogFormData.driverAreaId = data.data.driverArea
              })
            }
          }
        }
      ],
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        total: 0
      },
      columnList: [
        {label: '司机编号', prop: 'driverId'},
        {label: '姓名', prop: 'driverName'},
        {label: '手机', prop: 'driverPhone'},
        {label: '身份证', prop: 'idCard'},
        {label: '账号', prop: 'driverAccount'}
      ],
      tableData: [],
      formRules: {
        driverName: [
          { required: true, message: '请输入名称', trigger: 'change' }
        ],
        driverPhone: [
          { required: true, message: '请输入手机号码', trigger: 'change' },
          {message: '手机号格式不对', pattern: /^0{0,1}(13[0-9]|15[7-9]|153|156|18[7-9])[0-9]{8}$/}
        ],
        driverAccount: [
          { required: true, message: '请输入司机账号', trigger: 'change' }
        ],
        driverPassword: [
          { required: true, message: '请输入密码', trigger: 'change' }
        ],
        idCard: [
          { required: true, message: '请输入身份证', trigger: 'change' },
          { validator: this.validateIdCard, trigger: 'change' }
        ],
        driverProvinceId: [
          { required: true, message: '请选择省份', trigger: 'change' }
        ],
        driverCityId: [
          { required: true, message: '请选择城市', trigger: 'change' }
        ],
        driverAreaId: [
          { required: true, message: '请选择地区', trigger: 'change' }
        ]
      }
    }
  },
  watch: {
    'dialogFormData.driverProvinceId': {
      handler (value) {
        if (!value) {
          this.dialogFormData.driverCityId = ''
          this.dialogFormData.driverAreaId = ''
        }
      }
    },
    'dialogFormData.driverCityId': {
      handler (value) {
        if (!value) {
          this.dialogFormData.driverAreaId = ''
        }
      }
    }
  },
  created () {
    this.fetch()
  },
  mounted () {
    this.pageInfo.total = this.tableData.length
  },
  computed: {
    store () {
      return JSON.parse(sessionStorage.getItem('storeInfo')).role
    }
  },
  methods: {
    fetch () {
      this.pageInfo.pageIndex = 1
      this.pageInfo.pageSize = 5
      this.search()
    },
    search () {
      // console.log('搜索')
      req('listDrivers', {...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        this.tableData = data.data.list
        // console.log(data)
        // 列表栏获取
        this.pageInfo.pageIndex = data.data.pageNum
        // // 分页获取
        this.pageInfo.pageSize = data.data.pageSize
        this.pageInfo.totalSize = data.data.total
        // console.log(data)
      })
    },
    validateIdCard (rule, value, callback) {
      let reg = /^[1-9]\d{5}(18|19|20)\d{2}((0[1-9])|(1[0-2]))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/
      if (!reg.test(value)) {
        callback(new Error('请输入正确身份证号'))
      } else {
        callback()
      }
    },
    reset () {
      // console.log('重置')
      this.fetch()
    },
    handleSizeChange (value) {
      // console.log('每页多少条', value)
      this.pageInfo.pageNum = 1
      this.pageInfo.pageSize = value
      this.search()
    },
    handleCurrentChange (value) {
      // console.log('当前多少页', value)
      this.pageInfo.pageIndex = value
      this.search()
    },
    handleSelectionChange (row) {
      this.tableSelectRows = row
    },
    dialogCancel () {
      this.$refs.form.resetFields()
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      this.$refs.form.resetFields()
    },
    dialogConfirm () {
      // console.log('dialog确定按钮点击')
      this.dialogFormData.driverPhone = this.dialogFormData.driverPhone.toString()
      this.dialogFormData.idCard = this.dialogFormData.idCard.toString()
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogtype === 'create') {
            req('addDriver', {...this.dialogFormData
            }).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg) // 发送成功
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
                // this.$refs.form.resetFields()
                // this.dialogvisible = false
              }
            })
          } else if (this.dialogtype === 'uersup') {
            // console.log(this.dialogFormData.driverProvinceId === this.dialogFormData.driverProvince, 'this.driverProvinceId, driverP--------------')
            if (this.dialogFormData.driverProvinceId === this.dialogFormData.driverProvince) {
              // console.log(this.driverProvinceId, 'driverP1')
              this.dialogFormData.driverProvinceId = this.driverProvinceId
            }
            if (this.dialogFormData.driverCityId === this.dialogFormData.driverCity) {
              this.dialogFormData.driverCityId = this.driverCityId
            }
            if (this.dialogFormData.driverAreaId === this.dialogFormData.driverArea) {
              this.dialogFormData.driverAreaId = this.driverAreaId
            }
            // console.log(this.dialogFormData, 'driverP')
            req('updateDriverById', {
              userId: this.tableSelectRows[0].userId,
              ...this.dialogFormData,
              version: this.tableSelectRows[0].version
            }).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error('修改失败')
                // this.$refs.form.resetFields()
                // this.dialogvisible = false
              }
            })
          } else {
            this.$refs.form.resetFields()
            this.dialogvisible = false
          }
        } else {
          return false
        }
      })
    },
    // 连续省份查找
    funProvinceId () {
      // console.log('省份')
      req('queryStoreLocalClassify', {lastClassifyId: 0}).then(data => {
        this.province = data.data
        if (this.dialogFormData.driverCityId || this.dialogFormData.driverAreaId) {
          this.dialogFormData.driverCityId = null
          this.dialogFormData.driverAreaId = null
        }
      })
    },
    funCityId (value) {
      // console.log('城市', value)
      if (this.dialogFormData.driverProvince === value) {
        req('queryStoreLocalClassify', {lastClassifyId: this.driverProvinceId}).then(data => {
          this.city = data.data
          if (this.dialogFormData.driverAreaId) {
            this.dialogFormData.driverAreaId = null
          }
        })
      } else {
        req('queryStoreLocalClassify', {lastClassifyId: value}).then(data => {
          this.city = data.data
          if (this.dialogFormData.driverAreaId) {
            this.dialogFormData.driverAreaId = null
          }
        })
      }
    },
    funAreaId (value) {
      // console.log('地区', value)
      if (this.dialogFormData.driverCity === value) {
        req('queryStoreLocalClassify', {lastClassifyId: this.driverCityId}).then(data => {
          this.area = data.data
        })
      } else {
        req('queryStoreLocalClassify', {lastClassifyId: value}).then(data => {
          this.area = data.data
        })
      }
    }
  }
}
</script>
<style lang="scss" scoped>
.el-dialog{
  // background-color: #aaa;
  .el-form{
    .el-row{
      .el-form-item{
       /deep/ .el-form-item__label{
          font-size: 13px;
        }
      }
    }
  }
}
.i-search {
  .el-form{
    .el-form-item{
      margin-right: 2%;
      /deep/ .el-form-item__content {
        width: 221.4px;
      }
       /deep/ .el-form-item__label {
        font-size: 13px;
      }
    }
  }
}

</style>
