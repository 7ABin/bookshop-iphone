<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="下单人姓名" prop="customerName">
        <el-input v-model="formData.customerName"></el-input>
      </el-form-item>
      <el-form-item label="订单编码" prop="ordersId" @input="change($event)">
        <el-input v-model="formData.ordersId"></el-input>
      </el-form-item>
      <el-form-item label="付款时间起" prop="payTimeStart">
        <el-date-picker
      v-model="formData.payTimeStart"
      :picker-options="endTimePickerOptions"
      placeholder="选择日期">
    </el-date-picker>
      </el-form-item>

    <el-form-item label="付款时间晚" prop="payTimeEnd">
     <el-date-picker
    v-model="formData.payTimeEnd"
    :picker-options="endTimePickerOptions"
    placeholder="选择日期">
    </el-date-picker>
      </el-form-item>

       <el-form-item label="手机号" prop="customerPhone">
        <el-input v-model="formData.customerPhone"></el-input>
      </el-form-item>
      <el-form-item label="订单状态" prop="ordersState">
          <el-select v-model="formData.ordersState" >
           <el-option
           v-for="item in orderStatus"
           :key="item.value"
           :label="item.label"
           :value="item.value"
           ></el-option>
          </el-select>
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
      :show-overflow-tooltip="true"
      v-for="(item, index) in columnList"
      :key='index'
      :label="item.label"
      :prop="item.prop"
      align="center"
      :formatter="columnFormatter"
      >
      </el-table-column>
    </i-table>
  <i-dialog
    v-model="dialogvisible"
    :title='dialogTitle'
    @dialog-cancel="dialogCancel"
    @dialog-confirm="dialogConfirm"
    @dialog-before-close="dialogBeforeClose">
    <i-table
    :tableData="tableDatason"
    :selectionShow='false'
    :tablelength='false'
    >
    <el-table-column
    ref="form"
      v-for="(item, index) in columnListson"
      :key='index'
      :label="item.label"
      :prop="item.prop"
      align="center"
      >
      </el-table-column></i-table>
  </i-dialog>
  </div>
</template>
<script>
import ISearch from '../../components/common/i-search'
import ITable from '../../components/common/i-table'
import IDialog from '../../components/common/i-dialog'
import req from '@/api/order-manage'
import moment from 'moment'
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
      orderStatus: [
        {label: '未到货', value: '0'},
        {label: '已到货', value: '1'},
        {label: '未取货', value: '2'},
        {label: '已取货', value: '3'},
        {label: '未评价', value: '4'},
        {label: '已评价', value: '5'},
        {label: '已取消', value: '6'}],
      PayState: [
        {label: '未支付', value: '0'},
        {label: '已支付', value: '1'}
      ],
      // 时间处理
      startTimePickerOptions: {
        disabledDate: (value) => {
          return this.formData.payTimeEnd < value
        }
      },
      endTimePickerOptions: {
        disabledDate: (value) => {
          return this.formData.payTimeStart > value
        }
      },
      formData: {},
      dialogvisible: false,
      dialogTitle: '',
      dialogFormData: {},
      value1: '',
      value2: '',
      toolbar: [
        {
          btnName: '详情',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            if (this.tableSelectRows.length > 1) {
              this.$message.info('不支持多数据查看')
              return
            }
            this.dialogvisible = true
            this.dialogTitle = '订单详情'
            req('findOrderById', {ordersId: this.tableSelectRows[0].ordersId}).then(data => {
              this.tableDatason = data.data
            })
          }
        },
        {
          btnName: '订单取消',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            this.$confirm('是否确认取消订单').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.ordersId
              }).toString()
              req('updateOrderState', {ordersId: ids, ordersState: '6'}).then(data => {
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
          btnName: '订单到货',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            this.$confirm('是否确认订单到货').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.ordersId
              }).toString()
              req('updateOrderState', {ordersId: ids, ordersState: '1'}).then(data => {
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
          btnName: '取消到货',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            this.$confirm('是否确认取消到货').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.ordersId
              }).toString()
              req('updateOrderState', {ordersId: ids, ordersState: '0'}).then(data => {
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
          btnName: '订单已取货',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            this.$confirm('是否确认订单已取货').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.ordersId
              }).toString()
              req('updateOrderState', {ordersId: ids, ordersState: '3'}).then(data => {
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
          btnName: '取消已取货',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            this.$confirm('是否确认取消已取货').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.ordersId
              }).toString()
              req('updateOrderState', {ordersId: ids, ordersState: '2'}).then(data => {
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
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        total: 0
      },
      columnList: [
        {label: '订单编码', prop: 'ordersId'},
        {label: '订单总价', prop: 'ordersTotalMoney'},
        {label: '订单状态', prop: 'ordersState', distName: 'orderStatus'},
        {label: '支付状态', prop: 'ordersPayState', distName: 'PayState'},
        {label: '门店编码', prop: 'storeId'},
        {label: '下单人姓名', prop: 'customerName'},
        {label: '下单人手机号', prop: 'customerPhone'},
        {label: '确认付款时间', prop: 'ordersPayTime'}
      ],
      columnListson: [
        {label: '商品编码', prop: 'goodsId'},
        {label: '商品名称', prop: 'goodsName'},
        {label: '购买数量', prop: 'buyNumber'},
        {label: '总金额', prop: 'totalMoney'},
        {label: '售价', prop: 'goodsSaleMoney'},
        {label: '定价', prop: 'goodsFixMoney'}
      ],
      tableData: [],
      tableDatason: []
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
    change (e) {
      this.$forceUpdate()
    },
    fetch () {
      this.pageInfo.pageIndex = 1
      this.pageInfo.pageSize = 5
      this.search()
    },
    search () {
      // console.log('搜索')
      req('listOrders', {...this.formData,
        payTimeStart: this.formData.payTimeStart ? moment(this.formData.payTimeStart).format('YYYY-MM-DD') : '',
        payTimeEnd: this.formData.payTimeEnd ? moment(this.formData.payTimeEnd).format('YYYY-MM-DD') : '',
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        this.tableData = data.data.list
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
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      this.dialogvisible = false
    },
    dialogConfirm () {
      // console.log('dialog确定按钮点击')
      this.dialogvisible = false
      // this.$refs.form.validate((valid) => {
      //   if (valid) {
      //     // console.log('okk')
      //     this.dialogvisible = false
      //   } else {
      //     return false
      //   }
      // })
    },
    columnFormatter (row, column, cellValue, index) {
      let distName = this.columnList.filter(item => {
        return item.prop === column.property
      })[0].distName
      // console.log(row)
      if (distName) {
        return this[distName].filter(item => {
          return item.value === row[column.property]
        })[0].label
      } else {
        return row[column.property]
      }
    }
    // 日期格式化
    // dateFormat () {
    //   // console.log(this.dialogFormData.slideshowsTimeStart, 'dialogFormData.slideshowsTimeStart')
    //   this.formData.payTimeStart = moment(this.formData.payTimeStart).format('YYYY-MM-DD')
    //   this.formData.payTimeEnd = moment(this.formData.payTimeEnd).format('YYYY-MM-DD')
    // }
  }
}
</script>
<style lang="scss" scoped>
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
