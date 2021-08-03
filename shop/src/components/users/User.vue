<template>
    <div>
         <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card class="box-card">
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                    <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="6">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
                <!-- 添加用户对话框 -->
                <el-dialog title="添加用户"  :visible.sync="addDialogVisible"  width="40%" >
                    <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="70px">
                        <!-- 用户名 -->
                        <el-form-item label="用户名" prop="username">
                            <el-input v-model="addForm.username" prefix-icon="el-icon-user" placeholder="请输入用户名"></el-input>
                        </el-form-item>  
                        <el-form-item label="密码" prop="password">
                            <el-input v-model="addForm.password" prefix-icon="el-icon-view" placeholder="请输入密码"></el-input>
                        </el-form-item>
                        <el-form-item label="邮箱" prop="email">
                            <el-input v-model="addForm.email" prefix-icon="el-icon-chat-line-square" placeholder="请输入Email邮箱"></el-input>
                        </el-form-item>
                        <el-form-item label="手机号" prop="mobile">
                            <el-input v-model="addForm.ipone" prefix-icon="el-icon-mobile-phone" placeholder="请输入手机号"></el-input>
                        </el-form-item>      
                    </el-form>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="addDialogVisible = false">取 消</el-button>
                        <el-button type="primary" @click="addDialogVisible = false">确 定</el-button>
                    </span>
                </el-dialog>  
            </el-row>            
            <!-- 用户列表区域 -->
            <el-table :data="userlist" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <el-switch  v-model="scope.row.mg_state" @change="changeState(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" >
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                       <el-button type="primary" icon="el-icon-edit" circle></el-button>
                       <!-- 删除按钮 -->
                       <el-button type="danger" icon="el-icon-delete" circle></el-button>
                       <!-- 分配角色按钮 -->
                       <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" circle></el-button>
                        </el-tooltip>                       
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" 
            :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 20]" :page-size="queryInfo.pagesize" 
            layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>              
    </div>
</template>

<script>
export default {
    data(){
        // 自定义邮箱验证规则 在addFormRules 中通过 validator: checkEmail 绑定
        var checkEmail = (rule, value, callback) => {
        // 验证邮箱的正则表达式
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if (regEmail.test(value)) {
            // 合法的邮箱
            return callback()
        }
            callback(new Error('请输入合法的邮箱'))
        }
        // 自定义手机号的验证规则
        var checkMobile = (rule, value, callback) => {
        // 验证手机号的正则表达式
        const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
        if (regMobile.test(value)) {
            return callback()
        }
            callback(new Error('请输入合法的手机号'))
        }

        return{
            // 获取用户列表的参数对象
            queryInfo: {
                query: '',
                // 当前的页数
                pagenum: 1,
                // 当前每页显示多少条数据
                pagesize: 3
            },
            // 管理员列表
            userlist: [],
            // 管理员数据总条数
            total: 0,
            // 控制显示隐藏添加用户对话框
            addDialogVisible:false,
            // 添加用户的表单数据
            addForm:{
                    username:'',
                    password:'',
                    email:'',
                    mobile:''
            },
            // 添加用户表单的验证规则
            addFormRules:{
                 // 验证用户名是否合法
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' },
                    { min: 5, max: 10, message: '长度在 5 到 10 个字符', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 5, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur' }
                ],
                email: [
                    { required: false, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: false, message: '请输入手机号', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ],
            }
        }
    },
    created(){
        // 在页面渲染前加载数据
        this.getUserList()
    },
    methods:{
        // 获取数据
       async getUserList(){
           const {data:res} = await this.$http.get('users',{ params:this.queryInfo})
           if(res.meta.status != 200) { return this.$massage.error('获取管理员数据失败')}
           this.userlist = res.data.users
           this.total = res.data.total
        },
        // 分页器 监听 pagesize 改变的事件
        handleSizeChange(val){
            this.queryInfo.pagesize = val
            // 重新获取数据
            this.getUserList()
        },
        // 分页器 监听 页码值 改变的事件
        handleCurrentChange(val){
            this.queryInfo.pagenum = val
            // 重新获取数据
            this.getUserList()
        },
        // 将用户修改的状态，保存到数据库
        async changeState(scopeRow){
            console.log(scopeRow)
            // 接口地址：users/:uId/state/:type 请求方法：put
           const {data:res} = await this.$http.put(`users/${scopeRow.id}/state/${scopeRow.mg_state}`)
           if(res.meta.status != 200){
               // 如果状态更新失败，按钮重置为原来的
               scopeRow.mg_state = !scopeRow.mg_state
               this.$massage.error('用户状态更新失败')
           }
           this.$message.success('更新用户状态成功！')
        }  
    }
}
</script>

<style lang="less" scoped>

</style>
