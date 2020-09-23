<template>
    <div>
        <!--面包屑导航区域 -->
        <el-breadcrumb separator="/">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 添加角色按钮区 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addRole">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 获取角色列表 -->
            <el-table :data="rolelist" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom',i1 === 0 ? 'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id" >
                            <!-- 渲染一级权限 -->
                            <el-col  :span="5">
                                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                                <!-- 渲染二级权限 -->
                                <el-row :class="[i2 === 0 ? '':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col >
                                    <el-col :span="18" >
                                        <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                            </el-tag>
                                    </el-col>
                                    </el-row>                                                              
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" prop="" width="300px">
            <template slot-scope="scope">
             <!-- 修改按钮 -->
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                <!-- 删除按钮 -->
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleByid(scope.row.id)">删除</el-button>
                <!-- 分配角色按钮 -->
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="showsetRightDialog(scope.row)">分配权限</el-button>
                </template>
        </el-table-column>

        </el-table>
        </el-card>
        <!-- 分配权限对话框 -->
        <el-dialog
                    title="分配权限"
                    :visible.sync="setRightDialogVisible "
                    width="50%" @close="setRightDialogClose">
                    <el-tree :data="rightsList" :props="treeProps" 
                    show-checkbox node-key="id" default-expand-all
                    :default-checked-keys="defKeys" ref="treeRef"></el-tree>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="setRightDialogVisible = false">取 消</el-button>
                        <el-button type="primary" @click="allotRights">确 定</el-button>
                    </span>
        </el-dialog>
        <!-- 编辑角色的对话框 -->           
        <el-dialog
            title="编辑角色"
            :visible.sync="editRolesdialogVisible"
            width="50%">
            <el-form :model="editRolesList"  :rules="editRolesRules" label-width="100px">
                <el-form-item label="角色名称:" prop="roleName" >
                    <el-input v-model="editRolesList.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述:" >
                    <el-input v-model="editRolesList.roleDesc" ></el-input>
                    </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editRolesdialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
        </el-dialog>
        <!-- 添加角色对话框 -->
        <el-dialog
            title="添加角色"
            :visible.sync="addRolesdialogVisible"
            width="50%">
            <el-form :model="addRolesList"  :rules="editRolesRules" label-width="100px">
                <el-form-item label="角色名称:" prop="roleName" >
                    <el-input v-model="addRolesList.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述:" >
                    <el-input v-model="addRolesList.roleDesc" ></el-input>
                    </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addRolesdialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAddRole">确 定</el-button>
            </span>
        </el-dialog>


</div>
</template>

<script>
export default {
    data() {
        return {
            // 所有角色列表数据
            rolelist: [],
            // 控制分配权限对话框
            setRightDialogVisible: false,
            // 获取所有权限列表数据
            rightsList: [],
            // 树形控件的属性绑定对象
            treeProps: {
                children: 'children',
                label: 'authName'
            },
            // 默认选中节点的数组
            defKeys: [],
            // 当前即将分配权限的角色id
            roleId: '',
            // 控制编辑角色的对话框的显示和隐藏
            editRolesdialogVisible: false,
            // 编辑用户的数据
            editRolesList: [],
            // 编辑用户的验证规则
            editRolesRules: {
                roleName: [
          { required: true, message: '角色名称不能为空', trigger: 'blur' }
                ]
            },
            // 控制添加角色的对话框的弹出和隐藏
            addRolesdialogVisible: false,
            // 添加角色的数据列表
            addRolesList: {
                roleName: '',
                roleDesc: ''
            }
        }
    },
    // 生命周期函数
    created() {
        this.getRolesList()
    },
    methods: {
        // 获取所有角色的列表
        async getRolesList() {
            const {data: res} = await this.$http.get('roles')
            if (res.meta.status !== 200) { return this.$message.error('获取角色列表失败') }
            this.rolelist = res.data
            // console.log(this.rolelist)
        },
        // 根据id删除权限
        async removeRightById(role, rightId) {
            // 弹框提示确认删除
          const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirmResult !== 'confirm') {
            return this.$message.info('取消了删除')
        }
        const {data: res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
         if (res.meta.status !== 200) { return this.$message.error('删除权限失败') }
             role.children = res.data 
        },
        // 展示分配权限的对话框
        async showsetRightDialog (role) {
            // 获取所有权限的数据
            this.roleId = role.id
            const { data: res} = await this.$http.get('rights/tree') 
            if (res.meta.status !== 200) { return this.$message.error('获取权限数据失败') }
            this.rightsList = res.data
            console.log(this.rightsList)
            // 递归获取三级节点的ID
            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true
        },
        // 通过递归的形式获取三级权限得id并保存到kefKey 数组中
        getLeafKeys(node, arr){
            if (!node.children) {
                // 根据是否有children属性，判断是否为三级权限属性
                return arr.push(node.id)
            }
            node.children.forEach(item => 
            this.getLeafKeys(item, arr))
        },
         // 监听分配权限对话框的清空
        setRightDialogClose() {
            this.defKeys = []
        },
        // 点击为角色分配权限
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids:idStr})
            if (res.meta.status !== 200) { return this.$message.error('分配权限失败') }
            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.setRightDialogVisible = false
        },
        // 根据id删除角色列表
        async removeRoleByid(id) {
            // console.log(id)
             // 询问用户是否删除数据
                const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
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
        const {data: res} = await this.$http.delete('roles/' + id)
        if (res.meta.status !== 200){
                    return this.$message.error('删除用户失败！')
                }
        this.$message.success('删除用户成功！')
        // 刷新数据列表
                this.getRolesList()    
        },
        // 展示编辑用户对话框
        showEditDialog(editRolesList) {
            // 用作用域插槽获取数据，并赋值显示到对话框中
            this.editRolesList = editRolesList
            // console.log(this.editRolesList)
            this.editRolesdialogVisible = true
        },
        // 保存编辑角色信息
        async saveRoleInfo() {
            const {data: res} = await this.$http.put('roles/' + this.editRolesList.id, {
                roleName: this.editRolesList.roleName,
                roleDesc: this.editRolesList.roleDesc
            })
            // console.log(res)
            if (res.meta.status !== 200){
                    return this.$message.error('编辑角色信息失败！')
                }
        this.$message.success('编辑角色信息成功!')
        this.getRolesList()
        this.editRolesdialogVisible = false
        },
        // 添加角色
        addRole() {
            this.addRolesdialogVisible = true
        },
        // 保存提交添加新角色
        async saveAddRole() {
            const {data: res} = await this.$http.post('roles', this.addRolesList)
            if (res.meta.status !== 201){
                    return this.$message.error('添加角色失败！')
                }
        this.$message.success('添加角色成功!')
        this.getRolesList()
            this.addRolesdialogVisible = false
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top: 1px solid #eeeeee;
}

.bdbottom {
    border-bottom: 1px solid #eeeeee;
}

.vcenter {
    display: flex;
    align-items: center;
}

</style>