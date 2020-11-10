<template>
  <div class="container">
    <div class="comm-box">
      <el-button class="add-btn" @click="openDialog" type="primary" icon="el-icon-plus"></el-button>
      <span>{{goodsId}}</span>
    </div>

    <i-dialog
      v-model="dialogvisible"
      :title="dialogTitle"
      @dialog-before-close="dialogBeforeClose"
      @dialog-cancel="dialogCancel"
      @dialog-confirm="dialogConfirm"
      @dialog-open="dialogOpen"
      :appendToBody="true">

      <i-search ref="iSearch" :model="formData" @search="fetch" @reset="reset">
        <el-form-item label="商品名称" prop="goodsName">
          <el-input v-model="formData.goodsName"></el-input>
        </el-form-item>
        <el-form-item label="商品编码" prop="goodsId">
          <el-input v-model="formData.goodsId"></el-input>
        </el-form-item>
      </i-search>

      <i-table
        :tableData="tableData"
        :pageInfo="pageInfo"
        :selectionShow="true"
        v-loading="tableLoading"
        @handleSizeChange="handleSizeChange"
        @handleCurrentChange="handleCurrentChange"
        @handleSelectionChange="handleSelectionChange">
        <el-table-column
          v-for="(item, index) in columnList"
          :key="index"
          :label="item.label"
          :prop="item.prop"
          align="center"
          :width="item.width"
          :show-overflow-tooltip="true"
          :formatter="columnFormatter">
        </el-table-column>
      </i-table>
    </i-dialog>
  </div>
</template>

<script>
import IDialog from '../../components/common/i-dialog'
import ITable from '../../components/common/i-table'
import ISearch from '../../components/common/i-search'
import req from '@/api/hot-comm-manage.js'

export default {
  name: 'comm-select',
  components: {
    IDialog,
    ITable,
    ISearch
  },
  props: {
    value: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      goodsId: '',
      dialogvisible: false,
      dialogTitle: '轮播商品选择',
      formData: {
        goodsName: '',
        goodsId: ''
      },
      tableData: [],
      pageInfo: {
        pageIndex: 1,
        pageSize: 5,
        totalSize: 0
      },
      goodsStateOptions: [
        {label: '未发布', value: '0'},
        {label: '在售', value: '1'},
        {label: '已下架', value: '2'}
      ],
      tableLoading: false,
      tableSelectRows: [],
      columnList: [
        {label: '商品编号', prop: 'goodsId'},
        {label: '商品名称', prop: 'goodsName'},
        {label: '商品状态', prop: 'goodsState', distName: 'goodsStateOptions'},
        {label: '一级分类', prop: 'goodsFirstClassifyName'},
        {label: '二级分类', prop: 'goodsSecondClassifyName'}
      ]
    }
  },
  watch: {
    value: {
      handler (value) {
        this.goodsId = value
      },
      deep: true
    }
  },
  created () {
    this.fetch()
  },
  methods: {
    openDialog () {
      this.dialogvisible = true
    },
    dialogBeforeClose () {
      this.formData.goodsName = ''
      this.formData.goodsId = ''
      this.dialogvisible = false
    },
    dialogOpen () {
      this.fetch()
    },
    dialogCancel () {
      this.formData.goodsName = ''
      this.formData.goodsId = ''
      this.dialogvisible = false
    },
    dialogConfirm () {
      if (this.tableSelectRows.length === 0) {
        this.$message.info('请至少选择一条数据')
        return
      }

      if (this.tableSelectRows.length > 1) {
        this.$message.info('只能选择一条数据')
        return
      }
      // console.log(this.tableSelectRows[0].goodsId, '111111')
      this.$emit('input', this.tableSelectRows[0].goodsId)
      this.formData.goodsName = ''
      this.formData.goodsId = ''
      this.dialogvisible = false
    },
    fetch () {
      this.pageInfo.pageSize = 5
      this.pageInfo.pageNum = 1
      this.search()
    },
    search () {
      this.tableLoading = true

      req('listGoods', {
        ...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageNum
      }).then(data => {
        // console.log('pageSize', data.data.pageNum)
        this.tableLoading = false
        this.tableData = data.data.list
        this.pageInfo.pageNum = data.data.pageNum
        this.pageInfo.pageSize = data.data.pageSize
        this.pageInfo.totalSize = data.data.total
        // console.log('pageIfo', this.pageInfo.total)
      }).catch(() => {
        this.tableLoading = false
      })
    },
    reset () {
      this.fetch()
    },
    handleSizeChange (value) {
      // console.log('条', value)
      this.pageInfo.pageSize = value
      this.search()
    },
    handleCurrentChange (value) {
      // console.log('页', value)
      this.pageInfo.pageNum = value
      this.search()
    },
    handleSelectionChange (row) {
      // console.log('row', row)
      this.tableSelectRows = row
    },
    // 表格表头的循环list变量名一定要是columnList
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

<style lang="scss" scoped>
.container {
  .comm-box {
    display: flex;
    justify-content: space-between;

    .add-btn {
      padding: 0 10px;
    }

    span:nth-child(2) {
      flex: 1;
      height: 30px;
      line-height: 30px;
      overflow: hidden;
      padding: 0 5px;
      box-sizing: border-box;
      cursor: pointer;

      &:hover {
        color: #409EFF;
      }
    }

    .delete-btn {
      padding: 0 10px;
    }
  }
}
</style>
