<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
<!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与天津区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
            <div>
            <el-input placeholder="请输入内容" class="input-with-select" clearable @clear='getUserList' v-model="queryInfo.query">
              <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
            </el-input></div>
            </el-col>
             <el-col :span="4">
            <el-button type="primary"  @click="dialogVisible = true">添加用户</el-button>     
             </el-col>
      </el-row>

<!-- 用户列表去 -->
      <el-table :data='userlist' border stripe>
                <el-table-column label="序号" type="index"></el-table-column>
          <el-table-column label="姓名" prop="username"></el-table-column>
             <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                   <el-table-column label="角色" prop="role_name"></el-table-column>
                    <el-table-column label="状态">
                      <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
                        </el-switch>
                      </template>
                     </el-table-column>
                     <el-table-column label="操作">
                          <template slot-scope="scope">
                                    <!-- 编辑用户 -->
                                    <el-button type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)" circle size="mini"></el-button>
                                    <!-- 删除按钮 -->
                                    <el-button type="danger" @click="removeUser(scope.row.id)" icon="el-icon-delete" circle size="mini"></el-button>
                                    <!-- 分配角色 -->
                                    <el-tooltip class="item" effect="dark"  content="分配角色" placement="top" :enterable="false">
                                     <el-button type="warning" icon="el-icon-setting" @click="setRole(scope.row)" circle size="mini"></el-button>
                                    </el-tooltip>
                          </template>

                     </el-table-column>
          </el-table>
    <el-pagination class="pagination"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>

    </el-card>
<!-- 添加用户弹框 -->
    <el-dialog
     title="添加用户"
     :visible.sync="dialogVisible"
     @close="addDialogClosed"
     width="50%">
  <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px" >
  <el-form-item label="用户名" prop="username">
    <el-input v-model="addForm.username"></el-input>
  </el-form-item>
  <el-form-item label="密码" prop="password">
    <el-input type="password" v-model="addForm.password"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="addForm.email"></el-input>
  </el-form-item>
  <el-form-item label="手机" prop="mobile">
    <el-input v-model="addForm.mobile"></el-input>
  </el-form-item>
  </el-form>
   <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改用户的对话框 -->
<el-dialog
  title="修改用户"
  :visible.sync="editDialogVisible"
  width="50%" @close='editDialogClosed'>

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

<!-- 分配用户角色对话框 -->
<el-dialog
  title="分配角色"
  :visible.sync="setRoleDialogVisible"
  width="50%"
  @close="setRoleDialogClosed">
  <div>
   <p>当前用户：{{userInfo.username}}</p>
   <p>当前角色：{{userInfo.role_name}}</p>
   <p>分配新角色：
      <el-select v-model="selectedRoleId" placeholder="请选择">
     <el-option
       v-for="item in rolelist"
      :key="item.id"
      :label="item.roleName"
      :value="item.id">
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
  data() {
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的邮箱'))
    }

    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      // 返回一个错误提示
      cb(new Error('请输入合法的手机号码'))
    }

    return {
      editForm: {
        username: '',
        email: '',
        mobile: ' '
      },
      dialogVisible: false,
      editDialogVisible: false,
      setRoleDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }]

      },
      addFormRules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名长度为3-10个字符', trigger: 'blur' }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度为6-15个字符', trigger: 'blur' }
        ],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }],
        mobile: [{ required: true, message: '请输入手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }]

      },
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      // 需要被分配角色的用户信息对象
      userInfo: {},
      // 所有角色的数据列表
      rolelist: [],
      // 已选中的角色Id值
      selectedRoleId: [],
      total: 0
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    // 监听分配角色对话框的关闭事件
    setRoleDialogClosed() {
      this.selectedRoleId = ''
      this.userInfo = {}
    },
    async saveRoleInfo() {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择要分配的角色')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色失败')
      }
      this.$message.success('更新角色信息成功')
      this.getUserList()
      this.setRoleDialogVisible = false
    },
    // 用户分配角色对话框
    async setRole(userInfo) {
      this.userInfo = userInfo
      // 在展示对话框之前，获取所有角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.err('获取角色列表失败')
      this.rolelist = res.data
      console.log(this.rolelist)
      this.setRoleDialogVisible = true
    },
    async removeUser(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('更新用户信息成功')
      })
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/'+id)
      if (res.meta.status !== 200) {
        return this.message.error('查询用户信息失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.dialogVisible = false
        this.getUserList()
      })
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败')
      }
      this.userlist = res.data.users
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChange(userInfo) {
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (res.meta.status !== 200) {
        return this.$message.error('修改状态失败')
        userInfo.mg_state = !userInfo.mg_state
      }
      return this.$message.success('修改状态成功')
    }
  }
}
</script>

<style lang="less" scoped>
.text {
  font-size: 14px;
}

.item {
  padding: 18px 0;
}

.box-card {
  text-align: center;
  width: 480px;
}
.pagination{
text-align: center;

}
.el-tag {
  margin: 7px;
}
</style>>
