<template>
  <a-layout class="layout">
    <a-layout-header>
      <div class="logo" />
      <a-menu
        v-model:selectedKeys="selectedKeys"
        theme="dark"
        mode="horizontal"
        :style="{ lineHeight: '64px' }"
      >
        <a-menu-item key="2">
            <router-link to="/">摄像头</router-link>
        </a-menu-item>
        <a-menu-item key="1">
           <router-link to="/desktop">桌面分享</router-link>
        </a-menu-item>
        <a-menu-item key="3">
           <router-link to="/teacher">老师端 / 分享桌面</router-link>
        </a-menu-item>
        <a-menu-item key="4">
           <router-link to="/student">学生端 / 分享摄像头</router-link>
        </a-menu-item>
      </a-menu>
    </a-layout-header>
    <a-layout-content style="padding: 0 50px">
      <!-- <a-breadcrumb style="margin: 16px 0">
        <a-breadcrumb-item>
          <router-link to="/desktop">桌面分享</router-link>
        </a-breadcrumb-item>
        <a-breadcrumb-item>
          <router-link to="/">桌面分享</router-link>
        </a-breadcrumb-item>
        <a-breadcrumb-item>
          <router-link to="/connect">模拟上课</router-link>
        </a-breadcrumb-item>
      </a-breadcrumb> -->
      <main class="main">
        <router-view></router-view>
      </main>
    </a-layout-content>
    <a-layout-footer style="text-align: center">
      @WebRTC
    </a-layout-footer>
  </a-layout>
</template>
<script lang="ts" setup>
import { ref } from 'vue';
const selectedKeys = ref<string[]>(['2']);
</script>
<style scoped>
.site-layout-content {
  height: 100vh;
  padding: 24px;
  background: #fff;
}
.layout {
  height: 100vh;;
}
#components-layout-demo-top .logo {
  float: left;
  width: 100vw;
  height: 31px;
  margin: 16px 24px 16px 0;
  background: rgba(255, 255, 255, 0.3);
}
.ant-row-rtl #components-layout-demo-top .logo {
  float: right;
  margin: 16px 0 16px 24px;
}

[data-theme='dark'] .site-layout-content {
  background: #141414;
}
.main {
  width: 100%;
}
</style>
