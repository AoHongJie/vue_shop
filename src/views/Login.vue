<template>
  <div class="login_container" @keyup.enter="submitForm('loginForm')">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="" />
      </div>
      <!-- 登录表单区域 -->
      <el-form ref="loginForm" :rules="loginFormRules" :model="loginForm">
        <el-form-item prop="username">
          <i class="el-icon-user"></i>
          <el-input v-model="loginForm.username"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <i class="el-icon-unlock"></i>
          <el-input v-model="loginForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item class="btn">
          <el-button type="primary" @click="submitForm('loginForm')"
            >登录</el-button
          >
          <el-button class="reset" @click="reset('loginForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 16, message: '长度在 6 到 16 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    reset (formName) {
      this.$refs[formName].resetFields()
    },
    submitForm (formName) {
      this.$refs[formName].validate(async valid => {
        if (!valid) return
        const res = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) {
          return this.$message.error('用户名或密码错误!')
        }
        this.$message.success('登录成功!')
        sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
        this.username = ''
        this.password = ''
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  .el-form {
    margin: 0 15px;
    padding-top: 100px;
  }
  .el-form-item {
    position: relative;
    i {
      position: absolute;
      left: 8px;
      top: 12px;
      z-index: 99;
    }
    .el-input /deep/ input {
      padding-left: 28px;
    }
  }
  .btn {
    text-align: right;
    .reset {
      color: #fff;
      background-color: #909399;
    }
  }
}
.avatar_box {
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
</style>
