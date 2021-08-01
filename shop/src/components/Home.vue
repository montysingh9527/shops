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
     <!-- 侧边栏 -->
    <el-aside width="200px">
      <el-menu background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b">
        <!-- 一级菜单 -->
      <el-submenu index="1">
        <!-- 一级菜单的模板区域 -->
        <template slot="title">
          <!-- 图标 -->
          <i class="el-icon-location"></i>
          <span>导航一</span>
        </template>
        <el-menu-item index="1-4-1">
          <template slot="title">
            <i class="el-icon-location"></i>
            <span>导航一</span>
            </template>
        </el-menu-item>
      </el-submenu>      
    </el-menu>
    </el-aside>
    <!-- 右侧内容主体 -->
    <el-main>Main</el-main>
  </el-container>
</el-container>
</template>


<script>
export default {
  data() {
    return {};
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
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
}
.el-main{
  background-color: #F4F4F4;
}
</style>
