<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 角色权限列表 start -->
    <el-card class="box-card">
      <el-table :data="rolesList" style="width: 100%">
        <!-- 下拉菜单 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- 循环一级权限 -->
            <el-row
              :class="['bdbottom', index === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item, index) in scope.row.children"
              :key="item.id"
            >
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightsById(scope.row, item.id)"
                  >{{ item.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 循环二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[index2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, index2) in item.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      @close="removeRightsById(scope.row, item2.id)"
                      closable
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      type="warning"
                      v-for="item3 in item.children"
                      :key="item3.id"
                      @close="removeRightsById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="技术负责人"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit"
              >编辑</el-button
            >
            <el-button size="mini" type="danger" icon="el-icon-delete"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="setRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 角色权限列表 end -->

    <!-- 分配权限对话框 start -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <el-tree
        show-checkbox
        node-key="id"
        default-expand-all=""
        :data="rightsList"
        :props="treeProps"
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 分配权限对话框 end -->
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolesList: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前分配角色的Id
      roleId: ''
    }
  },
  methods: {
    // 渲染角色权限列表
    async getRolesList () {
      const res = await this.$http({
        methods: 'GET',
        url: 'roles'
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }
      this.rolesList = res.data
    },
    // 删除指定角色权限
    removeRightsById (role, rightId) {
      this.$confirm('确定删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const res = await this.$http({
            method: 'DELETE',
            url: `roles/${role.id}/rights/${rightId}`
          })
          if (res.meta.status !== 200) {
            return this.$message.error('删除角色权限失败')
          }
          role.children = res.data
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 分配权限
    async setRightDialog (role) {
      this.roleId = role.id
      const res = await this.$http({
        method: 'GET',
        url: 'rights/tree'
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败!')
      }
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 递归获取角色下所有的三级权限的id
    getLeafKeys (node, arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach((item) => this.getLeafKeys(item, arr))
    },
    // 关闭分配权限对话框清空数据
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const res = await this.$http({
        method: 'POST',
        url: `roles/${this.roleId}/rights`,
        data: {
          rids: idStr
        }
      })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败!')
      this.$message.success('分配权限成功!')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  },
  created () {
    this.getRolesList()
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
