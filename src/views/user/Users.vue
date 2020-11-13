<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索和添加区域 -->
    <el-card class="box-card">
      <el-row :gutter="20">
        <!-- 搜索框 -->
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            clearable
            @clear="getUserList"
            v-model="queryInfo.query"
            class="input-with-select"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="search"
            ></el-button>
          </el-input>
        </el-col>
        <!-- 添加按钮 -->
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
    </el-card>
    <!-- 用户列表区域 start -->
    <el-table :data="userList" style="width: 100%" border stripe>
      <el-table-column type="index" label="#"> </el-table-column>
      <el-table-column prop="username" label="姓名"> </el-table-column>
      <el-table-column prop="email" label="邮箱"> </el-table-column>
      <el-table-column prop="mobile" label="电话"> </el-table-column>
      <el-table-column prop="mg_state" label="状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            :disabled="scope.row.username === 'admin'"
            @change="userStatusChange(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180px">
        <template slot-scope="scope">
          <!-- 修改 -->
          <el-button
            @click="showEditDialog(scope.row)"
            type="primary"
            size="mini"
            ><i class="el-icon-edit"></i
          ></el-button>
          <!-- 删除 -->
          <el-button @click="del(scope.row.id)" type="danger" size="mini"
            ><i class="el-icon-delete"></i
          ></el-button>
          <!-- 分配角色 -->
          <el-tooltip
            effect="dark"
            content="分配角色"
            placement="top"
            :enterable="false"
          >
            <el-button type="warning" size="mini" @click="setRole(scope.row)"
              ><i class="el-icon-setting"></i
            ></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- 用户列表区域 end -->

    <!-- 分页器 start -->
    <div class="block">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 3, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </div>
    <!-- 分页器 end -->

    <!-- 添加用户对话框 start -->
    <el-dialog
      title="添加用户"
      :visible.sync="dialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
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
    <!-- 添加用户对话框 end -->

    <!-- 修改用户对话框 start -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editDialogVisible"
      width="50%"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名:">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱:" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号:" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 end -->

    <!-- 分配角色对话框 start -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
      @colse="setRoleDialogClosed"
    >
      <div>
        <p>当前的用户: {{ userInfo.username }}</p>
        <p>当前的角色: {{ userInfo.role_name }}</p>
        <p>
          分配新角色:
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 分配角色对话框 end -->
  </div>
</template>

<script>
export default {
  data () {
    const checkEmail = (rule, value, callback) => {
      // 自定义邮箱验证规则
      const regEmail = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      if (!regEmail.test(value)) return callback(new Error('请输入合法的邮箱'))
      return callback()
    }
    const checkMobile = (rule, value, callback) => {
      // 自定义手机验证规则
      const regMoubile = /^(?:(?:\+|00)86)?1(?:(?:3[\d])|(?:4[5-7|9])|(?:5[0-3|5-9])|(?:6[5-7])|(?:7[0-8])|(?:8[\d])|(?:9[1|8|9]))\d{8}$/
      if (!regMoubile.test(value)) {
        return callback(new Error('请输入合法的手机号'))
      }
      return callback()
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1, // 当前显示页码
        pagesize: 2 // 每页显示条数
      },
      userList: [],
      total: 0, // 总条数
      dialogVisible: false, // 控制添加用户对话框的显示和隐藏
      editDialogVisible: false, // 控制修改用户信息的对话框显示和隐藏
      setRoleDialogVisible: false, // 控制分配角色对话框的显示和隐藏
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户表单验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名的长度在3~10之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '密码长度在6~15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      // 编辑用户信息表单验证规则
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 需要被分配角色的用户信息
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      selectedRoleId: ''
    }
  },
  methods: {
    async getUserList () {
      // 获取用户列表
      const res = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败!')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      // 每页显示条数变化事件
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newPage) {
      // 页码切换事件
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStatusChange (userInfo) {
      // 切换用户状态
      const res = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败!')
      }
      this.$message.success('更新用户状态成功!')
    },
    search () {
      // 搜索
      this.getUserList()
    },
    addDialogClosed () {
      // 监听添加用户表单对话框的关闭事件
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      // 添加用户
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const res = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        this.$message.success('添加用户成功!')
        this.getUserList()
      })
      this.dialogVisible = false
    },
    showEditDialog ({ id, username, mobile, email }) {
      // 显示编辑用户对话框
      this.editDialogVisible = !this.editDialogVisible
      this.editForm.username = username
      this.editForm.mobile = mobile
      this.editForm.email = email
      this.editForm.id = id
    },
    editUserInfo () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const res = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户信息失败!')
        }
        this.$message.success('修改用户信息成功!')
        this.getUserList()
        this.editDialogVisible = !this.editDialogVisible
      })
    },
    del (id) {
      this.$confirm('确定删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const res = await this.$http({
            method: 'DELETE',
            url: `users/${id}`
          })
          if (res.meta.status !== 200) return this.$message.error('删除失败!')
          this.getUserList()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 分配角色
    async setRole (userInfo) {
      this.userInfo = userInfo
      const res = await this.$http({
        url: 'roles'
      })
      if (res.meta.status !== 200) { return this.$message.error('获取用户列表失败!') }
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    // 点击按钮分配角色
    async saveRoleInfo () {
      if (!this.selectedRoleId) return this.$message.error('请选择要分配的角色!')
      const res = await this.$http({
        method: 'PUT',
        url: `users/${this.userInfo.id}/role`,
        data: {
          rid: this.selectedRoleId
        }
      })
      if (res.meta.status !== 200) return this.$message.error('更新角色失败!')
      this.$message.success('更新角色成功!')
      this.getUserList()
      this.setRoleDialogVisible = false
    },
    // 分配角色对话框关闭事件
    setRoleDialogClosed () {
      this.selectedRoleId = ''
      this.userInfo = {}
    }
  },
  created () {
    this.getUserList()
  }
}
</script>

<style>
</style>
