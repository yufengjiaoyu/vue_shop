<template>
<el-container>
   <el-header >
     <div>
      <img src='../assets/gif.gif' alt="">
     <span>
       后台管理系统
     </span>

     </div>

     <el-button type="info" @click="logout">退出</el-button>
     </el-header>

   <el-container>

      <el-aside width="200px">

         <el-menu
      background-color="#afeeee"
      text-color="#2F4F4F"
      active-text-color="#ffd04b">
      <el-submenu index="1">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>导航一</span>
        </template>
          <el-menu-item index="1-1">
          <template slot="title">
          <i class="el-icon-location"></i>
          <span>导航一</span>
        </template>
        </el-menu-item>

      </el-submenu>
    </el-menu>
      </el-aside>

      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>
.el-container {
  width: 100%;
  height: 100%;
}
.el-header {
  background-color: #4682b4;
  display: flex;
  justify-content: space-between ;
  align-items: center;
  padding-left: 5px;
  color: aliceblue;
  font-size: 20px;
  div {
    display: flex;
    align-items: center;

 span{
    margin-left: 20px
  };
  img {
    width: 40px;
    height: 40px;
   }
}

}
.el-aside {
  background-color: #afeeee;
}

.el-main {
  background-color: #e1ffff;
}
</style>
