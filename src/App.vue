<template>
  <div class="root">
    <div class="content">
      <!-- 摄像头扫码识别 -->
      <div class="camera">
        <video ref="video" id="video" autoplay></video>
        <div class="border">
          <div class="item-1 item"></div>
          <div class="item-2 item"></div>
          <div class="item-3 item"></div>
          <div class="item-4 item"></div>
          <div class="plate-rank-content item-5">
            <div class="angle-border left-top-border"></div>
            <div class="angle-border right-top-border"></div>
            <div class="angle-border left-bottom-border"></div>
            <div class="angle-border right-bottom-border"></div>
            <div class="solid"></div>
          </div>
        </div>
      </div>
      <div class="btn">
        <button type="primary" @click="openCamera">
          开始扫码
        </button>
      </div>
    </div>
  </div>
  <ImagePreview :image-blob="imageBlob" v-if="imageBlob"></ImagePreview>
</template>
<script setup lang="ts">
import {reactive, onBeforeUnmount, ref} from "vue";
import {BrowserMultiFormatReader} from "@zxing/library"
import ImagePreview from "@/components/ImagePreview.vue";

const imageBlob = ref(null)
let codeReader = reactive(new BrowserMultiFormatReader());
let openScan = async () => {
  codeReader.listVideoInputDevices().then((videoInputDevices) => {
    console.log("videoInputDevices", videoInputDevices, "摄像头设备");
    // 默认获取第一个摄像头设备id
    let firstDeviceId = videoInputDevices[0].deviceId;  // 根据id选择摄像头
    decodeFromInputVideoFunc(firstDeviceId);
  }).catch((err: any) => {
    alert(err)
  });
};
let decodeFromInputVideoFunc = (firstDeviceId: any) => {
  // decodeFromInputVideoDeviceContinuously 已弃用，使用 decodeFromVideoDevice
  // firstDeviceId为null 时默认选择面向环境的摄像头
  codeReader.decodeFromVideoDevice(null, "video", (result: any, err: any) => {
        if (result) {

          saveImg()


          // 停止二维码识别
          codeReader.reset();
        }
        if (err && !err) {
          console.error(err);
        }
      }
  );
};
onBeforeUnmount(() => {
  codeReader.reset();
});

const openCamera = () => {
  openScan()
}

const saveImg = () => {
  //将当前画面保存为图片
  const videoElement = document.getElementById("video");
  const canvas = document.createElement("canvas");
  canvas.width = videoElement.videoWidth;
  canvas.height = videoElement.videoHeight;

  const context = canvas.getContext("2d");
  context.drawImage(videoElement, 0, 0, canvas.width, canvas.height);

  // 将画布转换为图片并下载
  canvas.toBlob((blob) => {
    const url = URL.createObjectURL(blob);
    imageBlob.value = blob
    // const a = document.createElement('a');
    // a.href = url;
    // a.download = 'snapshot.png';
    // a.click();
    URL.revokeObjectURL(url); // 释放内存
  });
}

</script>

<style scoped lang="scss">
.root {
  display: flex;

  .content {
    width: 100%;

    .camera {
      height: 50vh;
      position: relative;

      #video {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }

      .border {
        position: absolute;
        top: 0;
        height: 100%;
        width: 100%;
        display: grid;
        grid-template-columns: 1fr 3fr 3fr 1fr;
        grid-template-rows: 0.1fr 0.1fr 0.1fr 1fr;
        grid-template-areas:
                    'a a a a'
                    'b e e c'
                    'b e e c'
                    'd d d d';

        .plate-rank-content {
          box-shadow: inset 0px 0px 10px 0px rgba(32, 116, 247, 0.3);
          border: 1px solid rgba(255, 255, 255, 0.4);
          position: relative;

          .solid {
            width: 95%;
            height: 1px;
            position: absolute;
            background-color: rgb(0, 119, 255);
            border-radius: 6px;
            animation: 1.5s linear 0s infinite alternate move_eye;
            left: 50%;
            transform: translateX(-50%);
          }

          @keyframes move_eye {
            0% {
              top: 3px;
            }

            50% {
              top: 50%;
            }

            100% {
              top: calc(100% - 6px);
            }
          }


          .angle-border {
            position: absolute;
            width: 24px;
            height: 24px;
          }

          .left-top-border {
            top: -3px;
            left: -3px;
            border-left: 3px solid red;
            border-top: 3px solid red;
          }

          .right-top-border {
            top: -3px;
            right: -3px;
            border-right: 3px solid red;
            border-top: 3px solid red;
          }

          .left-bottom-border {
            bottom: -3px;
            left: -3px;
            border-bottom: 3px solid red;
            border-left: 3px solid red;
          }

          .right-bottom-border {
            bottom: -3px;
            right: -3px;
            border-right: 3px solid red;
            border-bottom: 3px solid red;
          }

        }

        .item {
          background-color: rgba(128, 128, 128, .3);
        }

        .item-1 {
          grid-area: a;
        }

        .item-2 {
          grid-area: b;
        }

        .item-3 {
          grid-area: c;
        }

        .item-4 {
          grid-area: d;
        }

        .item-5 {
          grid-area: e;
        }
      }
    }

    .btn {
      text-align: center;
      margin-top: 10px;
    }
  }
}
</style>
