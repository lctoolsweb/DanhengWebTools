<template>
  <div>
    <div :style="{ height: '20px' }"></div>
    <a-form :model="form" class="form-container" @submit="handleSubmit">
      <a-form-item field="keyType" label="Key Type">
        <a-input v-model="form.keyType" placeholder="请输入Key Type..." />
      </a-form-item>
      <a-form-item field="uid" label="UID">
        <a-input v-model="form.uid" placeholder="请输入玩家UID..." />
      </a-form-item>
      <a-form-item field="command" label="Command">
        <a-input v-model="form.command" placeholder="请输入命令..." />
      </a-form-item>
      <a-form-item>
        <a-space>
          <a-button type="primary" shape="round" size="large" html-type="submit">提交</a-button>
          <a-button type="primary" shape="round" size="large" html-type="reset" @click="handleReset">重置</a-button>
        </a-space>
      </a-form-item>
      <a-form-item label="response">
        <a-input v-model="responseData" :disabled="true" />
      </a-form-item>
    </a-form>
    
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue';
import { Message } from '@arco-design/web-vue';
import axios, { AxiosError } from 'axios';
const API_BASE_URL = import.meta.env.VITE_DHWT_API_SERVER;

type AlertType = 'success' | 'info' | 'warning' | 'error';

const form = reactive({
  keyType: 'PEM',
  uid: '',
  command: ''
});

const responseData = ref('');
const showMessage = ref(false);
const messageType = ref<AlertType>('info');
const message = ref('');

const handleReset = () => {
  localStorage.clear();
  form.keyType = "PEM";
  form.uid = "";
  form.command = "";
  responseData.value = "";
  showMessage.value = false;
  messageType.value = 'info';
  message.value = "";
};

const handleSubmit = async (data: { values: Record<string, any>; errors: Record<string, any> | undefined }, ev: Event) => {
  ev.preventDefault();

  try {
    const res = await axios.post(`${API_BASE_URL}/api/submit`, {
      keyType: form.keyType,
      uid: form.uid,
      command: form.command
    });

    
    const responseMessage = res.data.message; 
    if (res.data.code === 0) {
      localStorage.setItem('uid', form.uid);
      console.log('UID stored:', localStorage.getItem('uid'));
}

    responseData.value = JSON.stringify(res.data, null, 2);
    showMessage.value = true;
    messageType.value = 'success';
    message.value = `命令提交成功：${responseMessage}`;
    Message.success(`命令提交成功：${responseMessage}`);
  } catch (err: unknown) {
    let errorMessage = '请求失败';
    if (axios.isAxiosError(err)) {
      if (err.response) {
        if (err.response.status === 429) {
          errorMessage = '请求速率过快，请稍后重试';
        } else {
          errorMessage = (err.response.data as { error?: string }).error || err.response.statusText;
        }
      } else {
        errorMessage = err.message;
      }
    }

    showMessage.value = true;
    messageType.value = 'error';
    message.value = errorMessage;
    Message.error(errorMessage);
    console.error(err);
  }
};
</script>

<style lang="less" scoped>
.form-container {
  margin-top: 20px;
  width: 600px;
  margin: auto;
}

a-form-item {
  margin-bottom: 16px;
}

a-input-group {
  display: flex;
  align-items: center;
}

a-input-group > a-input {
  margin-right: 8px;
}

.input-small {
  width: 80px;
}

.input-medium {
  width: 160px;
}

@media screen and (max-width: 768px) {
  .form-container {
    width: 100%;
    padding: 10px;
  }

  a-input-group {
    flex-direction: column;
    align-items: flex-start;
  }

  a-input-group > a-input {
    margin-right: 0;
    margin-bottom: 8px;
    width: 100%;
  }

  .input-small,
  .input-medium {
    width: 100%;
  }

  a-space {
    display: flex;
    flex-direction: column;
    width: 100%;
  }

  a-space > .arco-button {
    width: 100%;
    margin-top: 10px;
  }

  a-form-item {
    margin-bottom: 12px;
  }
}
</style>
