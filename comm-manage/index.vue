<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData" size="medium">
      <el-form-item label="商品名称" label-width="70px" prop="goodsName">
        <el-input v-model="formData.goodsName"></el-input>
      </el-form-item>
      <el-form-item label="商品状态" label-width="70px" prop="goodsState">
    <el-select v-model="formData.goodsState">
      <el-option label="未发布" value="0"></el-option>
      <el-option label="在售" value="1"></el-option>
      <el-option label="已下架" value="2"></el-option>
    </el-select>
     </el-form-item>
      <el-form-item label="广告词" label-width="70px" prop="goodsAdvertise">
        <el-input v-model="formData.goodsAdvertise"></el-input>
      </el-form-item>
      <el-form-item label="出版社" label-width="70px" prop="goodsPress">
        <el-input v-model="formData.goodsPress"></el-input>
      </el-form-item>
      <el-form-item label="作者" label-width="70px" prop="goodsAuthor">
        <el-input v-model="formData.goodsAuthor"></el-input>
      </el-form-item>
    </i-search>

    <i-table
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
      :width="item.width"
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
          <el-form-item label="商品名称" prop="goodsName">
          <el-input v-model="dialogFormData.goodsName"></el-input>
          </el-form-item>
        </el-col>
         <el-col :span="12">
          <el-form-item label="书号" prop="goodsIsbnNumber">
          <el-input v-model="dialogFormData.goodsIsbnNumber"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="一级分类" prop="goodsFirstClassifyId">
          <el-select v-model="dialogFormData.goodsFirstClassifyId" @focus="funfirst">
           <el-option
           v-for="(item, index) in bookFirst"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
         <el-col :span="12">
           <el-form-item label="二级分类" prop="goodsSecondClassifyId">
          <el-select v-model="dialogFormData.goodsSecondClassifyId" @focus="funSecond(dialogFormData.goodsFirstClassifyId)">
           <el-option
           v-for="(item, index) in bookSecond"
           :key="index"
           :label="item.classifyName"
           :value="item.classifyId"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
          <el-col :span="12">
          <el-form-item label="广告词" prop="goodsAdvertise">
          <el-input v-model="dialogFormData.goodsAdvertise" type="textarea"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="商品介绍" prop="goodsProduce">
          <el-input  v-model="dialogFormData.goodsProduce" type="textarea"></el-input>
          </el-form-item>
        </el-col>
          <el-col :span="12">
           <el-form-item label="商家名称" prop="goodsMerchantName">
             <el-input  v-model="dialogFormData.goodsMerchantName" ></el-input>
         </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="库存" prop="goodsRestNumber">
          <el-input  v-model.number="dialogFormData.goodsRestNumber"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="定价" prop="goodsFixMoney">
         <input class="amount-input" size="12" maxlength="10"  min="0.00"  type="number" step="0.01" v-model.number="dialogFormData.goodsFixMoney">
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="售价" prop="goodsSaleMoney">
         <input class="amount-input" size="12" maxlength="6" min="0.00" type="number" step="0.01" v-model.number="dialogFormData.goodsSaleMoney">
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="出版社" prop="goodsPress">
          <el-input  v-model="dialogFormData.goodsPress"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="作者" prop="goodsAuthor">
          <el-input  v-model="dialogFormData.goodsAuthor"></el-input>
          </el-form-item>
        </el-col>
          <el-col :span="12">
          <el-form-item label="图片" prop="goodsPhoto">
            <i-file v-model="dialogFormData.goodsPhoto"></i-file>
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
import req from '@/api/comm-manage'
import IFile from '@/components/common/i-file.vue'
// const qs = require('qs')
export default {
  name: 'comm-manage',
  components: {
    ISearch,
    ITable,
    IDialog,
    IFile
  },
  data () {
    return {
      message: {access_token: JSON.parse(sessionStorage.getItem('userInfo')).access_token},
      option: [
        {label: '管理员', value: 1},
        {label: '店长', value: 2}],
      bookFirst: [],
      lastClassifyId: '',
      dialogBuildName: '',
      bookSecond: [],
      fileList: [],
      formData: {},
      dialogvisible: false,
      dialogTitle: '新增商品',
      dialogFormData: {
        goodsName: '',
        goodsIsbnNumber: '',
        goodsFirstClassifyId: '',
        goodsSecondClassifyId: '',
        goodsAdvertise: '',
        goodsProduce: '',
        goodsMerchantName: '',
        goodsRestNumber: '',
        goodsFixMoney: '',
        goodsSaleMoney: '',
        goodsPress: '',
        goodsAuthor: '',
        goodsPhoto: ''
      },
      dialogFirstId: '',
      dialogSecondId: '',
      tableSelectRows: [],
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            // console.log(this.pageInfo)
            this.dialogvisible = true
            this.dialogBuildName = 'create'
            this.dialogTitle = '新增商品'
          }
        },
        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
            } else if (this.tableSelectRows.length > 1) {
              this.$message.info('不支持多数据修改')
            } else {
              this.dialogvisible = true
              this.dialogTitle = '修改商品数据'
              this.dialogBuildName = 'userup'
              req('findGoodsById', {goodsId: this.tableSelectRows[0].goodsId}).then(data => {
                this.dialogFormData = data.data.data
                this.dialogFirstId = this.dialogFormData.goodsFirstClassifyId
                this.dialogSecondId = this.dialogFormData.goodsSecondClassifyId
                this.dialogFormData.goodsFirstClassifyId = this.dialogFormData.goodsFirstClassifyName
                this.dialogFormData.goodsSecondClassifyId = this.dialogFormData.goodsSecondClassifyName
                // console.log('类型', typeof this.dialogFormData.goodsFixMoney)
              })
            }
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            this.$confirm('是否确认删除').then(() => {
              if (this.tableSelectRows[0] === 0) {
                this.$message.info('请选择删除的数据')
                return
              }
              let ids = this.tableSelectRows.map(item => {
                return item.goodsId
              }).toString()
              req('deleteGood', {goodsId: ids}).then(data => {
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
        },
        {
          btnName: '上架',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
            } else {
              this.$confirm('是否确定上架').then(() => {
                let ids = this.tableSelectRows.map(item => {
                  return item.goodsId
                }).toString()
                req('updateGoodState', {goodsId: ids, goodsState: '1'}).then(data => {
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
        },
        {
          btnName: '下架',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
            } else {
              this.$confirm('是否确定下架').then(() => {
                let ids = this.tableSelectRows.map(item => {
                  return item.goodsId
                }).toString()
                req('updateGoodState', {goodsId: ids, goodsState: '2'}).then(data => {
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
        }
      ],
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        total: 0
      },
      columnList: [
        {label: '商品名称', prop: 'goodsName', width: 150},
        {label: '定价', prop: 'goodsFixMoney'},
        {label: '售价', prop: 'goodsSaleMoney'},
        {label: '销售量', prop: 'goodsSaleNumber'},
        {label: '一级分类', prop: 'goodsFirstClassifyName'},
        {label: '二级分类', prop: 'goodsSecondClassifyName'},
        {label: '广告词', prop: 'goodsAdvertise', width: 200},
        {label: '商品介绍', prop: 'goodsProduce', width: 200},
        {label: '商品状态', prop: 'goodsState'},
        {label: '上架时间', prop: 'goodsShelfTime', width: 200},
        {label: '浏览量', prop: 'goodsLookNumber'},
        {label: '商家名称', prop: 'goodsMerchantName', width: 200},
        {label: '库存', prop: 'goodsRestNumber'},
        {label: 'isbn书号', prop: 'goodsIsbnNumber', width: 200},
        {label: '出版社', prop: 'goodsPress'},
        {label: '作者', prop: 'goodsAuthor'}
      ],
      tableData: [],
      formRules: {
        goodsName: [
          { required: true, message: '请填写商品名称', trigger: 'change' }
        ],
        goodsIsbnNumber: [
          { required: true, message: '请选择书籍号', trigger: 'change' }
        ],
        goodsFirstClassifyId: [
          { required: true, message: '请选择一级分类', trigger: 'change' }
        ],
        goodsSecondClassifyId: [
          { required: true, message: '请选择二级分类', trigger: 'change' }
        ],
        goodsMerchantName: [
          { required: true, message: '请输入商家名称', trigger: 'change' }
        ],
        goodsRestNumber: [
          { required: true, message: '请输入库存量', trigger: 'change' }
        ],
        goodsFixMoney: [
          { required: true, message: '请输入成本价', trigger: 'change' },
          { type: 'number', message: '请输入数字', trigger: 'blur' }
        ],
        goodsSaleMoney: [
          { required: true, message: '请输入在售价', trigger: 'change' },
          { type: 'number', message: '请输入数字', trigger: 'blur' }
        ],
        goodsPress: [
          { required: true, message: '请输入出版社名', trigger: 'change' }
        ],
        goodsAuthor: [
          { required: true, message: '请作者名', trigger: 'change' }
        ],
        goodsPhoto: [
          { required: true, message: '请上传图片', trigger: 'change' }
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
  methods: {
    fetch () {
      this.pageInfo.pageIndex = 1
      this.pageInfo.pageSize = 5
      this.search()
    },
    search () {
      req('listGoods', {
        ...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        // console.log('listGoods', data)
        if (data.code === 0) {
          this.tableData = data.data.list
          // console.log(data)
          // 列表栏获取
          this.pageInfo.pageIndex = data.data.pageNum
          // // 分页获取
          this.pageInfo.pageSize = data.data.pageSize
          this.pageInfo.totalSize = data.data.total
          // console.log(this.tableData.length)
          for (let i = 0; i < this.tableData.length; i++) {
            if (this.tableData[i].goodsState === '0') {
              this.tableData[i].goodsState = '未发布'
            }
            if (this.tableData[i].goodsState === '1') {
              this.tableData[i].goodsState = '在售'
            }
            if (this.tableData[i].goodsState === '2') {
              this.tableData[i].goodsState = '下架'
            }
          }
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    // 一二级书籍
    funfirst () {
      // window.alert('aaaaa')
      req('queryGoodClassify', {lastClassifyId: '0'}).then(data => {
        // console.log('一级查找', data)
        this.bookFirst = data.data
        if (this.dialogFormData.goodsSecondClassifyId) {
          this.dialogFormData.goodsSecondClassifyId = null
        }
      })
    },
    funSecond (value) {
      // console.log('2级查找', value)
      if (this.dialogFormData.goodsFirstClassifyName === value) {
        req('queryGoodClassify', {lastClassifyId: this.dialogFirstId}).then(data => {
          this.bookSecond = data.data
        })
      } else {
        req('queryGoodClassify', {lastClassifyId: value}).then(data => {
          // console.log('2ji', data)
          this.bookSecond = data.data
        })
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
    handleSelectionChange (value) {
      // console.log(value)
      this.tableSelectRows = value
    },
    dialogCancel () {
      // this.$refs.upload.clearFiles()
      this.$refs.form.resetFields()
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      // this.$refs.upload.clearFiles()
      this.$refs.form.resetFields()
    },
    dialogConfirm () {
      // 按确定按钮时候
      this.dialogFormData.goodsRestNumber = this.dialogFormData.goodsRestNumber.toString()
      // console.log(this.dialogFormData.goodsRestNumber)
      this.dialogFormData.goodsFixMoney = Number(parseFloat(this.dialogFormData.goodsFixMoney).toFixed(2))
      this.dialogFormData.goodsSaleMoney = Number(parseFloat(this.dialogFormData.goodsSaleMoney).toFixed(2))
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogBuildName === 'create') {
            // console.log(...this.dialogFormData, 'this.dialogFormData')
            req('addGood', {...this.dialogFormData}).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.$refs.form.resetFields()
                this.fetch()
                // this.$refs.upload.clearFiles()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
              }
            })
          } else { // 修改数据
            if (this.dialogFormData.goodsFirstClassifyId === this.dialogFormData.goodsFirstClassifyName) {
              this.dialogFormData.goodsFirstClassifyId = this.dialogFirstId
            }
            if (this.dialogFormData.goodsSecondClassifyId === this.dialogFormData.goodsSecondClassifyName) {
              this.dialogFormData.goodsSecondClassifyId = this.dialogSecondId
            }
            req('updateGoodById', {...this.dialogFormData,
              version: this.tableSelectRows[0].version,
              goodsId: this.tableSelectRows[0].goodsId
            }).then(data => {
              // console.log(data)
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.$refs.form.resetFields()
                this.fetch()
                this.dialogvisible = false
              } else {
                this.$message.error('修改失败')
                // this.$refs.form.resetFields()
                // this.dialogvisible = false
              }
            })
          }
        } else {
          return false
        }
      })
    },
    // 图片上传
    handleRemove (data, file) {
      // console.log('Photos', data)
      this.dialogFormData.goodsPhoto = data.data
    },
    handlePreview (file) {
      // console.log(file)
    },
    handleExceed (files, fileList) {
      this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    beforeRemove (file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    }
  }
}

</script>
<style lang="scss" scoped>
.el-main {
.i-search {
  /deep/ .el-form {
     .el-form-item {
      margin-right: 5%;
      .el-form-item__content {
        width: 221.4px;
      }
    }
  }
}
}
.amount-input {
  width: 100%;
  height: 30px;
  border: 1px solid rgb(220, 223, 220);
  box-sizing: border-box;
  border-radius: 5px;
  margin: 0;
  // padding: 0;
  padding: 0 13px;
}
</style>
