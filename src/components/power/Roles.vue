<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色列表区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 角色列表区 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand" label="展开">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', 'vcenter', i1 === 0 ? 'bdtop' : '']"
              v-for="(item, i1) in scope.row.children"
              :key="item.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  type="primary"
                  @close="removeRightById(scope.row, item.id)"
                  closable
                >
                  {{ item.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item.children"
                  :key="item2.id"
                >
                  <!-- 二级 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      @close="removeRightById(scope.row, item2.id)"
                      closable
                    >
                      {{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级 -->
                  <el-col :span="18">
                    <el-tag
                      :class="i3 === 0 ? '' : 'bdtop'"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      @close="removeRightById(scope.row, item3.id)"
                      closable
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-search"
              @click="showEditRoleDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="showDeleteRoleDialog(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色弹框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%"
      @close="addRoleClosed"
    >
      <span>
        <el-form
          :model="roleForm"
          :rules="roleRules"
          ref="addRoleRef"
          label-width="100px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="roleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="roleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色弹框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="editRoleDialogVisible"
      width="50%"
      @close="editRoleClosed"
    >
      <span>
        <el-form
          :model="editRoleForm"
          :rules="roleRules"
          ref="editRoleRef"
          label-width="100px"
        >
          <el-form-item label="序号">
            <el-input v-model="editRoleForm.roleId" disabled></el-input>
          </el-form-item>
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo()">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 删除角色弹框 -->
    <el-dialog
      title="删除角色"
      :visible.sync="deleteRoleDialogVisible"
      width="50%"
    >
      <span :model="deleteRoleForm">
        确定要删除角色{{ deleteRoleForm.roleName }}吗？
      </span>
      <span slot="footer">
        <el-button @click="deleteRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="deleteRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%" @close='setRightDialogClosed'>
      <!-- 树形控件 -->
      <el-tree :data="rightlist"
       :props="rightProps" show-checkbox
       default-expand-all node-key="id"
       :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
         @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created() {
    this.getRolesList()
  },
  data() {
    return {
      // 所有角色列表数据
      roleForm: {
        roleName: '',
        roleDesc: ''
      },
      editRoleForm: {
        roleName: '',
        roleDesc: '',
        roleId: ''
      },
      deleteRoleForm: {
        roleName: '',
        roleDesc: '',
        roleId: ''
      },
      roleRules: {
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' },
        ],
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
        ]
      },
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      deleteRoleDialogVisible: false,
      setRightDialogVisible: false,
      rolelist: [],
      rightlist: [],
      // 默认选中的节点id值
      defKeys: [],
      //  树形控件属性绑定对象
      rightProps: {
        children: 'children',
        label: 'authName',
        // 即将分配权限的角色Id
        roleId: ''
      }
    }
  },
  methods: {
    // 点击分配角色权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.err('分配权限失败')
      this.$message.success('分配角色权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 监听分配权限对胡框关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 通过递归的形式，获取角色下所有三级权限id，并保存到defKeys 数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
        this.defKeys = arr
      }

      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },

    // 展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.err('获取权限列表失败')
      this.rightlist = res.data

      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    async removeRightById(role, rightId) {
      // 弹框提示用户是否删除三级标签
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件，是否继续',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) return this.$message.err('删除权限失败')
      role.children = res.data
    },
    addRoleClosed() {
      this.$refs.addRoleRef.resetFields()
    },
    editRoleClosed() {
      this.$refs.editRoleRef.resetFields()
    },
    addRoleInfo() {
      this.$refs.addRoleRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.roleForm)
        if (res !== 201) {
          this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.addRoleDialogVisible = false
        this.getRolesList()
      })
    },
    async showEditRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.editRoleDialogVisible = true
      this.editRoleForm = res.data
    },
    async showDeleteRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.deleteRoleDialogVisible = true
      this.deleteRoleForm = res.data
    },
    editRoleInfo() {
      this.$refs.editRoleRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editRoleForm.roleId,
          {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色信息失败')
        }
        this.$message.success('修改角色信息成功')
        this.editRoleDialogVisible = false
        this.getRolesList()
      })
    },
    async deleteRoleInfo() {
      const { data: res } = await this.$http.delete(
        'roles/' + this.deleteRoleForm.roleId
      )
      if (res.meta.status !== 200) return this.$message.error('删除角色失败')
      this.$message.success('删除角色成功')
      this.deleteRoleDialogVisible = false
      this.getRolesList()
    },
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
    }
  }
}
</script>

<style>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid whitesmoke;
}
.bdbottom {
  border-bottom: 1px solid whitesmoke;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
