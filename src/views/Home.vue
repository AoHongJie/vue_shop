<template>
  <el-container>
    <el-header>
      <div class="logo">
        <img src="../assets/heima.png" alt="" />
        <h1>电商后台管理系统</h1>
      </div>
      <el-button type="info" @click="next">退出</el-button>
    </el-header>
    <!-- 页面主体区域 start -->
    <el-container>
      <!-- 侧边栏 start -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!-- 菜单栏 -->
        <el-menu
          background-color="#303133"
          text-color="#fff"
          active-text-color="#409EFF"
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="$route.path"
        >
          <!-- 一级菜单 start -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 start -->
            <el-menu-item-group>
              <el-menu-item
                :index="'/home/' + subItem.path"
                v-for="subItem in item.children"
                :key="subItem.id"
              >
                <i class="el-icon-menu"></i>
                <span slot="title">{{ subItem.authName }}</span>
              </el-menu-item>
            </el-menu-item-group>
            <!-- 二级菜单 end -->
          </el-submenu>
          <!-- 一级菜单 end -->
        </el-menu>
        <!-- 菜单栏 end -->
      </el-aside>
      <!-- 侧边栏 end -->

      <!-- 右侧内容主体 start -->
      <el-main>
        <router-view></router-view>
      </el-main>
      <!-- 右侧内容主体 end -->
    </el-container>
    <!-- 页面主体区域 end -->
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      iconsObj: {
        125: 'el-icon-platform-eleme',
        103: 'el-icon-eleme',
        101: 'el-icon-delete-solid',
        102: 'el-icon-s-tools',
        145: 'el-icon-phone'
      },
      isCollapse: false
    }
  },
  methods: {
    next () {
      // 退出功能
      this.$confirm('确定退出吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        sessionStorage.removeItem('token')
        this.$router.push('/login')
        this.$message({
          type: 'success',
          message: '退出成功!'
        })
      })
    },
    async getMenList () {
      // 发送请求获取菜单选项
      const res = await this.$http.get('menus')
      this.menulist = res.data
    },
    toggleCollapse () {
      // 切换菜单栏
      this.isCollapse = !this.isCollapse
    }
  },
  created () {
    this.getMenList()
  }
}
</script>

<style lang="less" scope>
h1 {
  font-weight: normal;
}
.el-container {
  height: 100%;
  .el-header {
    // 头部
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #373d41;
    .logo {
      display: flex;
      align-items: center;
      color: #fff;
      img {
        margin-right: 30px;
      }
    }
  }
  .el-aside {
    // 侧边栏
    background-color: #303133;
    .el-menu {
      border-right: none;
    }
  }
  .toggle-button {
    // 切换侧边栏按钮
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
}
</style>
