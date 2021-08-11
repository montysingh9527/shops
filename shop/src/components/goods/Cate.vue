<template>
<div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>  

    <!-- 卡片视图区域 -->        
    <el-card class="box-card">
        <el-row>
            <el-col>
                <el-button type="primary">添加分类</el-button>                
            </el-col>
        </el-row>
        <!-- 树形结构插件vue-table-with-tree-grid -->
        <tree-table :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" :show-row-hover="false" border show-index>

        </tree-table>
    </el-card>            
</div>
</template>

<script>
export default {
    data(){
        return{
            // 商品分类数据
            cateList:[],
            // 查询参数
            queryInfo:{
                type:3,
                pagenum:1,
                pagesize:5
            },
            // 总数据条数
            total: 0,
            // 为tree-table设置属性
            columns: [{
                label: '分类名称',
                prop: 'cat_name',
            }]   
        }
    },

    created(){
       this.getCateList()
    },
    methods:{
       getCateList(){
           this.$http.get('categories',{params:this.queryInfo}).then((res)=>{
                console.log('res',res.data.data.result)
                // 把数据列表，赋值给 catelist
                this.cateList = res.data.data.result
                // 为总数据条数赋值
                this.total = res.data.total            
           }).catch(()=>{
               this.$message.error('获取数据失败')
           })
       }
    }

}     

</script>

<style lang="less" scoped>
.el-button{
    margin-bottom:10px;
}
</style>