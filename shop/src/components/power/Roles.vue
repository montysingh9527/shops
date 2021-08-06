<template>
<div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>  
    <!-- 添加角色弹窗 -->
    <el-dialog title="添加角色" :visible.sync="dialogVisibleRoles" width="30%" @close="handleCloseRoles">
    <el-form :model="rolesInfo" label-width="100px">
            <el-form-item label="角色名称：">
                <el-input v-model="rolesInfo.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述：">
                <el-input v-model="rolesInfo.roleDesc"></el-input>
            </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisibleRoles = false">取 消</el-button>
        <el-button type="primary" @click="postRolesInfo">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 卡片视图区域 -->        
    <el-card class="box-card">
        <el-button type="primary" @click="dialogVisibleRoles = true">添加角色</el-button>
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
            // 角色列表-角色名称数据
            rolesList:[],
            // 添加角色
            rolesInfo:{
                roleName:'',
                roleDesc:''
            },
            // 添加角色弹窗
            dialogVisibleRoles:false
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
        // console.log(res)
        if(res.meta.status !=200){
            return this.$message.error('获取角色列表失败！')
        }
        this.rolesList = res.data
      },
      // 添加角色
    async postRolesInfo(){
         // 发起请求添加角色
         const {data:res} =await this.$http.post('roles',this.rolesInfo)
        //  console.log(res)
         if(res.meta.status !=201){
             return this.$message.error('添加用户失败')
         }
         // 添加成功关闭弹窗，刷新页面数据
         this.dialogVisibleRoles = false
         this.getRolesList()
        },
        // 关闭添加角色弹窗，清空表单数据
         handleCloseRoles(){
          this.rolesInfo = {}
        }         
    }
}
</script>

<style lang="less" scoped>

</style>