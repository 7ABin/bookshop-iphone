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
        <div class="header-box2-son">菜单详情</div>
        </div>
    </div>
    <div class="body">
      <div class="body-box1">
        <!-- <div v-for="(item, index) in list"
        :key="index"
        class="body-box1-son"
        @click="change(item)"
        @mouseenter="active(index)"
        @mouseout="leave(index)" >{{item.label}}</div> -->
        <el-tree
          :data="list"
          :props="defaultProps"
          @node-click="changebox">
        </el-tree>
      </div>

      <div class="body-box2">
        <div class="body-box2-body">
        <div>
          <span>菜单名称：</span>
          <input type="text" v-model="menuName" readonly="readonly">
        </div>
        <div>
          <span>路径：</span>
          <input type="text" v-model="menuRoute" readonly="readonly">
        </div>
        </div>
        <div>
          <span>备注：</span>
          <input type="text" v-model="menuNote" readonly="readonly">
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
          <el-form-item label="名称" prop="menuName" @input="change($event)">
          <el-input v-model="dialogFormData.menuName"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <el-form-item label="路径" prop="menuRoute">
          <el-input v-model="dialogFormData.menuRoute"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="24">
          <el-form-item label="备注" prop="menuNote">
          <el-input v-model="dialogFormData.menuNote"></el-input>
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
import req from '@/api/menu-manage'
export default {
  name: 'menu-manage',
  components: {
    ITable,
    IDialog
  },
  data () {
    return {
      diatype: '',
      dialogFormData: {
        menuName: '',
        isMenu: '0',
        lastLevelId: '0',
        menuRoute: '',
        menuNote: ''
      },
      dialogvisible: false,
      dialogTitle: '', // 弹出框头
      dialogwidth: 30,
      menuName: '',
      menuRoute: '', // 菜单路径
      menuNote: '',
      isMenu: '',
      menuId: '',
      verson: '',
      dataparent: '',
      dialogMenu: {},
      formRules: {
        menuName: [
          { required: true, message: '请输入菜单名字', trigger: 'blur' }
        ],
        isMenu: [
          { required: true, message: '请选择是否为菜单', trigger: 'change' }
        ]
      },
      toolbar: [
        {
          btnName: '新增',
          type: 'primary',
          func: () => {
            this.dialogvisible = true
            this.dialogTitle = '新增菜单'
            this.diatype = 'create'
          }
        },

        {
          btnName: '修改',
          type: 'primary',
          func: () => {
            if (this.isMenu === '是') {
              this.isMenu = '0'
            }
            if (this.isMenu === '否') {
              this.isMenu = '1'
            }
            if (this.menuId) {
              this.dialogvisible = true
              this.dialogTitle = '修改菜单'
              this.diatype = 'getup'
              req('findMenuById', {menuId: this.menuId}).then(data => {
                this.dialogFormData = data.data
              })
            } else {
              this.$message('请选择修改的文件')
            }
          }
        },
        {
          btnName: '删除',
          type: 'primary',
          func: () => {
            if (this.menuId) {
              this.$confirm('是否确认删除' + this.menuName).then(() => {
                req('deleteMenu', {menuId: this.menuId}).then(data => {
                  if (data.code === 0) {
                    this.$message.success(data.msg)
                    location.reload()
                  } else {
                    this.$message.error(data.msg)
                  }
                })
              }).catch(() => {})
            } else {
              this.$message.error('请选择删除的管理')
            }
          }
        },
        {
          btnName: '刷新',
          type: 'primary',
          func: () => {
            location.reload()
          }
        }
      ],
      list: [],
      defaultProps: {
        label: 'label'
      }
    }
  },
  created () {
    this.frash()
  },
  // watch: {
  //   'dialogFormData.menuName': {
  //     handler (value) {
  //       console.log(value)
  //     }
  //   }
  // },
  methods: {
    change (e) {
      this.$forceUpdate()
    },
    frash () {
      req('listMenus', {}).then(data => {
        // console.log('list', data)
        this.list = data.data.rootTree
      })
    },
    changebox (data) {
      this.menuId = data.attributes.menuId // 记录menuid
      this.verson = data.attributes.version //
      // console.log('version', this.verson)
      req('findMenuById', {menuId: data.attributes.menuId}).then(data => {
        // console.log(data, '---')
        this.menuName = data.data.menuName
        this.menuRoute = data.data.menuRoute
        this.menuNote = data.data.menuNote
        this.isMenu = data.data.isMenu
        if (this.isMenu === '0') {
          this.isMenu = '是'
        }
        if (this.isMenu === '1') {
          this.isMenu = '否'
        }
      })
    },
    dialogCancel () {
      this.$refs.form.resetFields()
      this.dialogvisible = false
    },
    dialogBeforeClose () {
      // console.log('<---->')
      this.$refs.form.resetFields()
    },
    dialogConfirm () {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.diatype === 'create') {
            console.log(this.dialogFormData.menuName, '...this.dialogFormData')
            req('addMenu', {...this.dialogFormData, lastLevelId: '0'}).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg) // 发送成功
                this.frash()
                this.$refs.form.resetFields()
                this.dialogvisible = false
              } else {
                this.$message.error(data.msg)
              }
            })
          } else {
            // console.log(this.menuId, '{------------}')
            req('updateMenuById', {
              menuId: this.menuId,
              ...this.dialogFormData,
              version: this.verson}).then(data => {
              if (data.code === 0) {
                this.$message.success(data.msg + '请刷新页面') // 发送成功
                this.frash()
                req('findMenuById', {menuId: this.menuId}).then(data => {
                  // console.log(data, '---')
                  this.menuName = data.data.menuName
                  this.menuRoute = data.data.menuRoute
                  this.menuNote = data.data.menuNote
                  this.isMenu = data.data.isMenu
                  if (this.isMenu === '0') {
                    this.isMenu = '是'
                  }
                  if (this.isMenu === '1') {
                    this.isMenu = '否'
                  }
                })
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
    active (value) {
      document.getElementsByClassName('body-box1-son')[value].style.backgroundColor = '#f2f2f2'
    },
    leave (value) {
      document.getElementsByClassName('body-box1-son')[value].style.backgroundColor = '#fff'
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
   height: 480px;
  border: 1px solid rgb(228, 228, 228);
  position: relative;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  padding: 10px;
  overflow-y: auto;
  position: relative;
    div {
    height: 30px;
    width: 100%;
    top: 10px;
    box-sizing: border-box;
    padding-left: 10px;
    // position: absolute;
  }
}
.body-box2{
  width: 70%;
  border: 1px solid rgb(228, 228, 228);
  .body-box2-body {
    display: flex;
    margin: 5%;
    // justify-content: space-around;
    padding: 10px;
  }
  div:nth-child(2) {
    margin-left: 10%;
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
