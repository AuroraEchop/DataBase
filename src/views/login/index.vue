<script setup lang="ts">
import { loginAPI } from '@/api/employee'
import { useRouter } from 'vue-router'
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { useUserInfoStore } from '@/store'

const userInfoStore = useUserInfoStore()

const form = ref({
  userName: '',
  password: ''
})
// 表单校验的ref
const loginRef = ref()

const router = useRouter()

const loginFn = async () => {
  // 调用登录接口
  const { data: res } = await loginAPI(form.value)
  console.log(res)
  // 登录失败，提示用户，这个提示已经在响应拦截器中统一处理了，这里直接return就行
  if (res.code !== 0) {
    return false
  }
  // 登录成功，提示用户
  ElMessage.success('登录成功')
  // 把后端返回的当前登录用户信息(包括token)存储到Pinia里
  userInfoStore.userInfo = res.data
  console.log(userInfoStore.userInfo)
  // 跳转到首页
  await router.push('/')
}
</script>

<template>
  <div class="background">
    <el-form label-width="0px" class="login-box" :model="form" ref="loginRef">
      <div class="title-box">登 录</div>
      <el-form-item prop="userName">
        <el-input v-model="form.userName" placeholder="请输入账号"></el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input type="password" v-model="form.password" placeholder="请输入密码"></el-input>
      </el-form-item>
      <el-form-item class="my-el-form-item">
        <el-button type="primary" class="btn-login" @click="loginFn">登录</el-button>
        <el-link type="info" @click="$router.push('/reg')">去注册</el-link>
      </el-form-item>
    </el-form>
  </div>
</template>

<style lang="less" scoped>
body {
  margin: 0;
  padding: 0;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.background {
  width: 100%;
  height: 100vh;
  background-size: cover;
  background-image: url('../../assets/image/ship.jpeg');
  overflow: hidden; // 防止页面滚动条闪动
}

.background::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  /* 黑色半透明 */
  z-index: 1;
  /* 确保伪元素在背景图之上 */
}

@keyframes animate {
  0% {
    transform: translateY(100vh) scale(0);
  }

  100% {
    transform: translateY(-10vh) scale(1);
  }
}

.login-box {
  z-index: 10;
  width: 400px;
  height: 340px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  padding: 0 30px;
  box-sizing: border-box;
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  box-shadow: #dddddd 0 0 100px;

  .title-box {
    height: 100px;
    line-height: 100px;
    font-size: 24px;
    font-weight: bold;
    text-align: center;
    color: #00aaff;
  }

  .el-form-item {
    margin-bottom: 20px;
  }

  .btn-login {
    width: 100%;
  }

  .el-link {
    margin-top: 25px;
  }
}
</style>
