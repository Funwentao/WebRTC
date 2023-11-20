<script setup lang="ts">
import 'webrtc-adapter'
import { ref } from 'vue'
import type { Ref } from 'vue'

const local: Ref<HTMLVideoElement | null> = ref(null)
const remote: Ref<HTMLVideoElement | null> = ref(null)
const pc = new RTCPeerConnection({
  iceServers: [{ urls: 'stun:stun.voipbuster.com ' }]
})
navigator.mediaDevices
  .getUserMedia({
    audio: true,
    video: true
  })
  .then((loaclStream) => {
    if (local.value) {
      local.value.srcObject = loaclStream
      loaclStream.getTracks().forEach((track) => pc.addTrack(track, loaclStream))
    }
  })
pc.ontrack = (ev) => {
  if (remote.value) {
    remote.value.srcObject = ev.streams[0]
  }
}

const answerSdp = ref('')
const offerSdp2 = ref('')

const createAnswer = async () => {
  const offer = JSON.parse(offerSdp2.value)
  pc.onicecandidate = async (evt) => {
    if (evt.candidate) {
      answerSdp.value = JSON.stringify(pc.localDescription)
    }
  }
  await pc.setRemoteDescription(offer)
  const answer = await pc.createAnswer()
  await pc.setLocalDescription(answer)
}

function copyToClipboard(val: string) {
  navigator.clipboard.writeText(val)
}
</script>

<template>
  <a-space direction="vertical" size="middle">
    <div id="local">
      <video ref="local" autoplay playsinline muted width="300" height="300"></video>
      <div id="remote">
        <video ref="remote" autoplay playsinline width="300" height="300"></video>
      </div>
    </div>
    <div>
      <div class="mb20">
        输入老师端创建的offer:
        <br />
        <a-input-group compact>
          <a-input v-model:value="offerSdp2" style="width: calc(100% - 200px)" />
          <a-button type="primary" @click="createAnswer">创建answer</a-button>
        </a-input-group>
      </div>
      <div class="mb20">
        <a-input-group compact>
          <a-input v-model:value="answerSdp" style="width: calc(100% - 200px)" />
          <a-button type="primary" @click="copyToClipboard(answerSdp)">点击复制</a-button>
        </a-input-group>
      </div>
    </div>
  </a-space>
</template>
<style scoped>
#local {
  width: 1080px;
  height: 700px;
  padding: 20px;
  position: relative;
}
#local video {
  width: 100%;
  height: 100%;
  object-fit: fill;
}
#remote {
  width: 200px;
  height: 200px;
  position: absolute;
  right: 30px;
  bottom: 30px;
  z-index: 1;
}
#remote video {
  width: 100%;
  height: 100%;
  object-fit: fill;
}
.mb20 {
  margin-bottom: 20px;
}
</style>