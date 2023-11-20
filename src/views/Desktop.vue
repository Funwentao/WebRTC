<script setup lang="ts">
import {ref} from 'vue';
import type {Ref} from 'vue';
const video:Ref<HTMLVideoElement | null> = ref(null);
const onClick = () => {
  navigator.mediaDevices.getDisplayMedia({
    audio: true,
    video: true,
  }).then(loaclStream => {
    if (video.value) {
      video.value.srcObject = loaclStream;
    }
  })
}


</script>

<template>
  <a-flex gap="middle" horizontal>
    <div class="video-content">
      <video autoplay playsinline muted ref="video"></video>
    </div>
    <div class="photo-content">
    </div>
  </a-flex>
  <a-divider style="height: 2px; background-color: #1677ff" />
  <div class="btn-content">
      <a-button @click="onClick">分享桌面</a-button>
  </div>
</template>

<style scoped>
.video-content {
  width: 800px;
  height: 500px;
}
video {
  width: 100%;
  height: 100%;
}
.btn-content {
  text-align: center;
}
</style>