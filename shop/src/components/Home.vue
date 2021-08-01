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
      active-text-color="#29ACDB">
        <!-- 一级菜单 index值只能是字符串-->
      <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
        <!-- 一级菜单的模板区域 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
        <el-menu-item :index="subitem.id + ''" v-for="subitem in item.children" :key="subitem.id">
          <template slot="title">
            <i class="el-icon-menu"></i>
            <span>{{subitem.authName}}</span>
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
    };
  },
  created(){
    // 在created阶段请求左侧菜单数据
    this.getMenuList()  
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
     console.log(res)
     console.log(this.menulist)
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
}
.el-main{
  background-color: #F4F4F4;
}
// 字体图标和一级菜单间距
.iconfont{
  margin-right: 10px;
}
</style>
