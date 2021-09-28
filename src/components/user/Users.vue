<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
            <div>
            <el-input placeholder="请输入内容" class="input-with-select">
              <el-button slot="append" icon="el-icon-search"></el-button>
            </el-input></div>
            </el-col>
             <el-col :span="4">
            <el-button type="primary">添加用户</el-button>     
             </el-col>
      </el-row>

      <el-table :data='userlist' border stripe>
          <el-table-column label="姓名" prop="username"></el-table-column>
             <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                   <el-table-column label="角色" prop="role_name"></el-table-column>
                    <el-table-column label="状态" prop="mg_state"></el-table-column>
                     <el-table-column label="操作"></el-table-column>
          </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
    data(){
      return{
          queryInfo:{
              query: '',
              pagenum: 1,
              pagesize: 2
          },
          userlist:[],
          total:0
        
      }
    },
    created(){
        this.getUserList()

    },
    methods:{
      async getUserList(){
     const { data: res }= await this.$http.get('users',{params: this.queryInfo})
     if (res.meta.status !== 200) {return this.$message.error('获取用户列表失败')
     }
       this.userlist = res.data.users
       this.total = res.data.total
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
  width: 480px;
}
</style>>

