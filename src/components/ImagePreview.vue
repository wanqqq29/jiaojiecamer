<template>
  <img :src="imageUrl" style="max-width: 400px;max-height: 720px"/>
</template>

<script setup>
import {onBeforeUnmount, ref, watch} from "vue";

const props = defineProps({
  imageBlob: {
    type: Blob,
  }
})
const imageUrl = ref(null)


// 使用 props.imageBlob 来创建 URL
if (props.imageBlob) {
  imageUrl.value = URL.createObjectURL(props.imageBlob);
  const a = document.createElement('a');
  a.href = imageUrl.value;
  a.download = 'snapshot.png';
  a.click();
}

onBeforeUnmount(() => {
  if (imageUrl.value) {
    URL.revokeObjectURL(imageUrl.value);
  }
})


</script>

<style scoped>
.image-preview img {
  max-width: 100%;
  height: auto;
  margin-top: 10px;
}
</style>
