<template>
        <el-container class="home-container">
          <!-- 头部区域 -->
       <el-header>
         <div>
           <img src="../assets/heima.png">
           <span>电商后台管理系统</span>
         </div>
         <el-button type="info" @click="logout">退出</el-button>
         </el-header>
         <!-- 页面主体区域 -->
         <el-container>
           <!-- 侧边栏区域 -->
        <el-aside width="200px">
         <el-menu
      default-active="2"
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b">
      <el-submenu index="1">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>导航一</span>
        </template>
        <el-menu-item-group>
          <template slot="title">
        <i class="el-icon-location"></i>
           <span>导航一</span>
          </template>
        </el-menu-item-group>
      </el-submenu>
    </el-menu>
        </el-aside>
        <!-- 右侧主体区域 -->
        <el-main>
          Main
        </el-main>
       </el-container>
    </el-container>
   
</template>

<script>
export default {
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }

}
</script>

<style lang = "less" scoped>
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #ffffff;
  font-size: 20px;
  > div{
    display: flex;
    align-items: center;
    span{
      margin-left: 20px;
    }
  }
}

.home-container{
  height: 100%;
}

.el-aside{
  background-color: #333744;
}

.el-main{
  background-color: #eaedf1;
}
</style>
