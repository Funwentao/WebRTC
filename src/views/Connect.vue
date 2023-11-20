<script setup lang="ts">
import 'webrtc-adapter'
import {ref} from 'vue';
import type {Ref} from 'vue';
import {Input as aInput, InputGroup as aInputGroup, Button as aButton} from 'ant-design-vue'

const local:Ref<HTMLVideoElement | null> = ref(null);
const remote:Ref<HTMLVideoElement | null> = ref(null);
const pc = new RTCPeerConnection({
  iceServers: [{ urls: 'stun:stun.voipbuster.com ' }],
});
navigator.mediaDevices.getDisplayMedia({
    audio: true,
    video: true,
}).then(loaclStream => {
    if (local.value) {
        local.value.srcObject = loaclStream;
        loaclStream.getTracks().forEach(track => pc.addTrack(track, loaclStream))
    }
})
pc.ontrack = ev => {
    if(remote.value) {
        remote.value.srcObject = ev.streams[0]
    }
}

const offerSdp = ref('')
const answerSdp = ref('')
const offerSdp2 = ref('')
const answerSdp2 = ref('')

const createOffer = async() => {
    const offer = await pc.createOffer()
    await pc.setLocalDescription(offer)
    pc.onicecandidate = async(evt) => {
        if(evt.candidate) {
            offerSdp.value = JSON.stringify(pc.localDescription)
        }
    }
}

const createAnswer = async() => {
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

const addAnswer = async() => {
    const answer = JSON.parse(answerSdp2.value)
    if (!pc.currentLocalDescription) {
        pc.setRemoteDescription(answer)
    }
}

function copyToClipboard(val: string) {
  navigator.clipboard.writeText(val)
}
</script>

<template>
  <main>
    <video id="local" ref="local" autoplay playsinline muted width="300" height="300"></video>
    <video id="remote" ref="remote" autoplay playsinline width="300" height="300"></video>
    <div>
        <a-button @click="createOffer">创建offer</a-button>
    </div>
    <div>
        <a-input-group compact>
            <a-input v-model:value="offerSdp"/>
            <a-button type="primary" @click="copyToClipboard(offerSdp)">点击复制</a-button>
        </a-input-group>
    </div>
    <div>
        <a-input-group compact>
            <a-input v-model:value="offerSdp2"/>
            <a-button type="primary" @click="createAnswer">创建answer</a-button>
        </a-input-group>
        <a-input-group compact>
            <a-input v-model:value="answerSdp"/>
            <a-button type="primary" @click="copyToClipboard(answerSdp)">点击复制</a-button>
        </a-input-group>
    </div>
    <div>
        <a-input-group compact>
            <a-input v-model:value="answerSdp2"/>
            <a-button type="primary" @click="addAnswer">add answer</a-button>
        </a-input-group>
    </div>
  </main>
</template>
