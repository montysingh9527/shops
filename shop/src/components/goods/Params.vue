<template>
<div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>  
    <!-- 卡片视图区域 -->        
   <el-card class="box-card">
       <!-- 警告文字 -->
       <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
       <!-- 选择商品分类 -->
       <el-row class="cat_opt">
           <el-col>选择商品分类：</el-col>
       </el-row>
    </el-card>
</div>
</template>
<script>
    export default {
        data() {
            return {

            }
        },
        created() {

        },
        methods: {

        }
    }
</script>

<style scoped lang="less">
.cat_opt{
    margin: 20px 0;
}
</style>
