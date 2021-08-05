<template>
    <div>
         <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card class="box-card">
           <el-table :data="rightsList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="权限名称" prop="authName"></el-table-column>
                <el-table-column label="路径" prop="path"></el-table-column>
                <el-table-column label="权限等级">
                    <template slot-scope="scope">
                        <!-- {{scope.row.level}} -->
                        <el-tag v-if="scope.row.level==='0'">一级</el-tag>
                        <el-tag v-else-if="scope.row.level==='1'" type="success">二级</el-tag>
                        <el-tag v-else type="danger">三级</el-tag>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>            
    </div>
</template>

<script>
export default {
    data(){
        return{
            // 权限列表数据
            rightsList: []
        }
    },
    created(){
        // 在页面渲染前加载数据
        this.getRightsList()
    },
    methods:{
        // 获取数据 请求路径 rights/:type  值ist 或 tree,list列表显示权限,tree树状显示权限
      async getRightsList(){
         const {data:res} = await this.$http.get('rights/list')
        //  console.log(res)
         if(res.meta.status != 200){
             return this.$message.error('获取权限列表失败！')
         }
         this.rightsList = res.data
      }
    }
}
</script>

<style lang="less" scoped>

</style>