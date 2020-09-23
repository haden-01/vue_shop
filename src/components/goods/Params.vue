<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数管理</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 警告区域 -->
            <el-alert
            title="注意：只允许设为第三级分类设置相关参数！"
            type="warning"
            :closable="false"
            show-icon>
            </el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>
                        选择商品分类:
                    </span>
                        <!-- 级联选择器 -->
                    <el-cascader
                    v-model="selectedCateKeys"
                    :options="catelist"
                    :props="cascaderProps"
                    @change="handleChange" clearable :size="mini">
                    </el-cascader>
                </el-col>
            </el-row>
            <!-- tab 页签区域 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <!-- 添加动态参数的面板 -->
                <el-tab-pane label="动态参数" name="many">
                    <!-- 添加参数的按钮 -->
                    <el-button type="primary" size="mini" :disabled="idBtnDisable" @click="addDialogVisible = true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTabData" border >
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染Tag标签 -->
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClosed(i, scope.row)">{{item}}</el-tag>
                            <!-- 输入文本框 -->
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)"
                                >
                            </el-input>
                            <!-- 添加按钮 -->
                            <el-button v-else class="button-new-tag" 
                            size="small" @click="showInput(scope.row)">+ New Tag</el-button>

                            </template>
                        </el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showAditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>

                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!-- 添加静态属性的面板 -->
                <el-tab-pane label="静态属性" name="only" :disabled="idBtnDisable">
                    <!-- 添加属性的按钮 -->
                <el-button type="primary" size="mini" @click="addDialogVisible = true">添加属性</el-button>
                     <!-- 静态属性表格 -->
                    <el-table :data="onlyTabData" border >
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <!-- 循环渲染Tag标签 -->
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClosed(i, scope.row)">{{item}}</el-tag>
                            <!-- 输入文本框 -->
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)"
                                >
                            </el-input>
                            <!-- 添加按钮 -->
                            <el-button v-else class="button-new-tag" 
                            size="small" @click="showInput(scope.row)">+ New Tag</el-button>

                            </template>
                        </el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showAditDialog(scope.row.attr_id)">修改</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>

                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
             </el-tabs>
        </el-card>

        <!-- 添加参数的对话框 -->
        <el-dialog
            :title="'添加' + titleText"
            :visible.sync="addDialogVisible"
            width="50%" @close="addDialogClosed">
            <!-- 添加参数的表单 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" >
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改参数的对话框 -->
        <el-dialog
            :title="'修改' + titleText"
            :visible.sync="editDialogVisible"
            width="50%" @close="editDialogClosed">
            <!-- 添加参数的表单 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" >
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 商品分类的列表
            catelist: [],
            // 级联选择框的配置对象
            cascaderProps: {
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children',
                    expandTrigger: 'hover'
                    // checkStrictly: 'true'
                },
                // 级联选择框双向绑定到的数组
                selectedCateKeys: [],
                // 被激活的页签的名称
                activeName: 'many',
               // 动态参数的数据
               manyTabData: [],
               // 静态属性的数据
               onlyTabData: [],
               // 控制添加对话框的显示与隐藏
               addDialogVisible: false,
               // 添加参数的双向表单绑定数据
               addForm: {
                   attr_name: ''
               },
               // 添加调单的验证规则对象
               addFormRules: {
                   attr_name: [
                       {required: true, message: '请输入参数名称', trigger: 'blur'}
                   ]
               },
               // 控制修改对话框的显示与隐藏
               editDialogVisible: false,
               // 修改表单的数据对象
               editForm: {},
               // 修改表单的验证规则
               editFormRules: {
                   attr_name: [
                       {required: true, message: '请输入参数名称', trigger: 'blur'}
                   ]
               }
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        // 获取所有商品分类的列表
        async getCateList () {
        const {data: res} = await this.$http.get('categories')
        if (res.meta.status !== 200) { return this.$message.error('请求商品列表失败') }
        this.catelist = res.data
        // console.log(this.catelist)
        },
        // 级联选择框选中对象变化，会出发这个函数
        handleChange() {
            // console.log(this.selectedCateKeys)
            this.getParamsData()
        },
        // 这是tab页标签点击事件的处理函数
        handleTabClick() {
            console.log(this.activeName)
             this.getParamsData()
        },
        // 获取参数列表的数据
        async getParamsData() {
            if (this.selectedCateKeys.length !== 3){
                this.selectedCateKeys = []
                // 防止第一次点击的三级分类信息，在第二次无效点击之后还留存第一次的信息
                this.manyTabData = []
                this.onlyTabData = []
            }
            // 根据选中的Id，和当前的面板，获取对应的参数
            const {data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
                params: {
                    sel: this.activeName
                }
            })
           if (res.meta.status !== 200) { return this.$message.error('获取参数列表失败！') }
           res.data.forEach(item => {
               // 将item.attr_vals里面的字符串根据空格分隔，返回为数组
               // 先对item.attr_vals的数组进行三元表达式进行判断，如果为空就返回为一个空数组，否则就进行正常切割
               item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                // 控制文本框的显示和隐藏
                item.inputVisible = false
                // 文本框中输入值得绑定属性
                item.inputValue = ''
           })
           console.log(res.data)
           // 判断是动态参数many 还是静态属性any
           if (this.activeName === 'many'){
               // 获取到动态参数
               this.manyTabData = res.data
           } else {
               // 获取到静态属性
               this.onlyTabData = res.data
           }
        },
        // 监听添加对话框的关闭事件
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        // 点击按钮，添加参数
        addParams() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) return
                const {data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name: this.addForm.attr_name,
                    attr_sel: this.activeName
                     })
                if (res.meta.status !== 201) { return this.$message.error('添加参数失败！') }
                 this.$message.success('添加参数成功！')
                 this.getParamsData()
                 this.addDialogVisible = false   
            })
        },
        // 点击按钮，展示修改的对话框
        async showAditDialog(id) {
            // 查询当前参数的信息
            const {data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
                params: {
                    attr_sel: this.activeName
                }
            })
            if (res.meta.status !== 200) { return this.$message.error('获取参数失败！') }
            this.editForm = res.data
            this.editDialogVisible = true
        },
        // 表单清空重置
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        // 点击确定，进行预验证，提交编辑用户信息
        editParams() {
            //
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                const {data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
                    attr_name: this.editForm.attr_name,
                    attr_sel: this.editForm.attr_sel
                })
                if (res.meta.status !== 200) { return this.$message.error('修改参数失败！') }
                this.$message.success('修改参数成功！')
                this.getParamsData()
                this.editDialogVisible = false
            })
        },
        // 删除对应的参数项
        async removeParams(id) {
            // 返回一个confirm字符串
            const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
        }).catch(err => err)
        // 用户取消删除操作
        if (confirmResult !== 'confirm') {
            return this.$message.info('已取消删除')
        }
        const {data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
        if (res.meta.status !== 200) { return this.$message.error('删除参数失败！') }
                this.$message.success('删除参数成功！')
                this.getParamsData()
        },
        // 文本框是去焦点，或者摁下Enter键都会触发这个事件
        async handleInputConfirm(row) {
            // row.inputVisible = false
            if (row.inputValue.trim().length === 0) {
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            // 如果没有return 则证明输入的内容需要做后续的处理
            row.attr_vals.push(row.inputValue.trim())
            row.inputValue = ''
            row.inputVisible = false
            // 发起请求提交数据保存到数据库中
           this.saveAttrVals(row)
        },
        // 将对attr_vals的操作保存到数据库
        async saveAttrVals(row){
            const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
                attr_name: row.attr_name,
                attr_sel: row.attr_sel,
                attr_vals: row.attr_vals.join(' ')
            })
            if (res.meta.status !== 200) {
                return this.$message.error('修改参数项失败！')
            }
            this.$message.success('修改参数项成功！')
        },
        // 点击按钮，展示文本输入框
        showInput(row) {
            row.inputVisible = true
            // 让文本框自动获得焦点
            // $nextTick方法得作用 ，就是当页面上元素被重新渲染之后，才会指定回调函数的代码
            this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus()
        })
        },
        // 删除对应的参数可选项
        handleClosed(i, row) {
            // splice(a, b) 删除数组中索引为a,一次删除a(包含a)之后的b个元素
            row.attr_vals.splice(i, 1)
            this.saveAttrVals(row)
        }
    },
    computed: {
        // 如果按被禁用，则返回true,否则赶回false
        idBtnDisable() {
            if (this.selectedCateKeys.length !== 3) {
                return true
            }
            return false
        },
        // 当前选中的三级分类的ID
        cateId() {
            if (this.selectedCateKeys.length === 3) {
                return this.selectedCateKeys[2]
            }
            return null
        },
        // 动态计算标题的文本
        titleText() {
            if (this.activeName === 'many') {
                return '动态参数'
            } 
                return '静态属性'            
        }
    }

}
</script>
<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}
.el-tag {
    margin: 8px;
}
.input-new-tag {
    width: 130px;
}
</style>