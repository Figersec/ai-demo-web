<template>
  <div>
    <!-- 录音控制按钮 -->
    <button @click="startRecording" :disabled="isRecording">开始录音</button>
    <button @click="stopRecording" :disabled="!isRecording">停止录音</button>

    <!-- 状态提示 -->
    <div v-if="status" class="status">{{ status }}</div>

    <!-- 上传进度条 -->
    <div v-if="uploadProgress > 0 && uploadProgress < 100">
      上传进度: {{ uploadProgress }}%
    </div>

    <!-- 错误提示 -->
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
  </div>
</template>

<script setup>
import {onBeforeUnmount, ref} from 'vue';
import axios from 'axios';

const isRecording = ref(false);
const audioChunks = ref([]);
const mediaRecorder = ref(null);
const uploadProgress = ref(0);
const status = ref('');
const errorMessage = ref('');

// 1. 开始录音
const startRecording = async () => {
  try {
    // 获取麦克风权限
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

    // 初始化 MediaRecorder（设置为 WebM 格式）
    mediaRecorder.value = new MediaRecorder(stream, { mimeType: 'audio/webm' });

    // 监听数据块生成事件
    mediaRecorder.value.ondataavailable = (event) => {
      if (event.data.size > 0) {
        audioChunks.value.push(event.data);
      }
    };

    // 录音停止时触发上传
    mediaRecorder.value.onstop = () => {
      const audioBlob = new Blob(audioChunks.value, { type: 'audio/webm' });
      uploadToServer(audioBlob);
      audioChunks.value = []; // 清空数据
      status.value = '录音已停止';
    };

    mediaRecorder.value.start(100); // 每 100ms 生成一个数据块
    isRecording.value = true;
    status.value = '正在录音...';
  } catch (error) {
    errorMessage.value = '无法访问麦克风，请检查权限！';
  }
};

// 2. 停止录音
const stopRecording = () => {
  if (mediaRecorder.value && mediaRecorder.value.state === 'recording') {
    mediaRecorder.value.stop();
    isRecording.value = false;
  }
};

// 3. 上传到后端（接收原始 WebM 文件）
const uploadToServer = async (blob) => {
  try {
    // 创建 FormData 对象
    const formData = new FormData();
    const fileName = `recording-${Date.now()}.wav`;
    formData.append('file', new File([blob], fileName, { type: 'audio/wav' }));

    // 发送 POST 请求（替换为你的后端接口地址）
    const response = await axios.post(
        'http://localhost:10086/spring-ai-demo/speech/realTimeStreamSpeechToText', // 后端接收接口路径
        formData,
        {
          headers: {
            'Content-Type': 'multipart/form-data'
          },
          onUploadProgress: (progressEvent) => {
            const percent = Math.round((progressEvent.loaded * 100) / progressEvent.total);
            uploadProgress.value = percent;
          }
        }
    );

    if (response.status === 200) {
      status.value = '上传成功！';
      errorMessage.value = '';
    } else {
      throw new Error('上传失败，请重试');
    }
  } catch (error) {
    errorMessage.value = error.message || '未知错误，请重试';
  }
};

// 4. 组件卸载时清理资源
const cleanup = () => {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop();
    mediaRecorder.value = null;
  }
};

// 监听组件销毁事件
onBeforeUnmount(() => {
  cleanup();
});
</script>

<style scoped>
.status {
  color: #4CAF50;
  margin: 10px 0;
}

.error {
  color: #FF5722;
  margin: 10px 0;
}

button {
  padding: 10px 20px;
  margin: 5px;
  font-size: 16px;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
</style>