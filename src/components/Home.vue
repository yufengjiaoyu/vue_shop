<template>
  <el-container>
    <el-header>
      <div>
        <img src="../assets/gif.gif" alt="" />
        <span> 后台管理系统 </span>
      </div>

      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container>
      <el-aside :width ='isCollapse ? "64px" : "200px"'>
<div class="toggle-botton" @click="toggleCollapse"><i class='el-icon-d-arrow-left'></i></div>

        <el-menu
          background-color="#afeeee"
          text-color="#2F4F4F"
          active-text-color="#8B658B"
          :unique-opened="true"
          :collapse='isCollapse'
          :collapse-transition='false'
        >
          <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="sub_item.id+''" v-for="sub_item in item.children" :key="sub_item.id">
              <template slot="title">
                <i class='el-icon-menu'></i>
                <span >{{sub_item.authName}}</span>
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
  data(){
    return{
      menulist:[],
      isCollapse:false,
      iconsObj:{
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao' 
      }
    }
  },

created() {
    this.getMenuList()
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    toggleCollapse(){
      this.isCollapse=!this.isCollapse

    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menulist = res.data
        console.log(this.menulist)
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
  justify-content: space-between;
  align-items: center;
  padding-left: 5px;
  color: aliceblue;
  font-size: 20px;
  div {
    display: flex;
    align-items: center;

    span {
      margin-left: 20px;
    }
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
.iconfont {
  margin-right:10px;

}
.toggle-botton{
cursor: pointer;
 text-align:center;
 background-color: #40E0D0;
}

</style>
