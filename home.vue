<template>
  <div class="home-container">
  <el-container>
    <el-header height="70px">
      <router-link to="/home" class="logo-btn">行走书店管理系统</router-link>

      <div class="user-info">
        <el-popover
          placement="bottom"
          trigger="hover"
          width="150px"
          >
          <el-button type="type" @click="outlogin" size="medium">退出登录</el-button>
          <div slot="reference">
             <img :src='photo'>
             <span>{{userAccount}}</span>

          </div>
        </el-popover>
      </div>
    </el-header>

  <el-container>
    <el-aside width="200px">
    <el-tree height= '200px'
      :data="menuList"
      :props="defaultProps"
      @node-click="jump">
    </el-tree>
    </el-aside>
    <el-main>
      <router-view></router-view>
    </el-main>
  </el-container>
  </el-container>
  </div>
</template>
<script>
import req from '@/api/menu-manage'
export default {
  name: 'home',
  data () {
    return {
      menuList: [],
      defaultProps: {
        // children: 'children',
        label: 'label'
      },
      photo: '',
      userAccount: '',
      visiable: false
    }
  },
  created () {
    req('listUserMenus', {}).then(data => {
      // console.log('listUserMenus', data)
      this.menuList = data.data
    })
    req('findInformationById', {}).then(data => {
      this.photo = data.data.photo
      this.userAccount = data.data.userAccount
    })
  },
  methods: {
    handlerSeclect (key, path) {
      if (key !== this.$route.path) {
        this.$router.push({path: key})
      }
    },
    jump (value) {
      // console.log(value)
      if (value.id === '1') {
        this.$router.push({path: '/user-manage'})
      }
      if (value.id === '2') {
        this.$router.push({path: '/menu-manage'})
      }
      if (value.id === '3') {
        this.$router.push({path: '/comm-manage'})
      }
      if (value.id === '4') {
        this.$router.push({path: '/home-pic-manage'})
      }
      if (value.id === '5') {
        this.$router.push({path: '/comm-classify-manage'})
      }
      if (value.id === '6') {
        this.$router.push({path: '/client-manage'})
      }
      if (value.id === '7') {
        this.$router.push({path: '/order-manage'})
      }
      if (value.id === '8') {
        this.$router.push({path: '/hot-comm-manage'})
      }
      if (value.id === '9') {
        this.$router.push({path: '/shop-info-manage'})
      }
      if (value.id === '10') {
        this.$router.push({path: '/driver-info-manage'})
      }
    },
    active (value) {
      document.getElementsByClassName('divbox')[value].style.backgroundColor = '#ccc'
    },
    leave (value) {
      document.getElementsByClassName('divbox')[value].style.backgroundColor = '#fff'
    },
    outlogin () {
      sessionStorage.clear('userInfo')

      sessionStorage.clear('roleInfo')
      sessionStorage.clear('storeInfo')

      this.$router.push({path: 'login'})
    }
  }
}
</script>
<style lang="scss" scoped>
.home-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}
  .el-header{
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgb(243, 243, 243);
  .router-link-active {
  font-size: 25px;
  color:#409EFF;
  text-decoration: none;
}
  .user-info{
  display: flex;
  align-items: center;
  // width: 125px;
  height: 100%;
  margin-right: 10px;
  justify-content: space-between;
  img {
    width: 58px;
    height: 60px;
    border-radius: 50%;
    margin-right: 10px;
    vertical-align: middle;
  }
    /deep/ .el-popover__reference {
      display: flex;
      justify-content: flex-end;
      align-items: center;
    }
}
}

.el-main {
  padding: 0;
}
.el-aside {
  border-right: 1px solid rgb(224, 224, 224);
  .el-tree {
    // background-color: #aaa;
    margin-top: 10px;
    /deep/ .el-tree-node {
      height: 50px;
      span {
        font-size: 17px;
      }
    }
  }
}
.logo-btn {
  font-size: 25px;
    color: #409EFF;
    text-decoration: none;
}
</style>
