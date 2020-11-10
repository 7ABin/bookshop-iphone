<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="状态" prop='slideshowsState'>
      <el-select v-model='formData.slideshowsState' >
           <el-option
           v-for="item in stateOption"
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
    @handleSelectionChange='handleSelectionChange'
    >
      <el-table-column prop="slideshowsSort" label='排序' align='center'>
      </el-table-column>
      <el-table-column prop="slideshowsRoute" label='图片路径' align='center' :show-overflow-tooltip="true">
      </el-table-column>
      <el-table-column prop="slideshowsState" label='状态' align='center' :formatter="columnFormatter">
      </el-table-column>
      <el-table-column label='预览' align='center'>
        <template slot-scope="scope">
        <el-link :href="scope.row.slideshowsRoute" type="primary" v-if="scope.row.slideshowsRoute">预览</el-link>
        <div v-else>无图片</div>
        </template>
      </el-table-column>
      <el-table-column prop="slideshowsTimeStart" label='有效期起' align='center'>
      </el-table-column>
      <el-table-column prop="slideshowsTimeEnd" label='有效期止' align='center'>
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
          <el-form-item label="排序" prop="slideshowsSort">
          <el-input v-model.number="dialogFormData.slideshowsSort"></el-input>
          </el-form-item>
        </el-col>

        <el-col :span="12">
        <el-form-item label="有效起启" prop="slideshowsTimeStart">
        <el-date-picker type="date" placeholder="有效期起"
        v-model="dialogFormData.slideshowsTimeStart"
        :picker-options="startTimePickerOptions"
        style="width: 100%;"
        ></el-date-picker>
        </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="有效起止" prop="slideshowsTimeEnd">
        <el-date-picker type="date" placeholder="有效期止"
        v-model="dialogFormData.slideshowsTimeEnd"
        :picker-options="endTimePickerOptions"
         style="width: 100%;"></el-date-picker>
        </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="商品" prop="goodsId">
          <select-change v-model="dialogFormData.goodsId"></select-change>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="图片" prop="slideshowsRoute">
         <!-- <el-upload
          class="upload-demo"
          action="http://39.98.76.3:8080/webauth/uploadImage"
          :on-success="handleRemove"
          :data="message"
          ref="upload"
          >
          <el-button size="small" type="primary">点击上传</el-button>
            </el-upload> -->
            <i-file v-model="dialogFormData.slideshowsRoute"></i-file>
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
import req from '@/api/home-pic-manage'
import selectChange from './select-change'
import moment from 'moment'
import IFile from '@/components/common/i-file.vue'
export default {
  name: 'home-pic-manage',
  components: {
    ISearch,
    ITable,
    IDialog,
    selectChange,
    IFile
  },
  data () {
    return {
      tableSelectRows: [],
      message: {access_token: JSON.parse(sessionStorage.getItem('userInfo')).access_token},
      fileList: [],
      formData: {},
      // slideshowsState: '',
      stateOption: [
        {label: '禁用', value: '0'},
        {label: '启用', value: '1'}
      ],
      dialogvisible: false,
      dialogTitle: '新增轮播图',
      dialogFormData: {},
      // 时间处理
      startTimePickerOptions: {
        disabledDate: (value) => {
          return this.dialogFormData.slideshowsTimeEnd < value
        }
      },
      endTimePickerOptions: {
        disabledDate: (value) => {
          return this.dialogFormData.slideshowsTimeStart > value
        }
      },
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            this.dialogvisible = true
            this.dialogBuildName = 'create'
            this.dialogTitle = '新增轮播图'
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            // console.log(111)
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择删除的数据')
              return
            }
            this.$confirm('是否确认删除').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.slideshowsId
              }).toString()
              req('deleteSlideshow', {slideshowsId: ids}).then(data => {
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
          btnName: '启用',
          type: 'primary',
          func: () => {
            // console.log(this.tableSelectRows.length)
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择启用的数据')
              return
            }
            this.$confirm('是否确认启用').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.slideshowsId
              }).toString()
              req('updateSlideshowState', {slideshowsId: ids, slideshowsState: '1'}).then(data => {
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
          btnName: '禁用',
          type: 'primary',
          func: () => {
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择禁用的数据')
              return
            }
            this.$confirm('是否确认禁用').then(() => {
              let ids = this.tableSelectRows.map(item => {
                return item.slideshowsId
              }).toString()
              req('updateSlideshowState', {slideshowsId: ids, slideshowsState: '0'}).then(data => {
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
        totalSize: 0
      },
      columnList: [
        {label: '排序', prop: 'slideshowsSort'},
        {label: '图片路径', prop: 'slideshowsRoute'},
        {label: '状态', prop: 'slideshowsState', distName: 'stateOption'},
        // {label: '预览', prop: 'url1'},
        {label: '有效期起', prop: 'slideshowsTimeStart'},
        {label: '有效期止', prop: 'slideshowsTimeEnd'}
      ],
      tableData: [],
      formRules: {
        slideshowsRoute: [
          { required: true, message: '请选择图片', trigger: 'change' }
        ],
        slideshowsSort: [
          { required: true, message: '请选择顺序', trigger: 'change' }
        ],
        slideshowsTimeStart: [
          { required: true, message: '请选择开始时间', trigger: 'change' }
        ],
        slideshowsTimeEnd: [
          { required: true, message: '请选择截止时间', trigger: 'change' }
        ],
        goodsId: [
          { required: true, message: '请选择商品', trigger: 'change' }
        ]
      }
    }
  },
  watch: {
    'dialogFormData.goodsId': {
      handler (value) {
        this.$refs.form.validateField('goodsId')
      },
      deep: true
    },
    'dialogFormData.slideshowsRoute': {
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
      // console.log(this.formData)
      req('listSlideshows', {
        ...this.formData,
        pageSize: this.pageInfo.pageSize,
        pageNum: this.pageInfo.pageIndex
      }).then(data => {
        // console.log(data, '1231')
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
    handleSelectionChange (value) {
      // console.log(value)
      this.tableSelectRows = value
    },
    dialogCancel () {
      this.$refs.form.resetFields()
      // this.$refs.upload.clearFiles()
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      this.$refs.form.resetFields()
      // this.$refs.upload.clearFiles()
    },
    dialogConfirm () {
      // console.log('dialog确定按钮点击')
      this.$refs.form.validate((valid) => {
        if (valid) {
          // console.log('okk')
          this.dialogFormData.slideshowsSort = this.dialogFormData.slideshowsSort.toString()
          // console.log(this.dialogFormData.slideshowsSort)
          this.dateFormat()
          req('addSlideshow', {...this.dialogFormData}).then(data => {
            // console.log(data, '---------===---')
            if (data.code === 0) {
              this.$message.success(data.msg)
              this.fetch()
              this.$refs.form.resetFields()
              // this.$refs.upload.clearFiles()
              this.dialogvisible = false
            } else {
              this.$message.error(data.msg)
              // this.$refs.form.resetFields()
              // this.$refs.upload.clearFiles()
              // this.dialogvisible = false
            }
          })
        } else {
          return false
        }
      })
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
    },
    // 图片上传
    handleRemove (data) {
      // console.log('Photos', data)
      this.dialogFormData.slideshowsRoute = data.data
    },
    // 日期格式化
    dateFormat () {
      // console.log(this.dialogFormData.slideshowsTimeStart, 'dialogFormData.slideshowsTimeStart')
      this.dialogFormData.slideshowsTimeStart = moment(this.dialogFormData.slideshowsTimeStart).format('YYYY-MM-DD')
      this.dialogFormData.slideshowsTimeEnd = moment(this.dialogFormData.slideshowsTimeEnd).format('YYYY-MM-DD')
    }
  }
}

</script>
