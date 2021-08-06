<template>
    <div>
         <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>  
        <!-- 卡片视图区域 -->        
        <el-card class="box-card">
           <el-button type="primary">添加角色</el-button>
           <el-table :data="rolesList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                       <el-button type="primary" icon="el-icon-edit" size="mini" >编辑</el-button>
                       <!-- 删除按钮 -->
                       <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                       <!-- 分配角色按钮 -->
                       <el-tooltip effect="dark" content="分配权限" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>                       
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
            rolesList:[]
        }
    },
    created(){
        // 在页面渲染前加载数据
        this.getRolesList()
    },
    methods:{
        // 获取数据 请求路径 rights/:type  值ist 或 tree,list列表显示权限,tree树状显示权限
     async getRolesList(){
        const {data:res} = await this.$http.get('roles')
        console.log(res)
        if(res.meta.status !=200){
            return this.$message.error('获取角色列表失败！')
        }
        this.rolesList = res.data
      }
    }
}
</script>

<style lang="less" scoped>

</style>