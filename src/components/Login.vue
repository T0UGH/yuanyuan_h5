<script setup>
import { ref } from 'vue';

const password = ref('');
const isLoggedIn = ref(false);
const errorMessage = ref('');
const isGirl = ref(false);

const handleLogin = () => {
  console.log('尝试登录，输入密码:', password.value);
  if (password.value === '123' || password.value === '456') {
    isLoggedIn.value = true;
    isGirl.value = password.value === '123';
    errorMessage.value = '';
    console.log('登录成功，isLoggedIn:', isLoggedIn.value, 'isGirl:', isGirl.value);
  } else {
    errorMessage.value = '密码错误';
    console.log('登录失败，密码错误');
  }
};

defineExpose({
  isLoggedIn,
  isGirl
});
</script>

<template>
  <div class="login-container" v-if="!isLoggedIn">
    <div class="login-box">
      <h2>欢迎来到武汉樱花之旅</h2>
      <div class="input-group">
        <input 
          type="password" 
          v-model="password"
          placeholder="请输入密码"
          @keyup.enter="handleLogin"
        >
        <button @click="handleLogin">登录</button>
      </div>
      <p class="error-message" v-if="errorMessage">{{ errorMessage }}</p>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #ffd6e7, #ffb6c1);
  z-index: 1000;
}

.login-box {
  background: rgba(255, 255, 255, 0.9);
  padding: 2rem;
  border-radius: 1rem;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 80%;
  max-width: 400px;
}

h2 {
  color: #ff69b4;
  margin-bottom: 2rem;
  font-size: 1.5rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

input {
  padding: 0.8rem;
  border: 2px solid #ffb6c1;
  border-radius: 0.5rem;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.3s;
}

input:focus {
  border-color: #ff69b4;
}

button {
  padding: 0.8rem;
  background-color: #ff69b4;
  color: white;
  border: none;
  border-radius: 0.5rem;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #ff1493;
}

.error-message {
  color: #ff4444;
  margin-top: 1rem;
  font-size: 0.9rem;
}
</style>