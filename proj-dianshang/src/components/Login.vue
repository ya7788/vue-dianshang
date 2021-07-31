<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avater_box">
        <img src="../assets/logo.png" />
      </div>
      <!-- 登陆表单区域 -->
      <!-- model数据绑定 rule规则绑定 ref拿到表单实例对象-->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRule" label-width="0" class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <!-- 双向绑定v-model -->
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input type="password" v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <!-- 登陆按钮 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登陆</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 登陆表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单验证规则对象
      loginFormRule: {
        // 验证用户名密码是否合法
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      // async异步
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return false
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登陆失败')
        this.$message.success('登录成功')
        // 保存到客户端
        window.sessionStorage.setItem('token', res.data.token)
        // 跳转到后台主页，路由地址是 /home
        this.$router.push('/home')
      })
    }
  }
}
</script>

// scope只在当前组件生效
<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.avater_box {
  width: 130px;
  height: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  box-shadow: 0 0 10px #eee;
  padding: 10px;
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

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
