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
                    <el-button type="primary" @click="addDialogVisible = true" >添加用户</el-button>
                </el-col>
                <!-- 添加用户对话框 -->
                <el-dialog title="添加用户"  :visible.sync="addDialogVisible"  width="40%" @close="addDialogClosed">
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
                        <el-button type="primary" @click="addUser">确 定</el-button>
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
                       <el-button type="primary" icon="el-icon-edit" circle @click="showEditDialog(scope.row.id)"></el-button>
                       <!-- 删除按钮 -->
                       <el-button type="danger" icon="el-icon-delete" circle @click="removeUserById(scope.row.id)"></el-button>
                       <!-- 分配角色按钮 -->
                       <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" circle @click="setRole(scope.row)"></el-button>
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
        <!--操作列 修改用户的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo">确 定</el-button>
        </span>
        </el-dialog>    
        <!--操作列 分配角色的对话框 -->
        <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
        <div>
            <p>当前的用户：{{userInfo.username}}</p>
            <p>当前的角色：{{userInfo.role_name}}</p>
            <p>分配新角色：
            <el-select v-model="selectedRoleId" placeholder="请选择">
                <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
                </el-option>
            </el-select>
            </p>
        </div>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
        </span>
        </el-dialog>          
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
                    { min: 8, max: 15, message: '长度在 8 到 15 个字符', trigger: 'blur' }
                ],
            },
             // 修改用户对话框的显示与隐藏
            editDialogVisible: false,
            // 修改用户查询到的用户信息对象
            editForm: {},
            // 修改用户表单的验证规则对象
            editFormRules: {
                email: [
                    { required: true, message: '请输入用户邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入用户手机', trigger: 'blur' },
                    { min: 8, max: 15, message: '长度在 8 到 15 个字符', trigger: 'blur' }
                ]
            },
            // 分配角色对话框的显示与隐藏
            setRoleDialogVisible: false,
            // 需要被分配角色的用户信息
            userInfo: {},
            // 所有角色的数据列表
            rolesList: [],
            // 已选中的角色Id值
            selectedRoleId: ''
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
            // console.log(scopeRow)
            // 接口地址：users/:uId/state/:type 请求方法：put
           const {data:res} = await this.$http.put(`users/${scopeRow.id}/state/${scopeRow.mg_state}`)
           if(res.meta.status != 200){
               // 如果状态更新失败，按钮重置为原来的
               scopeRow.mg_state = !scopeRow.mg_state
               this.$massage.error('用户状态更新失败')
           }
           this.$message.success('更新用户状态成功！')
        },
        // 添加用户对话框的关闭事件,关闭弹窗清空表单项 resetFields()方法
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        // 添加新用户点击按钮  valid为true时表单验证通过,发起请求
        addUser() {
        this.$refs.addFormRef.validate(async valid => {
            if (!valid) return
            // 可以发起添加用户的网络请求
            const { data: res } = await this.$http.post('users', this.addForm)
            if (res.meta.status !== 201) {
                this.$message.error('添加用户失败！')
            }
            this.$message.success('添加用户成功！')
            // 隐藏添加用户的弹窗
            this.addDialogVisible = false
            // 重新获取用户列表数据
            this.getUserList()
            })
        },
        // 修改用户资料弹窗
        async showEditDialog(id) {
            // console.log(id)
            const { data: res } = await this.$http.get('users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error('查询用户信息失败！')
            }
            this.editForm = res.data
            this.editDialogVisible = true
            },
            // 监听修改用户对话框的关闭事件,清空表单数据resetFields()
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
        },
        //用户列表-操作列- 修改用户信息并提交
        editUserInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                // 发起修改用户信息的数据请求
                const { data: res } = await this.$http.put('users/' + this.editForm.id,
                {
                    email: this.editForm.email,
                    mobile: this.editForm.mobile
                }
                )
                if (res.meta.status !== 200) {
                    return this.$message.error('更新用户信息失败！')
                }
                // 关闭对话框
                this.editDialogVisible = false
                // 刷新数据列表
                this.getUserList()
                // 提示修改成功
                this.$message.success('更新用户信息成功！')
            })
        },
        // 用户列表-操作-删除用户根据Id删除对应的用户信息
        async removeUserById(id) {
             // 弹框询问用户是否删除数据
            const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?','提示',
                {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }
                // 捕获异常
            ).catch(err =>{
               return err
            })
            // 如果用户确认删除,则返回值为字符串confirm,如果用户取消了删除,则返回值为字符串cancel
            // console.log(confirmResult)
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete('users/' + id)
            if(res.meta.status != 200){
                this.$massage.error('删除用户失败！')
            }
                this.$message.success('删除用户成功')
                // 刷新列表
                this.getUserList()                
            },

            // 用户列表-分配角色 展示分配角色的对话框
            async setRole(userInfo) {
                this.userInfo = userInfo
                // 在展示对话框之前，获取所有角色的列表
                const { data: res } = await this.$http.get('roles')
                if (res.meta.status !== 200) {
                    return this.$message.error('获取角色列表失败！')
                }
                this.rolesList = res.data
                this.setRoleDialogVisible = true
            },
            // 点击按钮，分配角色
            async saveRoleInfo() {
                if (!this.selectedRoleId) {
                    return this.$message.error('请选择要分配的角色！')
                }
                const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`,
                    {
                    rid: this.selectedRoleId
                    }
                )
                if (res.meta.status !== 200) {
                    return this.$message.error('更新角色失败！')
                }
                    this.$message.success('更新角色成功！')
                    this.getUserList()
                    this.setRoleDialogVisible = false
            },
            // 监听分配角色对话框的关闭事件
            setRoleDialogClosed() {
                this.selectedRoleId = ''
                this.userInfo = {}
            }
    }
}
</script>

<style lang="less" scoped>

</style>
