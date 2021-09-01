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
                <el-button type="primary" @click="addCate">添加分类</el-button>                
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
        <!-- 添加分类弹窗 -->
        <el-dialog title="添加商品分类" :visible.sync="addCateDialog" width="30%" @close="addCateDialogClosed">
            <el-form :model="addCateForm" label-width="100px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类">
                    <!-- options 用来指定数据源 -->
                    <!-- props 用来指定配置对象 -->
                    <el-cascader  :options="parentCateList" :props="cascaderProps" v-model="selectedkeys" @change="parentCateChanged" clearable>
                    </el-cascader>  
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialog = false">取 消</el-button>
                <el-button type="primary" @click="addCatePost">确 定</el-button>
            </span>
        </el-dialog>
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
            ],
            // 商品分类是否弹窗
            addCateDialog:false,
            // 商品分类数据
            // 添加分类的表单数据对象: 父级分类Id、名称、等级
            addCateForm: {
                cat_pid: 0,
                cat_name: '',                
                cat_level: 0
            },
            // 父级分类列表
            parentCateList:[],
            cascaderProps: {
                value: 'cat_pid',
                label: 'cat_name',
                children:'children',
            },
            // 选中的父级分类的Id数组
            selectedkeys:[]
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
        },
        // 添加分类弹窗
        addCate(){
            // 在弹窗打开前请求数据
            this.getParentCateList()
            this.addCateDialog = true
        },
        // - 请求路径：categories - 请求方法：get  type[1,2,3] 值：1，2，3 分别表示显示一层二层三层分类列表
        async getParentCateList(){
            const { data: res } = await this.$http.get('categories', {
            params: { type: 2 }
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取父级分类数据失败！')
            }
            // console.log(res.data)
            this.parentCateList = res.data
        },
        // 级联选择器发生变化触发这个函数
        parentCateChanged() {
        // console.log(this.selectedKeys)
            // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
            // 反之，就说明没有选中任何父级分类
            if (this.selectedKeys.length > 0) {
                // 父级分类的Id
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]                // 为当前分类的等级赋值
                this.addCateForm.cat_level = this.selectedKeys.length
            } else {
                // 父级分类的Id
                this.addCateForm.cat_pid = 0
                // 为当前分类的等级赋值
                this.addCateForm.cat_level = 0
            }
        },
        // 点击按钮，添加新的分类
        addCatePost() {
            this.$http.post('categories',this.addCateForm).then(()=>{
                this.$message.success('添加分类成功！')
                this.getCateList()
                this.addCateDialog = false
            }).catch(()=>{
                this.$message.error('添加分类失败！')
            })
        },
        // 监听对话框的关闭事件，重置表单数据
        addCateDialogClosed() {
            // console.log(this.addCateForm)
            this.addCateForm = {}
            this.selectedKeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0
        },
      
    },
}     

</script>

<style lang="less" scoped>
.el-button{
    margin-bottom:10px;
}
.el-cascader{
    width: 100%;
}
</style>