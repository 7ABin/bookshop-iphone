<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="商品名称" prop="goodsName">
        <el-input v-model="formData.goodsName"></el-input>
      </el-form-item>
      <el-form-item label="商品编号" prop="goodsId">
        <el-input v-model="formData.goodsId"></el-input>
      </el-form-item>
    </i-search>

    <i-table
    :toolbar="toolbar"
    :tableData="tableData"
    :pageInfo="pageInfo"
    :selectionShow='true'
    @handleSizeChange="handleSizeChange"
    @handleCurrentChange="handleCurrentChange"
    @handleSelectionChange='handleSelectionChange'
    >
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
    @dialog-before-close="dialogBeforeClose"
    >
    <el-form ref="form" :model="dialogFormData" label-width="100px" :rules="formRules">
      <el-row >
        <el-col :span="12" v-if="dialogtype !=='Exhibition'">
          <el-form-item label="商品编号" prop="goodsId">
          <comm-select v-model="dialogFormData.goodsId"></comm-select>
          </el-form-item>
        </el-col>
        <el-col :span="12" v-if="dialogtype !=='Exhibition'">
           <el-form-item label="排序" prop="hotgoodsSort">
        <el-input v-model="dialogFormData.hotgoodsSort"></el-input>
        </el-form-item>
        </el-col>
        <!-- <el-col :span="24" v-if="dialogtype ==='Exhibition'">
          <el-form-item label="数量" prop="v">
        <el-input v-model.number="dialogFormData.v"></el-input>
        </el-form-item>
        </el-col> -->
        </el-row>
    </el-form>
    </i-dialog>
        <i-dialog
      v-model="hotDialogVisible"
      title="数量展示设置"
      @dialog-before-close="hotDialogBeforeClose"
      @dialog-cancel="hotDialogCancel"
      @dialog-confirm="hotDialogConfirm"
      >
      <el-form :model="hotDialogFormData" ref="hotForm" :rules="hotFormRules" label-width="120px">
        <el-row>
          <el-col :span='12'>
            <el-form-item label="数量展示设置" prop="v">
              <el-input v-model="hotDialogFormData.v"></el-input>
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
import CommSelect from './comm-select'
import req from '@/api/hot-comm-manage'
export default {
  name: 'user-manage',
  components: {
    ISearch,
    ITable,
    IDialog,
    CommSelect
  },
  data () {
    return {
      formData: {},
      dialogvisible: false,
      dialogTitle: '',
      dialogtype: '',
      dialogFormData: {},
      hotDialogFormData: {},
      tableSelectRows: [],
      hotDialogVisible: false,
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            this.dialogvisible = true
            this.dialogTitle = '新增用户'
            this.dialogtype = 'create'
          }
        },
        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            if (this.tableSelectRows.length > 1) {
              this.$message.info('不支持多数据修改')
              return
            }
            req('findHotgoodById', {hotgoodsId: this.tableSelectRows[0].hotgoodsId}).then(data => {
              // console.log(data)
              this.dialogFormData = data.data
              this.$nextTick(() => {
                this.$refs.form.validateField('goodsId')
              })
            })
            this.dialogvisible = true
            this.dialogTitle = '修改用户'
            this.dialogtype = 'getup'
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
                return item.hotgoodsId
              }).toString()
              req('deleteHotgood', {hotgoodsId: ids}).then(data => {
                this.$message.success(data.msg)
                this.fetch()
              })
            }).catch(() => {
            })
          }
        },
        {
          btnName: '展示数量设置',
          type: 'primary',
          func: () => {
            // if (this.tableSelectRows.length === 0) {
            //   this.$message.info('请选择数据')
            //   return
            // }
            // if (this.tableSelectRows.length > 1) {
            //   this.$message.info('不支持多数据修改')
            //   return
            // }
            this.hotDialogVisible = true
            // this.dialogtype = 'Exhibition'
            // this.dialogTitle = '修改数量'
            req('queryHotgoodNumber', {}).then(data => {
              this.hotDialogFormData = data.data
            })
          }
        }
      ],
      innerVisible: false,
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        totalSize: 0
      },
      columnList: [
        {label: '排序', prop: 'hotgoodsSort'},
        {label: '商品编码', prop: 'goodsId'},
        {label: '商品名称', prop: 'goodsName'},
        {label: '商品价格', prop: 'goodsSaleMoney'},
        {label: '商品介绍', prop: 'goodsProduce'}
      ],
      tableData: [],
      formRules: {
        goodsId: [
          { required: true, message: '请选择商品编号', trigger: 'change' }
        ],
        hotgoodsSort: [
          { required: true, message: '请输入商品排序', trigger: 'change' }
        ]
      },
      hotFormRules: {
        v: [
          { required: true, message: '请输入数量', trigger: 'change' }
        ]},
      formDatason: []
    }
  },
  watch: {
    'dialogFormData.goodsId': {
      handler (value) {
        this.$refs.form.validateField('goodsId')
      },
      deep: true
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
      req('listHotgoods', {
        ...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        this.tableData = data.data.list
        this.pageInfo.pageIndex = data.data.pageNum
        // // 分页获取
        this.pageInfo.pageSize = data.data.pageSize
        this.pageInfo.totalSize = data.data.total
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
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogtype === 'create') {
            req('addHotgood', {...this.dialogFormData}).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
              }
            })
          } else {
            // console.log(this.tableSelectRows[0], '<--------------')
            req('updateHotgoodById', {...this.dialogFormData,
              version: this.tableSelectRows[0].version,
              hotgoodsId: this.tableSelectRows[0].hotgoodsId
            }).then(data => {
              // console.log(data, '$--$')
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        } else {
          return false
        }
      })
    },
    func () {
      this.innerVisible = true
    },
    hotDialogBeforeClose () {
      this.$nextTick(() => {
        this.$refs.hotForm.resetFields()
      })
    },
    hotDialogCancel () {
      this.$refs.hotForm.resetFields()
      this.hotDialogVisible = false
    },
    hotDialogConfirm () {
      this.$refs.hotForm.validate((valid) => {
        if (valid) {
          this.hotDialogFormData.v = this.hotDialogFormData.v.toString()
          // console.log(this.dialogFormData.v, 'this.dialogFormData.v')
          req('updateHotgoodNumber', {
            ...this.hotDialogFormData
          }).then(data => {
            if (data.code === 0) {
              this.$message.success(data.msg)
              this.fetch()
              this.$refs.hotForm.resetFields()
              this.hotDialogVisible = false
            } else {
              this.$message.error(data.msg)
            }
          })
        } else {
          return false
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.el-dialog{
  .el-dialog__body{
    padding: 30px 192px;
    background-color: red;
    .el-form{
     .box1{
      display: flex;
      justify-content: center;
      .el-form-item{
        margin-left: -90px;
        /deep/ .el-form-item__content{
          margin-left: 10px;
        }
      }
      }
    }
  }
}
.box2{
    position: relative;
    display: flex;
    justify-content: center;
    box-sizing: border-box;
    padding-right: 161px;

}
</style>
