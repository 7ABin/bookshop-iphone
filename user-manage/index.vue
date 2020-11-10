<template>
  <div>
    <i-search ref="ISearch" @search="search" @reset="reset" :model="formData">
      <el-form-item label="用户名" prop="name">
        <el-input v-model="formData.name"></el-input>
      </el-form-item>
      <el-form-item label="用户账号" prop="userAccount">
        <el-input v-model="formData.userAccount"></el-input>
      </el-form-item>
      <el-form-item label="角色" prop="role">
        <el-select v-model="formData.role">
          <el-option
            v-for="(item, index) in optionup"
            :key="index"
            :label="item.label"
            :value="item.value">
          </el-option>
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
      v-for="(item, index) in columnList"
      :key='index'
      :label="item.label"
      :prop="item.prop"
      align="center"
      :show-overflow-tooltip="true"
      :formatter="columnFormatter">
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
          <el-form-item label="用户账号" prop="userAccount">
          <el-input v-model="dialogFormData.userAccount" @input="change($event)"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="用户姓名" prop="name">
          <el-input v-model="dialogFormData.name"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="用户性别" prop="sex">
           <el-select v-model="dialogFormData.sex" >
           <el-option
           v-for="(item, index) in sex"
           :key="index"
           :label="item.label"
           :value="item.value"
           ></el-option>
          </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="手机号" prop="phone">
          <el-input v-model="dialogFormData.phone"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="用户邮箱" prop="email">
          <el-input  v-model="dialogFormData.email"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="身份证" prop="idCard">
          <el-input  v-model="dialogFormData.idCard"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="用户密码" prop="userPassword">
          <el-input  v-model="dialogFormData.userPassword" type="password"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
           <el-form-item label="用户角色" prop="role">
          <el-select v-model="dialogFormData.role" >
           <el-option
           v-for="(item, index) in optionup"
           :key="index"
           :label="item.label"
           :value="item.value"
           ></el-option>
          </el-select>
         </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="上传头像" prop="photo">
            <!-- <el-upload
          class="upload-demo"
          action="http://39.98.76.3:8080/webauth/uploadImage"
          :on-success="handleRemove"
          :file-list="dialogFormData.photo"
          :data="message"
          ref="upload"
          list-type="picture"
          >
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip"></div>
          </el-upload> -->
          <i-file v-model="dialogFormData.photo"></i-file>
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
import req from '@/api/user-manage'
import IFile from '@/components/common/i-file.vue'
// import axios from 'axios'
export default {
  name: 'user-manage',
  components: {
    ISearch,
    ITable,
    IDialog,
    IFile
  },
  data () {
    return {
      message: {access_token: JSON.parse(sessionStorage.getItem('userInfo')).access_token},
      imageUrl: '', // 头像上传
      option: [
        {label: '管理员', value: '0'},
        {label: '司机', value: '1'},
        {label: '客户', value: '2'},
        {label: '店长', value: '3'}],
      optionup: [
        {label: '管理员', value: '0'},
        {label: '店长', value: '3'}],
      sex: [
        {label: '男', value: '0'},
        {
          label: '女', value: '1'
        }
      ],
      tableSelectRows: [], // 装入选中的
      formData: {},
      dialogvisible: false,
      dialogTitle: '新增用户',
      dialogtype: 'create',
      dialogFormData: {},
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            // console.log(this.pageInfo)
            this.dialogvisible = true
            this.dialogtype = 'create'
            this.dialogTitle = '新增用户'
          }
        },
        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            // console.log(this.tableSelectRows.length)
            if (this.tableSelectRows.length === 0) {
              this.$message.info('请选择数据')
              return
            }
            if (this.tableSelectRows.length > 1) {
              this.$message.info('不支持多数据修改')
              return
            }
            this.dialogvisible = true
            this.dialogTitle = '修改用户数据'
            this.dialogtype = 'userup'
            // 读取数据
            // console.log(this.tableSelectRows[0].version)
            req('findUserById', {userId: this.tableSelectRows[0].userId}).then(data => {
              this.dialogFormData = data.data
              this.dialogFormData.userId = this.tableSelectRows[0].userId
              this.dialogFormData.version = this.tableSelectRows[0].version
              // console.log(this.dialogFormData)
            })
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
                return item.userId
              }).toString()
              req('deleteUser', {userId: ids}).then(data => {
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
        {label: '账号', prop: 'userAccount'},
        {label: '用户编号', prop: 'userId'},
        {label: '姓名', prop: 'name'},
        {label: '角色', prop: 'role', distName: 'optionup'},
        {label: '性别', prop: 'sex', distName: 'sex'},
        {label: '手机', prop: 'phone'},
        {label: '邮箱', prop: 'email'},
        {label: '身份证', prop: 'idCard'}
      ],
      tableData: [],
      formRules: {
        userAccount: [
          { required: true, message: '请填写用户账号', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '请填写用户姓名', trigger: 'blur' }
        ],
        sex: [
          { required: true, message: '请选择性别', trigger: 'change' }
        ],
        phone: [
          { required: true, message: '请填写手机号', trigger: 'blur' },
          {message: '手机号格式不对', pattern: /^0{0,1}(13[0-9]|15[7-9]|153|156|18[7-9])[0-9]{8}$/}
        ],
        email: [
          { required: true, message: '请填写用户邮箱', trigger: 'blur' },
          {pattern: /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/, message: '请输入正确的邮箱格式'}
        ],
        idCard: [
          { required: true, message: '请填写身份证', trigger: 'blur' },
          { validator: this.validateIdCard, trigger: 'blur' }
        ],
        userPassword: [
          { required: true, message: '请填写用户密码', trigger: 'blur' }
        ],
        role: [
          { required: true, message: '请填写用户角色', trigger: 'change' }
        ]
      }
    }
  },
  created () {
    this.fetch()
  },
  mounted () {
    this.pageInfo.totalSize = this.tableData.length
    // this.loadList()
  },
  methods: {
    change (e) {
      this.$forceUpdate()
    },
    fetch () {
      this.pageInfo.pageIndex = 1
      this.pageInfo.pageSize = 5
      // console.log(this.formData, '重置进去时')
      this.search()
    },
    validateIdCard (rule, value, callback) {
      let reg = /^[1-9]\d{5}(18|19|20)\d{2}((0[1-9])|(1[0-2]))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/
      if (!reg.test(value)) {
        callback(new Error('请输入正确身份证号'))
      } else {
        callback()
      }
    },
    search () {
      // console.log(this.formData, '第一次')
      req('listUsers', {
        ...this.formData,
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
        // console.log(this.formData, 'this.formData获取后')
      })
    },
    reset () {
      // console.log('重置')
      this.fetch()
    },
    handleSizeChange (value) {
      // console.log('每页多少条', value)
      // this.fetch()
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
      // this.$refs.upload.clearFiles()
      this.$refs.form.resetFields()
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      // this.$refs.upload.clearFiles()
      this.$refs.form.resetFields()
    },
    dialogConfirm () {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogtype === 'create') {
            // console.log(this.dialogFormData)
            req('addUser', {...this.dialogFormData}).then(data => {
              // console.log(data)
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
          } else { // 修改数据
            req('updateUserById', {...this.dialogFormData}).then(data => {
              // console.log(data)
              if (data.code === 0) {
                this.$message.success(data.msg)
                this.fetch()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error('修改失败')
              }
            })
          }
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
    handleRemove (data, file) {
      // console.log('Photos', data)
      this.dialogFormData.photo = data.data
    }
  }
}
</script>
<style lang="scss" scoped>
 .el-dialog.el-dialog__body.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
