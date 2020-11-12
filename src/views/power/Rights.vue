<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <template>
        <el-table :data="rightsList" stripe border style="width: 100%">
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column prop="authName" label="商品管理"> </el-table-column>
          <el-table-column prop="path" label="路径"> </el-table-column>
          <el-table-column prop="level" label="权限等级">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.level === '0'">一级</el-tag>
              <el-tag type="success" v-if="scope.row.level === '1'">二级</el-tag>
              <el-tag type="warning" v-if="scope.row.level === '2'">三级</el-tag>
            </template>
          </el-table-column>
        </el-table>
      </template>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  methods: {
    // 渲染用户列表
    async getRightsList () {
      const res = await this.$http({
        method: 'GET',
        url: 'rights/list'
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败!')
      }
      this.rightsList = res.data
    }
  },
  created () {
    this.getRightsList()
  }
}
</script>

<style>
</style>
