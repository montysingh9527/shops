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
           <el-col>选择商品分类：
               <!-- 选择商品分类级联选择器 -->
               <el-cascader expand-trigger="hover" v-model="cateKeys" :options="cateList" :props="cateProps"  @change="handleChange"></el-cascader>
           </el-col>       
       </el-row>
    </el-card>
</div>
</template>
<script>
    export default {
        data() {
            return {
                // 商品分类列表
                cateList: [],
                // 级联选择框的配置对象
                cateProps: {
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children'
                },
                // 级联选择框双向绑定到的数组
                cateKeys: []
            }
        },
        created() {
            this.getCateList()
        },
        methods: {
            getCateList(){
                this.$http.get('/categories').then(res=>{
                   this.cateList = res.data.data
                   console.log('cat',this.cateList)
                })
            },
            handleChange(){
                // 证明选中的不是三级分类
                console.log(this.cateKeys)
                if (this.cateKeys.length !== 3) {
                    this.cateKeys = []
                    return
                }
            }
        }
    }
</script>

<style scoped lang="less">
.cat_opt{
    margin: 20px 0;
}
</style>
