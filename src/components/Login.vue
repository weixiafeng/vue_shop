<template>
  <div class="login_container">
      <div class="login_box">
        <div class="avatar_box">
          <!--头像-->
          <img src="../assets/logo.png" alt="">
        </div>
          <!--登录表单-->
        <el-form class="login_form" :model="loginForm" :rules="loginFormRules" ref="loginFormRef">
          <!--用户名-->
          <el-form-item  prop="username">
            <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
          </el-form-item>
          <!--密码-->
          <el-form-item  prop="password">
            <el-input prefix-icon="iconfont icon-3702mima" v-model="loginForm.password" type="password" ></el-input>
          </el-form-item>
          <!--按钮-->
          <el-form-item  class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type="info" @click="resetLoginForm">重置</el-button>
            <!--
            <el-button type="info" @click="resetLoginForm('loginFormRef')">重置</el-button>
            -->
          </el-form-item>
        </el-form>
      </div>
  </div>
</template>

<script>
export default {
  data(){
    return{
      loginForm:{
        username:'admin',
        password:'123456'
      },
      loginFormRules:{
        //验证用户名是否合法
        username:[
          { required: true, message: '请输入登录用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        //验证密码是否合法
        password:[
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods:{
    resetLoginForm(){
      //console.log(this);
      this.$refs.loginFormRef.resetFields();
      //要传入参数，loginFormRef，自定义，参考文档
      //this.$refs[loginFormRef].resetFields();
    },
    login(){
      //Function(callback: Function(boolean, object))
      this.$refs.loginFormRef.validate(async valid=>{
        //console.log(valid)//true or false
        if(!valid)return;
        //返回的数据里面只有data属性是服务器返回的真实数据，解构
        const {data:res}=await this.$http.post('login',this.loginForm);
        if(res.meta.status!==200)return this.$message.error('登录失败！');
        this.$message.success('登录成功')
        // 1. 将登录成功之后的 token，保存到客户端的 sessionStorage 中
        //   1.1 项目中出了登录之外的其他API接口，必须在登录之后才能访问
        //   1.2 token 只应在当前网站打开期间生效，所以将 token 保存在 sessionStorage 中
        window.sessionStorage.setItem("token",res.data.token)
        // 2. 通过编程式导航跳转到后台主页，路由地址是 /home
        this.$router.push('/home')
        console.log(res)
        //返回结果是promise,使用await，async简化，await只能用在被async修饰的方法中
        //const result=this.$http.post('login',this.loginForm);
        //console.log(result)
      });
    }
    
  }

}
</script>

<style lang="less" scoped>
.login_container{
  background: #2b4b6b;
  height: 100%;
}
.login_box{
  width: 450px;
  height: 300px;
  //记住
  position: absolute;
  top: 50%;
  left: 50%;
  //移动元素自身长和宽的50%，结合上面，居中了
  transform: translate(-50%,-50%);
  background: white;
  border-radius: 3px;

  .avatar_box{
    height: 130px;
    width: 130px;
    border:1px solid #eee;
    border-radius: 50%;
    position: absolute;
    left: 50%;
    transform: translate(-50%,-50%);
    background-color: #eee;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;

    img{
      width:100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
  .btns{
    display: flex;
    justify-content: flex-end;//项目位于容器的结尾。
  }
  .login_form{
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
  }

}
</style>
