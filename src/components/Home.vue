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
        <el-aside :width="isCollapse ? ' 64px':'200px' ">
          <div class="toggle-button" @click="toggleCollapse">
              |||
          </div>
         <el-menu
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#409EFF" :unique-opened = "true" 
      :collapse="isCollapse" :collapse-transition="false"
      :router="true" :default-active="activePath">
      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>

        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" 
        :key="subItem.id" @click="saveNavState('/'+subItem.path)">
          <template slot="title">
        <i class="el-icon-menu"></i>
           <span>{{subItem.authName}}</span>
          </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
        </el-aside>
        <!-- 右侧主体区域 -->
        <el-main>
          <!-- 路由占位符 -->
          <router-view></router-view>
        </el-main>
       </el-container>
    </el-container>
   
</template>

<script>
export default {
  data(){
    return {
      // 左侧菜单数据
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 默认不折叠
      isCollapse: false,
      // 被激活的地址(展示高亮的地址)
      activePath: ''
    }
  },
  created(){
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单
    async getMenuList(){
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      // console.log(res)
    },
    // 点击按钮，切换菜单与展开
    toggleCollapse (){
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
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
  .el-menu{
    border-right: none;
  }
}

.el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin: 10px;
}

.toggle-button{
  background-color: #4A506A;
  font-size: 10px;
  line-height: 24px;
  color: #ffffff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}

</style>
