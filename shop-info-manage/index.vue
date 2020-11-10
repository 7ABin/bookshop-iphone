<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="门店编号" prop="storeId">
        <el-input v-model="formData.storeId"></el-input>
      </el-form-item>
      <el-form-item label="门店名称" prop="storeName">
        <el-input v-model="formData.storeName"></el-input>
      </el-form-item>
      <el-form-item label="店长名称" prop="storeManagerName">
        <el-input v-model="formData.storeManagerName"></el-input>
      </el-form-item>
      <el-form-item label="所在省份" prop="storeProvinceId" >
          <el-select v-model="formData.storeProvinceId" @focus="funProvinceId">
           <el-option
           v-for="(item, index) in province"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
         <el-form-item label="所在城市" prop="storeCityId">
          <el-select v-model="formData.storeCityId"  @focus="funCityId(formData.storeProvinceId)">
           <el-option
           v-for="(item, index) in city"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
          <el-form-item label="所在地区" prop="storeAreaId" >
          <el-select v-model="formData.storeAreaId" @focus="funAreaId(formData.storeCityId)">
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
      align="center"
      :show-overflow-tooltip="true"
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
      align="center"
      :show-overflow-tooltip="true"
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
          <el-form-item label="门店名称" prop="storeName">
          <el-input v-model="dialogFormData.storeName"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="联系电话" prop="storePhone">
          <el-input v-model.number="dialogFormData.storePhone"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="店长编号" prop="storeManagerId">
          <el-input v-model="dialogFormData.storeManagerId"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="营业执照编码" prop="bussinessCode">
          <el-input v-model.number="dialogFormData.bussinessCode"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="所在省份" prop="storeProvinceId">
          <el-select v-model="dialogFormData.storeProvinceId" @focus="funProvinceId">
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
           <el-form-item label="所在城市" prop="storeCityId">
          <el-select v-model="dialogFormData.storeCityId" @focus="funCityId(dialogFormData.storeProvinceId)">
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
           <el-form-item label="所在地区" prop="storeAreaId">
          <el-select v-model="dialogFormData.storeAreaId" @focus="funAreaId(dialogFormData.storeCityId)">
           <el-option
           v-for="(item, index) in area"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
        <el-col :span="12" width='50%'>
        <el-form-item label="详细地址" prop="storeAddress">
       <el-input type="textarea" v-model="dialogFormData.storeAddress"></el-input>
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
import req from '@/api/shop-info-manage'
export default {
  name: 'user-manage',
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
      storeProvinceId: '',
      storeCityId: '',
      storeAreaId: '',
      areaId: '',
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
              this.dialogTitle = '门店详情'
              req('findStoreById', {storeId: this.tableSelectRows[0].storeId}).then(data => {
                this.dialogFormData = data.data
                this.storeCityId = this.dialogFormData.storeCityId
                this.storeProvinceId = this.dialogFormData.storeProvinceId
                this.areaId = this.dialogFormData.areaId
                // console.log('storeProvinceId', this.storeProvinceId)
                // console.log('storeCityId', this.storeCityId)
                // console.log('areaId', this.areaId)
                this.dialogFormData.storeProvinceId = data.data.storeProvince
                this.dialogFormData.storeCityId = data.data.storeCity
                this.dialogFormData.storeAreaId = data.data.storeArea
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
            this.dialogTitle = '新增门店'
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
              this.dialogtype = 'getup'
              this.dialogTitle = '修改门店信息'
              req('findStoreById', {storeId: this.tableSelectRows[0].storeId}).then(data => {
                this.dialogFormData = data.data
                this.storeCityId = this.dialogFormData.storeCityId
                this.storeProvinceId = this.dialogFormData.storeProvinceId
                this.areaId = this.dialogFormData.storeAreaId
                // console.log('storeProvinceId', this.dialogFormData)
                this.dialogFormData.storeProvinceId = data.data.storeProvince
                this.dialogFormData.storeCityId = data.data.storeCity
                this.dialogFormData.storeAreaId = data.data.storeArea
              })
            }
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择删除的数据')
              return
            }
            this.$confirm('是否确认删除').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.storeId
              }).toString()
              req('deleteStore', {storeId: ids}).then(data => {
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
              this.dialogTitle = '门店详情'
              req('findStoreById', {storeId: this.tableSelectRows[0].storeId}).then(data => {
                this.dialogFormData = data.data
                this.storeCityId = this.dialogFormData.storeCityId
                this.storeProvinceId = this.dialogFormData.storeProvinceId
                this.areaId = this.dialogFormData.areaId
                // console.log('storeProvinceId', this.storeProvinceId)
                // console.log('storeCityId', this.storeCityId)
                // console.log('areaId', this.areaId)
                this.dialogFormData.storeProvinceId = data.data.storeProvince
                this.dialogFormData.storeCityId = data.data.storeCity
                this.dialogFormData.storeAreaId = data.data.storeArea
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
        {label: '门店编号', prop: 'storeId'},
        {label: '门店名称', prop: 'storeName'},
        {label: '电话', prop: 'storePhone'},
        {label: '详细地址', prop: 'storeAddress'},
        {label: '店长姓名', prop: 'storeManagerName'},
        {label: '邀请码', prop: 'inviteCode'},
        {label: '门店账号', prop: 'storeAccount'}
      ],
      tableData: [],
      formRules: {
        storeName: [
          { required: true, message: '请输入名称', trigger: 'blur' }
        ],
        storePhone: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          {message: '手机号格式不对', pattern: /^0{0,1}(13[0-9]|15[7-9]|153|156|18[7-9])[0-9]{8}$/}
        ],
        storeManagerId: [
          { required: true, message: '请输入店长编号', trigger: 'blur' }
        ],
        bussinessCode: [
          { required: true, message: '请输入营业执照', trigger: 'blur' }
        ],
        storeProvinceId: [
          { required: true, message: '请选择省份', trigger: 'change' }
        ],
        storeCityId: [
          { required: true, message: '请选择城市', trigger: 'change' }
        ],
        storeAreaId: [
          { required: true, message: '请选择地区', trigger: 'change' }
        ],
        storeAddress: [
          { required: true, message: '请填写详情', trigger: 'blur' }
        ]
      }
    }
  },
  mounted () {
    this.pageInfo.total = this.tableData.length
  },
  created () {
    this.fetch()
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
      req('listStores', {...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        // console.log('搜索', data)
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
      this.dialogFormData.storePhone = this.dialogFormData.storePhone.toString()
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogtype === 'create') {
            // console.log(typeof this.dialogFormData.storePhone, '修改类型查看')
            req('addStore', {...this.dialogFormData}).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg) // 发送成功
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
                // this.dialogvisible = false
              }
            })
          } else if (this.dialogtype === 'getup') {
            // console.log(typeof this.dialogFormData.storePhone, '修改类型查看')
            if (this.dialogFormData.storeProvinceId === this.dialogFormData.storeProvince) {
              this.dialogFormData.storeProvinceId = this.storeProvinceId
            }
            if (this.dialogFormData.storeCityId === this.dialogFormData.storeCity) {
              this.dialogFormData.storeCityId = this.storeCityId
            }
            if (this.dialogFormData.storeAreaId === this.dialogFormData.storeArea) {
              this.dialogFormData.storeAreaId = this.areaId
              // console.log('2222')
            }
            // console.log(typeof this.dialogFormData.storePhone, '修改类型查看')
            req('updateStoreById', {
              storeId: this.tableSelectRows[0].storeId,
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
        if (this.dialogFormData.storeCityId || this.dialogFormData.storeAreaId) {
          this.dialogFormData.storeCityId = null
          this.dialogFormData.storeAreaId = null
        }
      })
    },
    funCityId (value) {
      // console.log('城市', value)
      if (this.dialogFormData.storeProvince === value) {
        req('queryStoreLocalClassify', {lastClassifyId: this.storeProvinceId}).then(data => {
          this.city = data.data
          if (this.dialogFormData.storeAreaId) {
            this.dialogFormData.storeAreaId = null
          }
        })
      } else {
        req('queryStoreLocalClassify', {lastClassifyId: value}).then(data => {
          this.city = data.data
          if (this.dialogFormData.storeAreaId) {
            this.dialogFormData.storeAreaId = null
          }
        })
      }
    },
    funAreaId (value) {
      // console.log('地区', value)
      if (this.dialogFormData.storeCity === value) {
        // console.log('111111111地区', this.storeCityId)
        req('queryStoreLocalClassify', {lastClassifyId: this.storeCityId}).then(data => {
          this.area = data.data
          // console.log(this.area, '地区')
        })
      } else {
        req('queryStoreLocalClassify', {lastClassifyId: value}).then(data => {
          this.area = data.data
          // console.log(this.area)
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
