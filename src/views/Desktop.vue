<script setup lang="ts">
import {Button as aButton} from 'ant-design-vue';
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
  <main>
    <video autoplay playsinline muted ref="video" width="500" height="500"></video>
    <a-button @click="onClick">分享桌面</a-button>
  </main>

</template>
