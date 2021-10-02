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
       <!-- 动态参数、静态属性tabs -->
       <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <el-tab-pane label="动态参数" name="only">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
                <!-- 动态参数表格 -->
                <el-table :data="onlyData" border stripe> 
                    <!-- 展开行 -->
                    <el-table-column type="expand"></el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column> 
                    <el-table-column prop="attr_name" label="参数名称"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button size="mini" type="primary" icon="el-icon-edit">修改</el-button>
                            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                        </template>
                    </el-table-column>                    
                </el-table>
            </el-tab-pane>
            <el-tab-pane label="静态属性" name="many">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
                <!-- 静态属性表格 -->
                <el-table :data="manyData" border stripe> 
                    <!-- 展开行 -->
                    <el-table-column type="expand"></el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column> 
                    <el-table-column prop="attr_name" label="属性名称"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button size="mini" type="primary" icon="el-icon-edit">修改</el-button>
                            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                        </template>
                    </el-table-column>                    
                </el-table>
            </el-tab-pane>
        </el-tabs>
    </el-card>
    <!-- 添加参数和属性弹窗 -->
    <el-dialog :title="'提示' + titleText" :visible.sync="addDialogVisible"  width="30%" @close="addDialogClosed">        
    <el-form :model="addForm" ref="addFormRef" label-width="100px">
    <el-form-item :label="titleText" prop="attr_name">
        <el-input v-model="addForm.attr_name"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialogVisible = false">确 定</el-button>
    </span>
    </el-dialog>
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
                cateKeys: [],
                // 动静态参数选中项[only,many]
                activeName:'only',
                // 动态数据
                onlyData:[],
                // 静态数据
                manyData:[],
                // 动静态属性的对话框
                addDialogVisible:false,
                // 添加参数的表单数据对象
                addForm: {
                    attr_name: ''
                },
            }
        },
        created() {
            this.getCateList()
        },
        methods: {
            getCateList(){
                this.$http.get('/categories').then(res=>{
                   this.cateList = res.data.data
                //    console.log('cat',this.cateList)
                })
            },
            // 商品分类选择
            handleChange(){
                this.getParamsData()
            },
            // 动静态参数tabs选中切换
            handleTabClick(){
                this.getParamsData()
            },
            // 监听添加参数和属性对话框的关闭事件
            addDialogClosed() {
            this.$refs.addFormRef.resetFields()
            },

            // 获取参数函数
            getParamsData(){
                // 证明选中的不是三级分类,清空表格
                // console.log(this.cateKeys)
                if (this.cateKeys.length !== 3) {
                    this.cateKeys = []
                    this.onlyData = []
                    this.manyData = []
                    return
                }else{
                    // 根据所选分类的Id，和当前所处的面板，获取对应的参数
                    this.$http.get(`categories/${this.idCate}/attributes`,{
                        params:{sel:this.activeName}
                    }).then(res=>{
                        // console.log(res.data.data) 
                        // 根据activeName 判断是静态还是动态
                       if(this.activeName=='only'){
                           this.onlyData = res.data.data
                       }else{
                           this.manyData = res.data.data
                       }
                    })
                }
            }
        },
        computed: {
            // 通过计算属性判断添加属性按钮是否启用
            isBtnDisabled(){
                if(this.cateKeys.length!==3){
                    return true
                }else{
                    return false
                }
            },
            idCate(){
                if(this.cateKeys.length === 3){
                    return this.cateKeys[2]
                }else{
                    return null
                }
            },
            titleText(){
                if(this.activeName=='only'){
                    return '动态参数'
                }else{
                    return '静态属性'
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
