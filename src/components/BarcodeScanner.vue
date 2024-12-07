<template>
  <div>
    <h2>条形码扫描</h2>
    <video ref="videoElement" width="300" height="200"></video>
    <button @click="captureImage">拍照</button>
    <canvas ref="canvasElement" style="display: none;"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { BrowserMultiFormatReader } from '@zxing/library';

const props = defineProps({
  customerName: String
});

const videoElement = ref(null);
const canvasElement = ref(null);
const reader = new BrowserMultiFormatReader();

onMounted(async () => {
  // 请求摄像头权限
  const stream = await navigator.mediaDevices.getUserMedia({ video: true });
  videoElement.value.srcObject = stream;
  videoElement.value.play();
  startScanning();
});

const startScanning = () => {
  const interval = setInterval(async () => {
    try {
      const result = await reader.decodeFromVideoDevice(null, videoElement.value);
      console.log(result);
      // 处理条形码结果
      await saveImage(result.text);
      clearInterval(interval);
    } catch (err) {
      // 忽略错误
    }
  }, 1000);

  onBeforeUnmount(() => {
    clearInterval(interval);
  });
};

const captureImage = () => {
  const context = canvasElement.value.getContext('2d');
  context.drawImage(videoElement.value, 0, 0, canvasElement.value.width, canvasElement.value.height);
  const imageData = canvasElement.value.toDataURL('image/png');

  // 在这里可以实现识别条形码并上传图片到 API
  // 假设 saveImage 是一个函数，用于保存图片
  saveImage(imageData);
};

const saveImage = async (imageData) => {
  const response = await fetch('YOUR_API_ENDPOINT', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      name: props.customerName, // 使用客户名称
      image: imageData,
    }),
  });

  if (response.ok) {
    alert('图片上传成功');
  } else {
    alert('图片上传失败');
  }
};
</script>

<style>
/* 可以在这里添加一些样式 */
</style>
