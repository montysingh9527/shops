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
    <!-- 编辑角色权限弹窗 -->
    <el-dialog title="修改角色" :visible.sync="editdialogVisibleRoles" width="30%">
    <el-form :model="editRolesForm" label-width="100px">
            <el-form-item label="角色名称：">
                <el-input v-model="editRolesForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述：">
                <el-input v-model="editRolesForm.roleDesc"></el-input>
            </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisibleRoles = false">取 消</el-button>
        <el-button type="primary" @click="editRolesName">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 卡片视图区域 -->        
    <el-card class="box-card">
        <el-button type="primary" @click="dialogVisibleRoles = true">添加角色</el-button>
        <el-table :data="rolesList" border stripe>
            <el-table-column type="expand">
                <template slot-scope="scope">
                <!-- <pre>{{scope.row}}</pre>  -->
                    <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级和三级权限 -->
                        <el-col :span="19">
                            <!-- 通过 for 循环 嵌套渲染二级权限 -->
                            <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type="index" index-text="#"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <!-- {{scope.row}}
                    {{scope.row.roleName}} -->
                    <!-- 修改按钮 -->
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="editRoles(scope.row.id)">编辑</el-button>
                    <!-- 删除按钮 -->
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRoles(scope.row.id,scope.row.roleName)">删除</el-button>
                    <!-- 分配角色按钮 -->
                    <el-tooltip effect="dark" content="分配权限" placement="top" :enterable="false">
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </el-tooltip>                       
                </template>
            </el-table-column>
        </el-table>
    </el-card>    

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="40%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>           
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
            dialogVisibleRoles:false,
            // 编辑用户查询到的用户信息
            editRolesForm: {},
            // 编辑角色弹窗
            editdialogVisibleRoles:false,
            // 分配权限树形弹窗的显示与隐藏
            setRightDialogVisible:false,
            // 所有权限的数据
            rightslist: [],
            // 树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的节点Id值数组
            defKeys: [],
            // 当前即将分配权限的角色id
            roleId: ''
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
        },
    // 编辑用户按钮
    async editRoles(rolesId){
        // console.log(rolesId)
        const {data:res} = await this.$http.get('roles/' + rolesId)
        // console.log(res)  
        if(res.meta.status != 200){
            return this.$message.error('获取数据失败')
        }
        this.editRolesForm = res.data
        this.editdialogVisibleRoles= true             
        },
    // 提交编辑用户信息 路径：roles/:id  请求方式：put
    editRolesName(){
            this.$http.put('roles/' + this.editRolesForm.roleId,{
               roleName:this.editRolesForm.roleName,
               roleDesc:this.editRolesForm.roleDesc
            }).then(()=>{
                this.$message.success('编辑用户信息成功！')
                this.editdialogVisibleRoles=false,
                this.getRolesList()
             })
        },
    //删除角色 请求路径：roles/:id 请求方法：delete
    deleteRoles(rolesId,roleName){        
       this.$confirm(`是否【${roleName}】删除该角色！`,'提示', {           
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http.delete('roles/' + rolesId).then(()=>{
            this.$message.success('删除成功')
            this.getRolesList()
          })         
        }).catch((err => err)
        )
    },
    // 根据Id删除对应的权限
    async removeRightById(role, rightId) {
        // 弹框提示用户是否要删除
        const result = await this.$confirm(
            '是否删除该权限？',
            '提示',
            {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }
        ).catch(err => err)
        if (result !== 'confirm') {
            return this.$message.info('取消了删除！')
        }
        // 请求路径：roles/:roleId/rights/:rightId  - 请求方法：delete
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
            return this.$message.error('删除权限失败！')
        }
        role.children = res.data
    },

    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      // 保存id到roleId中
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }
      // 把获取到的权限数据保存到 data 中
      this.rightslist = res.data
      // console.log(this.rightslist)
      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 定义函数 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件,关闭弹窗将树形结构选中项清空
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        // setCheckedKeys() 通过 keys 设置目前勾选的节点,使用此方法必须设置 node-key 属性
        // getHalfCheckedKeys() 若节点可被选择（即show-checkbox为true),则返回目前半选中的节点的 key 所组成的数组
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      //  console.log(idStr)
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  
}     
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>