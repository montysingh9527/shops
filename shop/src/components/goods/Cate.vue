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
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" style="color: #39CE39" v-if="scope.row.cat_deleted == false"></i>
                <i class="el-icon-error" style="color: #AA1C15" v-else></i>
            </template>
            <template slot="sort" slot-scope="scope">
                <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
                <el-tag type="success" v-else-if="scope.row.cat_level === 1" size="mini">二级</el-tag>
                <el-tag type="danger" size="mini" v-else>三级</el-tag>
            </template>
            <template slot="operate" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>            
        </tree-table>
        <!-- 分页区域 -->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" 
        :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 20, 50]" :page-size="queryInfo.pagesize" 
        layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
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
            },
            {
            label: '是否有效',
            type: 'template',
            template: 'isok',
            },
            {
            label: '排序',
            type: 'template',
            template: 'sort',
            },
            {
            label: '操作',
            type: 'template',
            template: 'operate',
            }            
            ]   
        }
    },

    created(){
       this.getCateList()
    },
    methods:{
       getCateList(){
           this.$http.get('categories',{params:this.queryInfo}).then((res)=>{
                // console.log('res',res.data.data.result)
                // 把数据列表，赋值给 catelist
                this.cateList = res.data.data.result
                // 为总数据条数赋值
                this.total = res.data.data.total     
           }).catch(()=>{
               this.$message.error('获取数据失败')
           })
       },
        // 分页器 监听 pagesize 改变的事件
        handleSizeChange(val){
            this.queryInfo.pagesize = val
            this.getCateList()
        },
        // 分页器 监听 页码值 改变的事件
        handleCurrentChange(val){
            this.queryInfo.pagenum = val
            this.getCateList()
        }
    }

}     

</script>

<style lang="less" scoped>
.el-button{
    margin-bottom:10px;
}
</style>