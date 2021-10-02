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
                  <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表区 -->
        <el-table :data="rolelist" border stripe>
            <!-- 展开列 -->
                 <el-table-column type="expand" label="展开"></el-table-column>
            <!-- 索引列 -->
          <el-table-column type="index" label="序号"></el-table-column>
          <el-table-column  label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
              <template slot-scope="scope">
                  <el-button size="mini" type="primary" icon="el-icon-search">编辑</el-button>
                  <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                  <el-button size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
              </template>
          </el-table-column>
        </el-table>
    </el-card>
<!-- 添加角色弹框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%">
      <span>
        <el-form :model="roleForm" :rules="roleRules" ref="addRoleRef" label-width="100px" >
        <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="roleForm.roleName" ></el-input>
         </el-form-item>                   
         <el-form-item label="角色描述" prop="roleDesc">
         <el-input v-model="roleForm.roleDesc"></el-input>
        </el-form-item>
        </el-form>
      </span>
     <span slot="footer" >
    <el-button @click= "addRoleDialogVisible = false">取 消</el-button>
    <el-button type="primary"  @click="addRoleInfo">确 定</el-button>
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
      roleForm:{
        roleName: '',
        roleDesc: ''
    },
      roleRules: {roleDesc: [{ required: true, message: '请输入角色描述',  trigger:'blur' }],
                 roleName: [{ required: true, message: '请输入角色名称',   trigger:'blur'}]
                 
                 },
      addRoleDialogVisible:false,
      rolelist: []
    }
  },
  methods: {
    addRoleInfo(){
    this.$refs.addRoleRef.validate(async valid => {
     if(!valid) return 
     const {data: res} = await this.$http.post('roles',this.roleForm)
     if ( res!==201 ) {
       this.$message.error('添加角色失败')
     }
      this.$message.success('添加角色成功')
     this.addRoleDialogVisible=false
     this.getRolesList()

    })
    },
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    }

  }

}
</script>

<style>

</style>
