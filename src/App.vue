<script setup>
import { ref, watch } from 'vue';
import Login from './components/Login.vue';
import ItineraryView from './components/ItineraryView.vue';

const loginComponent = ref(null);

watch(() => loginComponent.value?.isLoggedIn, (newVal) => {
  console.log('登录状态变更:', newVal);
  console.log('当前登录组件状态:', {
    isLoggedIn: loginComponent.value?.isLoggedIn,
    isGirl: loginComponent.value?.isGirl
  });
}, { immediate: true });
</script>

<template>
  <Login ref="loginComponent" />
  <ItineraryView 
    v-if="loginComponent?.isLoggedIn"
    v-model:isGirl="loginComponent.isGirl"
  />
</template>

<style>
#app {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #fff5f7;
  min-height: 100vh;
}

body {
  margin: 0;
  padding: 0;
}
</style>
