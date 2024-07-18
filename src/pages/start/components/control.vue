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
      <a-form-item label="返回数据">
        <a-input v-model="responseData" :disabled="true" />
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue';
import { Message } from '@arco-design/web-vue';
import axios from 'axios';
import JSEncrypt from 'jsencrypt';


const ADMIN_KEY = import.meta.env.VITE_DANHENG_ADMIN_KEY;
const API_BASE_URL = import.meta.env.VITE_DANHENG_DISPATCH_SERVER;

const form = reactive({
  adminKey: ADMIN_KEY,
  keyType: 'PEM',
  uid: '',
  command: ''
});

const responseData = ref('');
const showMessage = ref(false);
const messageType = ref<'error' | 'success' | ''>('');
const message = ref('');

const base64Decode = (str: string): string => {
  try {
    return decodeURIComponent(escape(window.atob(str)));
  } catch (e) {
    console.error('Base64解码失败:', e);
    return '解码失败';
  }
};

const handleReset = () => {
  localStorage.clear();
  form.keyType = "PEM";
  form.uid = "";
  form.command = "";
  responseData.value = "";
  showMessage.value = false;
  messageType.value = "";
  message.value = "";
};

const handleSubmit = async (data: { values: Record<string, any>; errors: Record<string, any> | undefined }, ev: Event) => {
  ev.preventDefault();

  try {
    // Step 1: 获取授权
    const authRes = await axios.post(`${API_BASE_URL}/muip/auth_admin`, {
      admin_key: form.adminKey,
      key_type: form.keyType
    });
    
    if (authRes.data.code !== 0) {
      throw new Error('授权失败: ' + authRes.data.message);
    }

    const { rsaPublicKey, sessionId } = authRes.data.data;

    // Step 2: 使用rsaPublicKey加密命令
    const encrypt = new JSEncrypt();
    encrypt.setPublicKey(rsaPublicKey);
    const encryptedCommand = encrypt.encrypt(form.command);

    if (!encryptedCommand) {
      throw new Error('命令加密失败');
    }

    // Step 3: 提交命令
    const execCmdRes = await axios.post(`${API_BASE_URL}/muip/exec_cmd`, {
      SessionId: sessionId,
      Command: encryptedCommand,
      TargetUid: form.uid
    });

    if (execCmdRes.data.code !== 0) {
      throw new Error('命令提交失败: ' + execCmdRes.data.message);
    }

    const decodedMessage = base64Decode(execCmdRes.data.data.message);

    responseData.value = JSON.stringify(execCmdRes.data, null, 2);
    showMessage.value = true;
    messageType.value = 'success';
    message.value = `命令提交成功：${decodedMessage}`;
    Message.success(`命令提交成功：${decodedMessage}`);
  } catch (err: unknown) {
    const errorMessage = err instanceof Error ? err.message : '请求失败';
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
