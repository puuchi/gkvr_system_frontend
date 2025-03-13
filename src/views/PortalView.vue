<template>
  <div class="background">
    <div class="center-container">
      <div class="animated-text">
        <h1>高考志愿填报推荐系统</h1>
        <h3>GaoKao Volunteer Application and Recommendation System</h3>
      </div>
      <div class="btn-container animated-text">
        <button class="btn-hover btn-color" @click="showRegisterDialog = true">
          开始注册
        </button>
        <button class="btn-hover btn-color" @click="showLoginDialog = true">
          立即登录
        </button>
      </div>
    </div>
    <el-dialog
        v-model="showRegisterDialog"
        width="400px"
        align-center
        center
        :show-close="false"
        class="custom-dialog"
    >
      <template #header>
        <div class="dialog-header">
          <h2>用户注册</h2>
          <p>欢迎加入我们的平台</p>
        </div>
      </template>
      <el-form :model="registerForm" class="login-form">
        <el-form-item>
          <el-input
              v-model="registerForm.username"
              autocomplete="off"
              placeholder="请输入用户名"
          >
            <template #prefix>
              <el-icon class="input-icon"><Avatar /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-input
              type="password"
              v-model="registerForm.password"
              autocomplete="off"
              show-password
              placeholder="请输入密码"
          >
            <template #prefix>
              <el-icon class="input-icon"><Lock /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-input
              type="password"
              v-model="registerForm.checkPassword"
              autocomplete="off"
              show-password
              placeholder="请确认密码"
          >
            <template #prefix>
              <el-icon class="input-icon"><Lock /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <div class="dialog-footer">
          <el-button class="dialog-btn cancel-btn" @click="showRegisterDialog = false">取消</el-button>
          <el-button class="dialog-btn confirm-btn" type="primary" @click="register">注册</el-button>
        </div>
      </el-form>
    </el-dialog>
    <el-dialog
        v-model="showLoginDialog"
        width="400px"
        align-center
        center
        :show-close="false"
        class="custom-dialog"
    >
      <template #header>
        <div class="dialog-header">
          <h2>用户登录</h2>
          <p>欢迎回来</p>
        </div>
      </template>
      <el-form :model="loginForm" class="login-form">
        <el-form-item>
          <el-input
              v-model="loginForm.username"
              autocomplete="off"
              placeholder="请输入用户名"
          >
            <template #prefix>
              <el-icon class="input-icon"><Avatar /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-input
              type="password"
              v-model="loginForm.password"
              autocomplete="off"
              show-password
              placeholder="请输入密码"
          >
            <template #prefix>
              <el-icon class="input-icon"><Lock /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <div class="dialog-footer">
          <el-button class="dialog-btn cancel-btn" @click="showLoginDialog = false">取消</el-button>
          <el-button class="dialog-btn confirm-btn" type="primary" @click="login">登录</el-button>
        </div>
      </el-form>
    </el-dialog>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";
import { useRouter } from "vue-router";
import { ElLoading, ElMessage } from "element-plus";
import request from "@/utils/request.js";

const router = useRouter();
const showLoginDialog = ref(false);
const showRegisterDialog = ref(false);

const registerForm = reactive({
  username: "",
  password: "",
  checkPassword: "",
});
const loginForm = reactive({
  username: "",
  password: "",
});

const login = async () => {
  if (!loginForm.username || !loginForm.password) {
    ElMessage.error("请填写完整的登录信息！");
    return;
  }
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.post("/user/login", loginForm);
    if (response.code == 200) {
      ElMessage.success("登录成功！");
      showLoginDialog.value = false;
      localStorage.setItem("username", loginForm.username);
      loginForm.username = "";
      loginForm.password = "";
      router.push("/home/school");
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};

const register = async () => {
  if (
      !registerForm.username ||
      !registerForm.password ||
      !registerForm.checkPassword
  ) {
    ElMessage.error("请填写完整的注册信息");
    return;
  }

  if (registerForm.password !== registerForm.checkPassword) {
    ElMessage.error("两次输入的密码不一致");
    return;
  }
  const loadingInstance = ElLoading.service({
    fullscreen: true,
    text: "正在加载中...",
  });
  try {
    const response = await request.post("/user/register", registerForm);
    if (response.code == 200) {
      ElMessage.success("注册成功！");
      showRegisterDialog.value = false;
      registerForm.username = "";
      registerForm.password = "";
      registerForm.checkPassword = "";
    } else {
      ElMessage.error(response.message);
    }
  } catch (error) {
    ElMessage.error(error);
  }
  loadingInstance.close();
};
</script>

<style scoped>
.background {
  background: linear-gradient(135deg, #ffffff 0%, #e6f7ff 50%, #1890ff 100%);
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.center-container {
  display: flex;
  flex-direction: column;
  text-align: center;
  justify-content: center;
  align-items: center;
  height: 100%;
}

h1 {
  line-height: 0%;
  padding: 0;
  margin-top: 90px;
  font-family: "Microsoft Yahei";
  font-weight: bold;
  color: #1890ff;
  letter-spacing: 8px;
  font-size: 70px;
  text-align: center;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

h3 {
  padding: 0;
  font-family: "Microsoft Yahei";
  font-weight: bold;
  color: #1890ff;
  letter-spacing: 4px;
  font-size: 26px;
  text-align: center;
  opacity: 0.8;
}

@keyframes slideInFromBottom {
  0% {
    transform: translateY(100%);
    opacity: 0;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

.animated-text {
  animation: slideInFromBottom 1s ease forwards;
}

.btn-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20vh;
}

.btn-hover {
  width: 200px;
  font-size: 16px;
  font-weight: 600;
  color: #fff;
  cursor: pointer;
  margin: 20px;
  height: 55px;
  text-align: center;
  border: none;
  background-size: 300% 100%;
  border-radius: 50px;
  transition: all 0.4s ease-in-out;
}

.btn-hover:hover {
  background-position: 100% 0;
  transition: all 0.4s ease-in-out;
}

.btn-hover:focus {
  outline: none;
}

.btn-hover.btn-color {
  background-image: linear-gradient(
      to right,
      #1890ff,
      #40a9ff,
      #69c0ff,
      #91d5ff
  );
  box-shadow: 0 4px 15px 0 rgba(24, 144, 255, 0.5);
}

.custom-dialog :deep(.el-dialog) {
  border-radius: 15px;
  overflow: hidden;
}

.dialog-header {
  text-align: center;
  padding: 20px 0;
}

.dialog-header h2 {
  margin: 0;
  color: #333;
  font-size: 24px;
}

.dialog-header p {
  margin: 10px 0 0;
  color: #666;
  font-size: 14px;
}

.login-form {
  padding: 20px 30px;
}

.input-icon {
  color: #909399;
}

.dialog-footer {
  display: flex;
  justify-content: space-between;
  margin-top: 30px;
}

.dialog-btn {
  flex: 1;
  margin: 0 10px;
  height: 40px;
  font-size: 16px;
  border-radius: 20px;
}

.cancel-btn {
  background: #f5f7fa;
  border-color: #dcdfe6;
  color: #606266;
}

.confirm-btn {
  background-image: linear-gradient(to right, #1890ff, #69c0ff);
  border: none;
}

.confirm-btn:hover {
  background-image: linear-gradient(to right, #40a9ff, #91d5ff);
}

:deep(.el-input__wrapper) {
  border-radius: 20px;
  padding: 1px 15px;
}

:deep(.el-form-item) {
  margin-bottom: 20px;
}
</style>
