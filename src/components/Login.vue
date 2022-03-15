<template>
  <div class="login_bg"
      v-loading="loading"
      element-loading-text="Logging in, please wait！"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)">
    <div class="login_box">
      <div class="title_img"></div>
      <!-- 登录表单 -->
      <el-form
        class="login_form"
        :model="ruleForm"
        :rules="rules"
        ref="form"
        label-width="0"
      >
        <el-form-item prop="adminname">
          <el-input v-model="ruleForm.adminname" placeholder="Please input name">
            <i slot="prefix" class="fa fa-user"></i>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="ruleForm.password" placeholder="Please input password">
            <i slot="prefix" class="fa fa-lock"></i>
          </el-input>
        </el-form-item>
        <el-button type="primary" @click="login">Login</el-button>
        <!-- <a href="#">注册</a> -->
        <!-- <el-button type="info">重置</el-button> -->
      </el-form>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
export default {
  data() {
    return {
      loading:false,
      ruleForm: {
        adminname: 'admin',
        password: '123456',
      },
      rules: {
        adminname: [
          { required: true, message: 'Please input name', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: 'The length is between 3 and 10 characters',
            trigger: 'blur',
          },
        ],
        password: [
          { required: true, message: 'Please input password', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: 'The length is between 6 and 15 characters',
            trigger: 'blur',
          },
        ],
      },
    }
  },
  methods: {
    login(){
      this.loading = true;
      axios.post('/api/dincan/admin/checkAdmin',this.ruleForm).then(res=>{
          this.loading = false;
          if(res.data){
            this.$message.success('Login successful！');
            //  token只应在当前网站打开期间生效，所以将token保存在sessionStorage中
            window.sessionStorage.setItem('token', this.ruleForm.adminname);
             this.$router.push('/home');
          }else{
             this.$message.error('Login failed, please login again!');
          }
      })
       
    }
  },
}
</script>
<style lang="less" scoped>
.login_bg {
  height: 100%;
  background-image: url('../assets/bj.jpg');
  background-repeat: no-repeat;
  background-size: 100%;
}
.login_bg:after {
      content: "";
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      /* 从父元素继承 background 属性的设置 */
      background: rgba(0, 0, 0, 0.4);
      filter: blur(2px);
      z-index: 2;
  }
.login_box {
  border-radius: 5px;
  position: relative;
  width: 450px;
  height: 400px;
  background-color: rgb(107, 236, 208);
  background-repeat: no-repeat;
  background-size: 100%;
  box-shadow: 5px 5px 10px 5px rgb(107, 236, 208);
  position: absolute;
  left: 50%;
  top: 50%;
  z-index: 9;
  transform: translate(-225px, -200px);
  .title_img{
    width: 200px;
    height: 200px;
    border-radius: 50%;
    background-image: url('../assets/header.jpg');
    background-size: cover;
    box-shadow: 10px 10px 20px 10px rgb(85, 186, 233);
    position: relative;
    top: 0;
    left: 50%;
    transform: translate(-100px,-100px);
  }
  // border-left: 1px solid rgb(25, 186, 235);
}
.login-title{
  position: absolute;
  top: 10%;
  width: 70%;
  left: 50%;
  transform: translate(-50%,20%);
}
.login_form {
  position: absolute;
  width: 70%;
  left: 50%;
  transform: translate(-50%,-50px);
}
.login_form > .el-input {
  width: 100%;
  padding: 20px;
}
.login_form > .el-button {
  width: 100%;
  margin-left: 50%;
  transform: translate(-50%);
}
.fa {
  /* margin:0 15px; */
  margin-left: 7px;
}
</style>