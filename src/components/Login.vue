<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 登录默认头像 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!-- 登录表单区域 -->
      <!-- ref(定位表单), :model(绑定表单), :rules(表单验证规则) -->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" class="login_form" label-width="0px">
        <!-- 用户名 -->
        <el-form-item prop="username" label="">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password" label="">
          <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item class="btns" label="">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form> 
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // 登录表单的数据绑定对象
        loginForm: {
          username: 'admin',
          password: '123456'
        },
        // 表单的验证规则对象
        loginFormRules: {
          // 表单的用户名验证是否合法
          username: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 2, max: 10, message: '长度在2到5个字符', trigger: 'blur' }
          ],
          // 表单的密码验证是否合法
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在6到15个字符', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      // 点击重置按钮，重置登录表单
      resetLoginForm() {
        // console.log(this);
        this.$refs.loginFormRef.resetFields();
      },
      login() {
        // async 将函数修饰为异步函数
        this.$refs.loginFormRef.validate(async valid => {
          // console.log(valid);
          if(!valid) return;
          // await 只能用于被async修饰的方法中
          // $http 是 加载到原型上的 axios
          const { data: res } = await this.$http.post("login", this.loginForm)
          // $message 是element ui的弹框提示
          if(res.meta.status !== 200) return this.$message.error('登录失败！')
          this.$message.success('登录成功!')
          // 1. 将登录成功之后的 token，保存到客户端的 sessionStorage 中
          //   1.1 项目中出了登录之外的其他API接口，必须在登录之后才能访问
          //   1.2 token 只应在当前网站打开期间生效，所以将 token 保存在 sessionStorage 中
          // console.log(res);
          window.sessionStorage.setItem('token', res.data.token)
          // 2. 通过编程式导航跳转到后台主页，路由地址是 /home
          this.$router.push('/home')
          
        })
      }
    }
  }
</script>

<style lang="less" scoped>
.login_container{
  height: 100%;
  // background-color: #2b4b6b;
  background-image: linear-gradient(to top, #fff1eb 0%, #ace0f9 100%);
}
// 登录容器样式
.login_box{
  width:450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top:50%;
  transform: translate(-50%, -50%);

  .avatar_box{
    width: 130px;
    height: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img{
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
  .login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 30px;
    box-sizing: border-box;
  }
  .btns {
    display: flex;
    justify-content: flex-end;
  }
}
</style>