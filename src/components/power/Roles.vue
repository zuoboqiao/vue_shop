<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
                <!-- 通过 for 循环嵌套渲染一级权限 -->
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop':'', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag  closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过 for 循环嵌套渲染二级权限 -->
                <el-row :class="[i2 === 0 ? '':'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
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

            <!-- <pre>
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
            <!-- 删除按钮 -->
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <!-- 分配权限按钮 -->
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" 
      show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色的对话框 -->
    <el-dialog title="删除用户" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addRoleForm" :rules="addRoleFormRules" ref="addRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editRoleDialogVisible" width="50%" @close="editRoleDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色的列表数据
      roleList: [],
      // 控制分配权限对话框的显示和隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点ID值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',

      // 控制添加角色对话框的显示与隐藏
      addRoleDialogVisible: false,
      // 添加角色的表单数据
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色的表单数据验证规则对象
      addRoleFormRules: {
        roleName:[
          { required: true, message: '请输入角色名称', trigger: 'blur'},
          { min: 2, max: 8, message: '长度在2-8个字符', trigger: 'blur' }
        ],
        roleDesc: ''
      },

      // 控制编辑角色对话框的显示与隐藏
      editRoleDialogVisible: false,
      // 编辑角色的表单数据
      editRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑角色的表单数据验证规则对象
      editRoleFormRules: {
        roleName:[
          { required: true, message: '请输入角色名称', trigger: 'blur'},
          { min: 2, max: 8, message: '长度在2-8个字符', trigger: 'blur' }
        ],
        roleDesc: ''
      }
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.roleList = res.data;
      // console.log(this.roleList);
    },

    // 根据 id 删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否删除权限
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

        if(confirmResult !== 'confirm'){
          return this.$message.info('取消了删除')
        }

        // console.log('确定了删除');
        const { data:res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !==200) {
          return this.$message.error('删除权限失败')
        }

        // this.getRolesList()
        role.children = res.data
    },

    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有的权限数据
      const { data:res } = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }

      // 把获取到的权限数据保存到 data 中
      this.rightsList = res.data

      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)
      
      this.setRightDialogVisible = true
      // console.log(this.rightsList);
    },
    // 通过递归的形式，获取角色下所有的三级权限的id，并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if(!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item =>
      this.getLeafKeys(item, arr)
      )
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.
        getCheckedKeys(),
        ...this.$refs.treeRef.
        getHalfCheckedKeys()
      ]
      // console.log(keys);
      const idStr = keys.join(',')

      const { data:res } = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
      if(res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },

    // 监听添加角色对话框的关闭事件
      addRoleDialogClosed() {
        this.$refs.addRoleFormRef.resetFields()
      },
    // 展示添加角色的对话框
    addRole() {
      this.$refs.addRoleFormRef.validate(async valid => {
        if(!valid) return
        // 可以发起添加角色的网络请求
        const { data:res } = await this.$http.post('roles', this.addRoleForm)

        if(res.meta.status !== 201) {
          this.$message.error('添加角色失败')
        }

        this.$message.success('添加角色成功')
        // 隐藏添加角色对话框
        this.addRoleDialogVisible = false
        // 重新获取角色列表
        this.getRolesList()
      })
    },

    // 根据id删除对应的角色信息
    async removeRoleById(id) {
      // 弹框确认是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 确认删除 返回 confirm
      // 取消删除 返回 cancel
      // console.log(confirmResult);
      if(confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      const { data:res } = await this.$http.delete('roles/' + id)

      if(res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }

      this.$message.success('删除角色成功!')

      // 重新渲染角色列表
      this.getRolesList()
    },

    // 展示编辑按钮的对话框
    async showEditRoleDialog(id) {
      const { data:res } = await this.$http.get('roles/' + id)
      // console.log(res);

      if(res.meta.status !== 200) {
        return this.$message.error('查询角色失败')
      }

      this.editRoleForm = res.data
      // console.log(this.editRoleForm);

      this.editRoleDialogVisible = true
    },
    // 监听编辑角色对话框的关闭事件
      editRoleDialogClosed() {
        this.$refs.editRoleFormRef.resetFields()
      },
    // 修改角色信息并提交
    editRoleInfo() {
      this.$refs.editRoleFormRef.validate(async valid => {
        if(!valid) return
        // 发起修改角色信息的数据请求
        const { data:res } = await this.$http.put(
          'roles/' + this.editRoleForm.roleId,
            {
              roleName: this.editRoleForm.roleName,
              roleDesc: this.editRoleForm.roleDesc
            }
          )

          if(res.meta.status !== 200) {
            return this.$message.error('更新角色失败')
          }

          // 关闭对话框
          this.editRoleDialogVisible = false
          // 重新获取角色列表
          this.getRolesList()
          // 提示更新角色成功
          this.$message.success('更新角色信息成功')
      })
    }

  }
};
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.bdtop{
  border-top: 1px solid #eee;
}

.bdbottom{
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>