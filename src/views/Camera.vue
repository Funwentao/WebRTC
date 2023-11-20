<script setup lang="ts">
import { ref } from 'vue'
import type { Ref } from 'vue'
import { message } from 'ant-design-vue'
const [messageApi, contextHolder] = message.useMessage()

const video: Ref<HTMLVideoElement | null> = ref(null)
const img: Ref<HTMLImageElement | null> = ref(null)
let stream: MediaStream | null = null
const onOpenCamere = () => {
  navigator.mediaDevices
    .getUserMedia({
      audio: true,
      video: true
    })
    .then((loaclStream) => {
      if (video.value) {
        video.value.srcObject = loaclStream
        stream = loaclStream
      }
    })
}
const imgSrc = ref('')
const takePhoto = (filter?) => {
  if (!filter) {
    imgSrc.value = ''
  }
  const canvas = document.createElement('canvas')
  canvas.width = video.value.videoWidth / 2
  canvas.height = video.value.videoHeight / 2
  const ctx = canvas.getContext('2d')
  ctx.filter = filter ? filter : filterValue.value
  ctx.drawImage(imgSrc.value ? img.value : video.value, 0, 0, canvas.width, canvas.height)
  imgSrc.value = canvas.toDataURL('image/png')
}

const filterList = [
  '',
  'blur(5px)', // 模糊
  'brightness(0.5)', // 亮度
  'contrast(200%)', // 对比度
  'grayscale(100%)', // 灰度
  'hue-rotate(90deg)', // 色相旋转
  'invert(100%)', // 反色
  'opacity(90%)', // 透明度
  'saturate(200%)', // 饱和度
  'saturate(20%)', // 饱和度
  'sepia(100%)', // 褐色
  'drop-shadow(4px 4px 8px blue)' // 阴影
]

const filterValue = ref('')

const supportOptions: Ref<Array<{ label: string; value: string }>> = ref([])
const mimeTypes = ref('video/webm;codecs=vp9')
// 获取支持的媒体类型
function getSupportedMimeTypes() {
  const media = 'video'
  // 常用的视频格式
  const types = ['webm', 'mp4', 'ogg', 'mov', 'avi', 'wmv', 'flv', 'mkv', 'ts', 'x-matroska']
  // 常用的视频编码
  const codecs = ['vp9', 'vp9.0', 'vp8', 'vp8.0', 'avc1', 'av1', 'h265', 'h264']
  // 支持的媒体类型
  const supported: string[] = []
  const isSupported = MediaRecorder.isTypeSupported
  // 遍历判断所有的媒体类型
  types.forEach((type: string) => {
    const mimeType = `${media}/${type}`
    codecs.forEach((codec: string) =>
      [`${mimeType};codecs=${codec}`, `${mimeType};codecs=${codec.toUpperCase()}`].forEach(
        (variation) => {
          if (isSupported(variation)) supported.push(variation)
        }
      )
    )
    if (isSupported(mimeType)) supported.push(mimeType)
  })
  supportOptions.value = supported.map((item) => ({
    label: item,
    value: item
  }))
  return supported
}

getSupportedMimeTypes()
const selectType = (value) => {
  mimeTypes.value = value
}

const timer = ref(0)
let mediaRecorder: MediaRecorder
// 录制媒体流
function startRecord() {
  if (!stream) {
    messageApi.warning('请先获取本地音视频流')
    return
  }
  if (mediaRecorder?.state === 'recording') {
    mediaRecorder.stop()
    return
  }
  const options = {
    audioBitsPerSecond: 128000,
    videoBitsPerSecond: 2500000,
    mimeType: mimeTypes.value
  }
  const chunks: Blob[] = []
  let timerId: any
  mediaRecorder = new MediaRecorder(stream, options)
  mediaRecorder.start()

  mediaRecorder.ondataavailable = (e) => {
    chunks.push(e.data)
  }
  mediaRecorder.onstart = () => {
    timerId = setInterval(() => {
      timer.value++
    }, 1000)
  }
  mediaRecorder.onstop = (e: Event) => {
    // 停止录制
    timer.value = 0
    clearInterval(timerId)
    // 将录制的数据合并成一个 Blob 对象
    // const blob = new Blob(chunks, { type: chunks[0].type })
    const blob = new Blob(chunks, { type: mediaRecorder?.mimeType })
    downloadBlob(blob)
    chunks.length = 0
  }
}

// 下载 Blob
function downloadBlob(blob: Blob) {
  // 将 Blob 对象转换成一个 URL 地址
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  // 设置 a 标签的 href 属性为刚刚生成的 URL 地址
  a.href = url
  // 设置 a 标签的 download 属性为文件名
  a.download = `${new Date().getTime()}.${blob.type.split('/')[1].split(';')[0]}`
  // 模拟点击 a 标签
  a.click()
  // 释放 URL 地址
  URL.revokeObjectURL(url)
}
</script>

<template>
  <context-holder />
  <a-flex gap="middle" horizontal>
    <a-card class="video-content">
      <video autoplay playsinline muted ref="video"></video>
    </a-card>
    <a-card>
      <img :src="imgSrc" alt="" ref="img" />
    </a-card>
  </a-flex>
  <a-divider style="height: 2px; background-color: #1677ff" />
  <div class="btn-content">
    <div class="mb20">
      <a-button type="primary" @click="onOpenCamere">开启摄像头</a-button>
    </div>
    <div class="mb20">
      <a-button type="primary" @click="takePhoto()">拍照</a-button>
      选择滤镜：
      <a-select v-model:value="filterValue" @change="takePhoto" style="width: 120px">
        <a-select-option v-for="item in filterList" :value="item" :key="item">{{
          item
        }}</a-select-option>
      </a-select>
    </div>
    <div class="mb20">
      选择视频格式：
      <a-select v-model:value="mimeTypes" @change="selectType" style="width: 250px">
        <a-select-option v-for="item in supportOptions" :value="item.value" :key="item">{{
          item.label
        }}</a-select-option>
      </a-select>
      <a-button @click="startRecord">
        {{ timer === 0 ? ' 开始录制' : '终止录制 | ' + timer }}</a-button
      >
    </div>
  </div>
</template>

<style scoped>
.video-content {
  width: 960px;
}
video {
  width: 100%;
  object-fit: fill;
}
.btn-content {
  /* text-align: center; */
}
.mb20 {
    margin-bottom: 20px;
}
</style>