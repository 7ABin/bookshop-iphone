<template>
  <div class="bodylogin">
    <div class="windowlogin">
      <div class="head">请登录</div>
      <el-input v-model="userAccount" placeholder="请输入用户名">{{userAccount}}</el-input>
      <el-input type="password" v-model="userPassword" placeholder="请输入密码">{{userPassword}}</el-input>
      <el-button type="primary" @click="send">登录</el-button>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import req from '@/api/user-manage'
const qs = require('qs')
export default {
  data () {
    return {
      userAccount: '',
      userPassword: ''
    }
  },
  methods: {
    send () {
      axios({
        method: 'post',
        url: 'http://39.98.76.3:8080/uaa/auth/form',
        data: qs.stringify({
          username: this.userAccount,
          password: this.userPassword
        }),
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }).then(res => {
        // console.log(res)
        if (res.data.code === 0) {
          this.$message({
            type: 'success',
            message: res.data.msg
          })
          sessionStorage.setItem('userInfo', JSON.stringify(res.data.data))
          req('findInformationById', {}).then(data => {
            // console.log(data, 'findInformationById')
            sessionStorage.setItem('storeInfo', JSON.stringify(data.data))
          })
          // console.log(res.data.data.role, '11111111111')
          this.$router.push({path: '/home'})
        } else {
          this.$message({
            type: 'error',
            message: res.data.msg
          })
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.bodylogin{
  width: 100%;
  height: 100%;
  // background-color: saddlebrown;
  background-image: url('../assets/u0.jpg');
  background-repeat: no-repeat;
   background-size: 100% 100%;
   position: relative;
}
.windowlogin{
  width: 300px;
  height: 300px;
  background-color: rgba(288,288,288,.2);
  display: flex;
  flex-direction: column;
  align-self: center;
  justify-content: center;
  position: absolute;
  top: 50%;
  transform: translate(0,-50%);
  right: 50px;
  box-sizing: border-box;
  padding: 20px;
}
.head {
  height: 30px;
  text-align: center;
  margin-bottom: 20px;
  font-size: 16px;
  color: rgb(204, 202, 202);
}
.el-input {
  width: 80%;
  left: 50%;
  transform: translate(-50%);
  margin-bottom: 30px;
  border-radius: 50%;
  // height: 30px;
  /deep/ .el-input__inner{
    border-radius: 14px;
    height: 35px;
  }
}
.el-button {
  width: 50%;
  position: relative;
    left: 50%;
  transform: translate(-50%);
}
</style>
