<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="客户名称" prop="customerName">
        <el-input v-model="formData.customerName"></el-input>
      </el-form-item>
      <el-form-item label="客户账号" prop="customerAccount">
        <el-input v-model="formData.customerAccount"></el-input>
      </el-form-item>
    </i-search>

    <i-table
    :toolbar="toolbar"
    :tableData="tableData"
    :pageInfo="pageInfo"
    :selectionShow='false'
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
      :formatter="columnFormatter"
      >
      </el-table-column>
    </i-table>

  </div>
</template>
<script>
import ISearch from '../../components/common/i-search'
import ITable from '../../components/common/i-table'
import IDialog from '../../components/common/i-dialog'
import req from '@/api/client-manage'
export default {
  name: 'client-manage',
  components: {
    ISearch,
    ITable,
    IDialog
  },
  data () {
    return {
      formData: {
        customerName: '',
        customerAccount: ''
      },
      toolbar: [
      ],
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        total: 0
      },
      sex: [
        {label: '男', value: '0'},
        {
          label: '女', value: '1'
        }
      ],
      columnList: [
        {label: '账号', prop: 'customerAccount'},
        {label: '姓名', prop: 'customerName'},
        {label: '性别', prop: 'customerSex', distName: 'sex'},
        {label: '手机', prop: 'customerPhone'},
        {label: '邮箱', prop: 'email'},
        {label: '身份证', prop: 'idCard'}
      ],
      tableData: []
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
      // console.log('搜索')
      // console.log(this.formData)
      req('listCustomers', {
        ...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        // console.log(data)
        this.tableData = data.data.list
        // console.log(data)
        // 列表栏获取
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
  }
}
</script>
