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
        <!-- 添加分类按钮区 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 表格区域 -->
           <tree-table
           class="treeTable"
           :data="cateList"
           :columns="columns"
           :selection-type="false"
           :expand-type="false"
           show-index index-text="#" border>
           <!-- 是否有效 -->
            <template slot="isok" slot-scope="scope">
                <i  style="color: lightgreen" class="el-icon-success" v-if="scope.row.cat_deleted === false"></i>
                <i style="color: red" class="el-icon-error" v-else></i>
            </template>
            <!-- 排序 -->
             <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                <el-tag type="warning" size="mini" v-else>三级</el-tag>
            </template>
            <template slot="opt" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeSort(scope.row.cat_id)">删除</el-button>
            </template>
           </tree-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

    <!-- // 添加分类的对话框 -->
    <el-dialog
  title="添加分类"
  :visible.sync="addCateDialogVisible"
  width="50%" @close="addCateDialogClose">
  <!-- 添加分类的表单 -->
  <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" >
  <el-form-item label="分类名称：" prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父级分类" >
      <!-- option 用来指定数据元 -->
    <el-cascader
    v-model="selectedKeys"
    :options="cateList"
    :props="cascaderProps"
    @change="parentCateChange" clearable :size="mini">
    </el-cascader>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
<!-- 点击编辑按钮所弹出来的对话框 -->
<el-dialog
  title="提示"
  :visible.sync="editDialogVisible"
  width="50%"
  :before-close="handleClose">
  <el-form :model="editUser" :rules="addCateFormRules" ref="editCateFormRef" label-width="100px" >
  <el-form-item label="分类名称：" prop="cat_name">
    <el-input v-model="editUser.cat_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editSubmit">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
        // 商品分类的数据列表
        cateList: [],
        // 获取商品列表的参数对象
            queryInfo: {
                type: 3,
                // 当前的页数
                pagenum: 1,
                // 当前每页显示多少条数据
                pagesize:5
            },
            // 总数居条数
            total:0,
            // 为table指定列的定义
            columns: [
                {
                    prop: 'cat_name',
                    label: '分类名称'
                },
                {
                    label: '是否有效',
                    // 将当前列指定为自定义模板
                    type: 'template',
                    // 自定义模板名字为isok
                    template: 'isok'
                },
                {
                    label: '排序',
                    // 将当前列指定为自定义模板
                    type: 'template',
                    // 自定义模板名字为isok
                    template: 'order'

                },
                {
                    label: '操作',
                    // 将当前列指定为自定义模板
                    type: 'template',
                    // 自定义模板名字为isok
                    template: 'opt'

                }
            ],
            // 控制添加分类的对话框的显示与隐藏
                addCateDialogVisible: false,
                // 添加分类的数据表单对象
                addCateForm: {
                    // 将要添加分类的名称
                    cat_name: '',
                    // 父级分类的id
                    cat_pid: 0,
                    // 分类的等级，默认一级
                    cat_level:0
                },
                // 添加分类表单的验证规则对象
                addCateFormRules: {

                    cat_name: [{
                        required: true, 
                        message: '请输入分类名称',
                        trigger: 'blur'
                    }]
                },
                // 父级分类列表
                parentCateList: [],
                // 指定级选择器的配置对象
                cascaderProps: {
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children',
                    expandTrigger: 'hover',
                    checkStrictly: 'true'
                },
                // 选中的父级分类的Id数组
                selectedKeys: [],
                // 显示编辑对话框的关闭与否
                editDialogVisible: false,
                // 编辑用户分类的数据
                editUser: []
        }          
    },
    created() {
        this.getCateList()
    },
    methods: {
        // 获取商品列表
        async getCateList () {
        const {data: res} = await this.$http.get('categories', {
            params: this.queryInfo
        })
        if (res.meta.status !== 200) { return this.$message.error('请求商品列表失败') }
        this.cateList = res.data.result
        // 总数据条数赋值
        this.total = res.data.total
        console.log(this.cateList)
        },
        // 监听pagesize改变的属性
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize
            this.getCateList()
        },
        // 监听pagenum的改变
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage
            this.getCateList()
        },
        // 点击按钮，展示添加分类的按钮
        showAddCateDialog() {
            // 获取父级分类书列表
            this.getParentCateList()
            // 展示对话框
            this.addCateDialogVisible = true
        },
        // 获取父级分类数据列表
        async getParentCateList() {
           const {data: res} = await this.$http.get('categories', {
                params: {
                    type:2
                }
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取父级分类数据失败！')
            }
            console.log(res.data)
            this.parentCateList = res.data
        },
        // 选择项发生变化时触发这个函数 length
        parentCateChange() {
            console.log(this.selectedKeys)
            if (this.selectedKeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]  
                this.addCateForm.cat_level = this.selectedKeys.length         
                } else {
                this.addCateForm.cat_pid = 0
                this.addCateForm.cat_level = 0
            }
        },
        // 点击按钮，添加新的分组
        addCate() {
            // this.parentCateChange()
            console.log(this.addCateForm)
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return
                const {data: res} = await this.$http.post('categories', this.addCateForm)
                if (res.meta.status !== 201) { return this.$message.error('添加分类失败！') }
                this.$message.success('添加分类成功！')
                this.getCateList()
                this.addCateDialogVisible = false
            })
            },
            // 监听对话框的关闭事件，重置数据表单
            addCateDialogClose() {
                this.$refs.addCateFormRef.resetFields()
                this.selectedKeys = []
                this.addCateForm.cat_pid = 0
                this.addCateForm.cat_level = 0
            },
            // 展示编辑对话框
            showEditDialog(editUser) {
                this.editDialogVisible = true
                this.editUser = editUser
                // console.log(this.editUser)
            },
            // 点击提交编辑，
            async editSubmit() {
                const {data: res} = await this.$http.put(`categories/${this.editUser.cat_id}`, {
                        cat_name: this.editUser.cat_name
                })
                if (res.meta.status !== 200) { return this.$message.error('编辑分类名称失败！') }
                this.$message.success('编辑分类名称成功！')
                this.getCateList()
                this.editDialogVisible = false
            },
            // 删除分类的单击事件函数
            async removeSort(id) {
            const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
        }).catch(err => err)
        // 如果用户确认删除，则返回值为字符串 confirm
        // 如果用户取消删除，则返回值为字符串 cancel
        if (confirmResult !== 'confirm') {
            return this.$message.info('已取消删除！')
        }
        // 确认删除的操作
        const {data: res } = await this.$http.delete(`categories/${id}`)
        if (res.meta.status !== 200) { return this.$message.error('删除分类失败！') }
        this.$message.success('删除分类成功！')
        this.getCateList()
         }
    }
 }
</script>

<style lang="less" scoped>
.treeTable{
    margin-top:15px
}
.el-cascader{
    width:100%
}

</style>>


