<template>
  <el-container class="home-height">
    <!-- 头部区域 -->
  <el-header>
    <div>
      <img src="../assets/home.png">
      <span>后台管理系统</span>
    </div>
    <el-button type="info" @click="logout" class="logout">退出</el-button>
  </el-header>
   <!-- 页面主体区域 -->
  <el-container>
     <!-- 侧边栏-->
    <el-aside :width="isCollapse ? '64px' : '200px'">
      <div class="toggle-button" @click="toggleCollapse" v-if="isCollapse">》》》</div>
      <div class="toggle-button" @click="toggleCollapse" v-else-if="!isCollapse">《《《</div>
       <!-- unique-opened设置一级菜单折叠栏只展开当前项默认为true ；collapse-transition折叠动画;
        router启用该模式会在激活导航时以 index 作为 path 进行路由跳转-->
      <el-menu background-color="#545c64" text-color="#fff" active-text-color="#29ACDB" 
      unique-opened 
      :collapse="isCollapse" 
      :collapse-transition="false" 
      router
      :default-active="activePath">
        <!-- 一级菜单 index值只能是字符串-->
      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
        <!-- 一级菜单的模板区域 -->
        <template slot="title">
          <!-- 图标 设置一级菜单字体图标-->
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
        <el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children" :key="subitem.id"
         @click="saveNavState('/' + subitem.path)">
          <template slot="title">
            <i class="el-icon-menu"></i>
            <span>{{subitem.authName}}</span>
            </template>
        </el-menu-item>
      </el-submenu>      
    </el-menu>
    </el-aside>
    <!-- 右侧内容主体 -->
    <el-main>
      <!-- 路由占位符 -->
        <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>


<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menulist:[],
      // 菜单图标 键:一级菜单id(item.id) 值:字体图标
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      // 是否折叠一级菜单
      isCollapse: false,
      // 被激活二级路由地址
      activePath:''
    };
  },
  created(){
    // 在created阶段请求左侧菜单数据
    this.getMenuList(),
    // 获取二级菜单激活状态
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
   async getMenuList(){
     // 发送请求获取左侧菜单数据
     const {data:res} = await this.$http.get('menus')
     // 将数据存放到menulist列表中
     this.menulist = res.data
     if(res.meta.status!=200) return this.$massage.error(res.meta.msg)
     console.log('菜单列表',res)
    },
    // 折叠一级菜单
    toggleCollapse(){
      this.isCollapse = !this.isCollapse
    },
    // 将二级菜单激活状态保存
    saveNavState(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath = activePath
    }
  },
};
</script>

<style lang="less" scoped>
.home-height{
  height:100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content:space-between;
  padding-left: 30px;
  align-items: center;
  color: #fff;
  font-size: 24px;
  >div{
    display: flex;
    align-items: center;
    span{
      margin-left: 40px;
    }
  }
}
.el-aside{
  background-color: #B9D0E2;
  .el-menu{
    border-right:none;
  }
}
.el-main{
  background-color: #F4F4F4;
}
// 字体图标和一级菜单间距
.iconfont{
  margin-right: 10px;
}
// 折叠一级菜单
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
