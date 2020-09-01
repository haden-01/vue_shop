<template>
<div class="login_container">
  <div class="login_box">
    <!-- 头像区域 -->
    <div class="avatar_box">
      <img src="../assets/logo.png" alt="">
      <!-- 登陆表单区 -->
      <el-form ref="loginFormRef" label-width="0px" class="login_form" :model="loginForm" :rules="loginFormRules">
        <!-- 用户名 -->
  <el-form-item prop="username">
    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user" placeholder="admin"></el-input>
  </el-form-item>
  <!-- 密码区 -->
  <el-form-item prop="password">
    <el-input v-model="loginForm.password" type="password" placeholder="password" prefix-icon="iconfont icon-3702mima"></el-input>
  </el-form-item>
  <!-- 按钮区 -->
  <el-form-item class="Btns">
   <el-button type="primary" @click="login">登陆</el-button>
   <el-button type="info" @click="resetLoginForm">重置</el-button>
  </el-form-item>
  </el-form>
    </div>
  </div>
</div>
</template>

<script>
export default {
  data () {
    return {
      // 这是登陆表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 这是表单验证规则对象
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入登陆名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入登陆密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 点击重置按钮，重置登陆表单
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登陆失败！')
        this.$message.success('登陆成功！')
        console.log(res.data)
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }

  }
}

</script>

<style lang="less" scoped>
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: #ffffff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform:translate(-50%, -50%) ;
}

.avatar_box {
  height: 130px;
  width: 130px;
  border: 1px solid #eeeeee;
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #dddddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%,-50%);
  background-color: #FFF;
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eeeeee ;
  }
}
.Btns {
  text-align: right;
}

.login_form{
  position: absolute;
  top: 180px;
  width: 400px;
  left: -125px;
  box-sizing: border-box;
}
</style>
