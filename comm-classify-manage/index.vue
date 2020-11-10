<template>
<div>
  <div  class="conter">
    <div class="header">
      <div class="header-box1">
    <i-table
    :toolbar="toolbar"
    :selectionShow='false'
    :selectionShowForm='false'
    :tablelength='false'
    >
    </i-table>
      </div>
      <div class="header-box2">
        <div class="header-box2-son">分类详情</div>
        </div>
    </div>
    <div class="body">
      <div class="body-box1">
        <el-tree :data="book" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
      </div>
      <div class="body-box2" >
        <div>
          <span>分类名称：</span>
          <input v-model="classifyName" />
        </div>
        <div>
          <span>备注：</span>
          <input v-model="classifyNote" />
        </div>
      </div>
    </div>
  </div>
   <i-dialog
    v-model="dialogvisible"
    :title='dialogTitle'
    :dialogwidth='dialogwidth'
    @dialog-cancel="dialogCancel"
    @dialog-confirm="dialogConfirm"
    @dialog-before-close="dialogBeforeClose">
    <el-form ref="form" :model="dialogFormData" label-width="100px" :rules="formRules">
      <el-row >
        <el-col :span="24">
          <el-form-item label="名称" prop="classifyName">
          <el-input v-model="dialogFormData.classifyName" @input="change($event)"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <el-form-item label="备注" prop="classifyNote">
          <el-input v-model="dialogFormData.classifyNote" @input="change($event)"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="24" v-if="this.dialogtype === 'create'">
          <el-form-item label="新增等级" prop="curreroption">
           <el-select v-model="dialogFormData.curreroption" >
           <el-option
           v-for="(item, index) in curreroption"
           :key="index"
           :label="item.label"
           :value="item.value"
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
import ITable from '../../components/common/i-table'
import IDialog from '../../components/common/i-dialog'
import req from '@/api/comm-classify-manage'
export default {
  name: 'comm-classify-manage',
  components: {
    ITable,
    IDialog
  },
  data () {
    return {
      classifyName: '',
      classifyNote: '',
      dialogtype: '',
      dialogFormData: {
        classifyName: '',
        classifyNote: ''
      },
      dialogvisible: false,
      dialogTitle: '',
      dialogwidth: 30,
      currentData: [], // 当前数据读取
      book: [],
      curreroption: [
        {label: '新增一级', value: 0},
        {label: '新增二级', value: 1}
      ],
      defaultProps: {
        // children: 'firstClassifyName',
        children: 'nextInfos',
        label: 'classifyName'
      },
      formRules: {
        classifyName: [
          { required: true, message: '请填写名称', trigger: 'blur' }
        ],
        curreroption: [
          { required: true, message: '请选择等级', trigger: 'change' }
        ]
      },
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            // if (this.currentData.length === 0) {
            //   this.currentData.classifyId = '0'
            // }
            // console.log('新增', this.currentData.classifyId)
            this.dialogvisible = true
            this.dialogTitle = '新增分类'
            this.dialogtype = 'create'
            // console.log(this.dialogvisible)
          }
        },

        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            // console.log(this.pageInfo)
            if (this.currentData.length === 0) {
              this.$message('请选择删除数据')
              return
            }
            this.dialogvisible = true
            this.dialogFormData.classifyName = this.classifyName
            this.dialogFormData.classifyNote = this.classifyNote
            this.dialogTitle = '修改'
            this.dialogtype = 'modify'
            // console.log(this.currentData)
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            // console.log(this.pageInfo)
            if (this.currentData.length === 0) {
              this.$message('请选择删除数据')
              return
            }
            this.$confirm('是否确认删除').then(() => {
              if (this.currentData.classifyId) {
                req('deleteGoodClassifyById', {
                  classifyId: this.currentData.classifyId
                }).then(data => {
                  if (data.code === 0) {
                    this.$message.success(data.msg)
                    this.fetch()
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              }
            }).catch(() => {
            })
          }
        },
        {
          btnName: '刷新',
          type: 'primary',
          func: () => {
            this.fetch()
          }
        }
      ]
    }
  },
  created () {
    this.fetch()
  },
  methods: {
    change (e) {
      this.$forceUpdate()
    },
    fetch () {
      req('queryGoodClassify', {}).then(data => {
        // console.log('book', data)
        this.book = data.data
      })
    },
    dialogCancel () {
      this.$refs.form.resetFields()
      this.dialogFormData = Object.assign({}, '')
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      this.$refs.form.resetFields()
      this.dialogFormData = Object.assign({}, '')
    },
    dialogConfirm () {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.dialogtype === 'create') {
            // console.log(this.currentData.classifyId)
            if (this.currentData.classifyId) {
              if (this.dialogFormData.curreroption === 0) {
                req('addGoodClassify', {
                  classifyName: this.dialogFormData.classifyName,
                  classifyNote: this.dialogFormData.classifyNote,
                  lastClassifyId: '0'
                }).then(data => {
                  if (data.code === 0) {
                    this.$message.success(data.msg) // 发送成功
                    this.dialogvisible = false
                    this.$refs.form.resetFields()
                    this.dialogFormData = Object.assign({}, '')
                    this.fetch()
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              } else {
                req('addGoodClassify', {
                  classifyName: this.dialogFormData.classifyName,
                  classifyNote: this.dialogFormData.classifyNote,
                  lastClassifyId: this.currentData.classifyId
                }).then(data => {
                  if (data.code === 0) {
                    this.$message.success(data.msg) // 发送成功
                    this.dialogvisible = false
                    this.$refs.form.resetFields()
                    this.dialogFormData = Object.assign({}, '')
                    this.fetch()
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              }
            } else { // 一开始新增的
              if (this.dialogFormData.curreroption === 0) {
                req('addGoodClassify', {
                  classifyName: this.dialogFormData.classifyName,
                  classifyNote: this.dialogFormData.classifyNote,
                  lastClassifyId: '0'
                }).then(data => {
                  if (data.code === 0) {
                    this.$message.success(data.msg) // 发送成功
                    this.dialogvisible = false
                    this.$refs.form.resetFields()
                    this.dialogFormData = Object.assign({}, '')
                    this.fetch()
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              } else {
                this.$message.error('请选择先选择一级分类')
              }
            }
          } else {
            req('updateGoodClassifyById', {
              classifyId: this.currentData.classifyId,
              classifyName: this.dialogFormData.classifyName,
              classifyNote: this.dialogFormData.classifyNote,
              version: this.currentData.version
            }).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg + '请刷新页面')
                // this.fetch()
                this.serch()
                this.dialogFormData = Object.assign({}, '')
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
    // 右侧获取数据
    serch () {
      // console.log('currentData', this.currentData)
      req('findGoodClassifyById', {classifyId: this.currentData.classifyId}).then(data => {
        // console.log(data, '1111111')
        this.classifyName = data.data.classifyName
        this.classifyNote = data.data.classifyNote
      })
    },
    handleNodeClick (data) {
      this.currentData = data
      // console.log('currentData', this.currentData)
      req('findGoodClassifyById', {
        classifyId: data.classifyId
      }).then(data => {
        // console.log(data)
        this.classifyName = data.data.classifyName
        this.classifyNote = data.data.classifyNote
      })
    }
  }
}
</script>
 <style lang="scss" scoped>
.conter{
  padding: 10px;
  width: 90%;
  height: 600px;
  }
  .header{
  width: 100%;
  height: 10%;
  display: flex;
  border: 1px solid rgb(228, 228, 228);
}

.header-box1{
  flex: 3;
  box-sizing: border-box;
  border: 1px solid rgb(228, 228, 228);
}
.header-box2{
  flex: 7;
  border: 1px solid rgb(228, 228, 228);
  position: relative;
}
.header-box2-son{
  position: absolute;
  top: 50%;
  transform: translate(0,-50%);
  left: 20px;
  font-size: 20px;
}
  .body{
  width: 100%;
  height: 80%;
  display: flex;
  border: 1px solid rgb(228, 228, 228);
  border-top:0;
}
.body-box1{
   width: 30%;
  border: 1px solid rgb(228, 228, 228);
  box-sizing: border-box;
  padding: 10px;
}
.body-box2{
  width: 70%;
  border: 1px solid rgb(228, 228, 228);
  display: flex;
  div:first-child{
    margin: 20px 20px 10px;
  }
  div:nth-child(2){
    margin: 20px;
  }
}
input {
  // outline: none;
  border:1px solid #ccc;
  // background: none;
  width: 150px;
  height: 25px;
  border-radius: 5px;
  padding: 0 10px;
}
</style>
