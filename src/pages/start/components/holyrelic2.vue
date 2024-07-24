<template>
  <div>
    
    <div class="scrolling-notice" v-if="showNotice">
      <marquee behavior="scroll" direction="left">{{ noticeContent }}</marquee>
    </div>

    
    <div class="commuse">

      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">选择角色: </div>
        <a-cascader allow-search v-model="value2" :options="options" placeholder="请输入物品" filterable />
      </div>

      <div class="generate">
        <a-input v-model="value" placeholder="" />
        <a-button type="primary" shape="round" size="large" @click="copyvalue">复制</a-button>
        <a-button type="primary" shape="round" size="large" @click="execute">执行</a-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed, inject, onMounted } from 'vue'
import { useClipboard } from '@vueuse/core'
import { Message } from '@arco-design/web-vue'
import page1 from './json/holyrelic2.json'
import { useAppStore } from '@/store/modules/app'
import axios from 'axios'
import qs from 'qs'
import JSEncrypt from 'jsencrypt';
const { text, isSupported, copy } = useClipboard()
const appStore = useAppStore()

var value2 = ref('先选择角色再选择遗器部位')
var value3 = ref('relic')

const value = computed(() => {
  
  // 删除6位标识字符 括号内的标识字符均为角色代码 如果以后一个角色有更多的套装搭配 可以更改括号类型
  const sixsixsix = value2.value.slice(6);

  return `${value3.value} ${sixsixsix}`
})

const options = reactive(page1)

const message = Message
const ADMIN_KEY = import.meta.env.VITE_DANHENG_ADMIN_KEY;
const API_BASE_URL = import.meta.env.VITE_DANHENG_DISPATCH_SERVER;

function copyvalue() {
  copy(value.value)
  if (isSupported) {
    message.success(`已复制${value.value}`)
  }
}

const send: any = inject("send")

const showNotice = ref(true)
const noticeContent = 'LunarCore及其他任何衍生工具都是免费软件，如果你是付费购买的，那你就被骗了，请及时退款并举报。'

// 在页面加载时设置一个延时，用于显示滚动公告，你可以根据需求调整延时时长
onMounted(() => {
  setTimeout(() => {
    showNotice.value = true
  }, 1000)
})

const execute = async () => {
  const uid = localStorage.getItem('lastSubmittedUid');

  if (!uid) {
    message.info('用户未登录,请先前往”远程“页面执行一次命令，然后重试');
    return;
  }

  const command = value.value;

  try {
    // Step 1: 获取授权
    const authRes = await axios.post(`${API_BASE_URL}/muip/auth_admin`, {
      admin_key: ADMIN_KEY,
      key_type: 'PEM'
    });

    if (authRes.data.code !== 0) {
      throw new Error('授权失败: ' + authRes.data.message);
    }

    const { rsaPublicKey, sessionId } = authRes.data.data;

    // Step 2: 使用rsaPublicKey加密命令
    const encrypt = new JSEncrypt();
    encrypt.setPublicKey(rsaPublicKey);
    const encryptedCommand = encrypt.encrypt(command);

    if (!encryptedCommand) {
      throw new Error('命令加密失败');
    }

    // Step 3: 提交命令
    const execCmdRes = await axios.post(`${API_BASE_URL}/muip/exec_cmd`, {
      SessionId: sessionId,
      Command: encryptedCommand,
      TargetUid: uid
    });

    if (execCmdRes.data.code !== 0) {
      throw new Error('命令提交失败: ' + execCmdRes.data.message);
    }

    const decodedMessage = base64Decode(execCmdRes.data.data.message);

    message.success(`命令提交成功：${decodedMessage}`);
  } catch (err: unknown) {
    const errorMessage = err instanceof Error ? err.message : '请求失败';
    message.error(errorMessage);
    console.error(err);
  }
};

const base64Decode = (str: string): string => {
  try {
    return decodeURIComponent(escape(window.atob(str)));
  } catch (e) {
    console.error('Base64解码失败:', e);
    return '解码失败';
  }
};


</script>

<style lang="less" scoped>
/* 添加样式以美化滚动公告 */
.scrolling-notice {
  color: #BEBEBE;
  padding: 8px;
  font-size: 14px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  border-radius: 10px; /* 添加圆角样式 */
}

.commuse {
  width: 500px;
  margin: auto;
}

.commuse-item {
  display: flex;
  align-items: center;
  color: #000;
  margin: 18px 0;

  > div {
    &:nth-child(1) {
      width: 150px;
      text-align: right;
      padding-right: 10px;
      color: #6b6a6a !important;
    }
  }
}

.generate {
  display: flex;
  align-items: center;
  margin-left: 100px;
}
@media screen and (max-width: 768px) {
  .commuse {
    width: 100%; 
    padding: 10px; 
  }

  .commuse-item {
    margin: 18px 0 10px; 
  }

  .commuse-item > div:nth-child(1) {
    width: auto; 
    text-align: left; 
    padding: 0; 
    margin-bottom: 5px; 
  }

  .generate {
    display: block; 
    margin-left: 0; 
    width: 100%; 
    margin-bottom: 80px; 
    margin-top: 10px; 
    text-align: center; 
  }
  .generate > .arco-input {
    margin-bottom: 10px; 
  }
  .generate button { 
    display: block; 
    width: 100%; 
    margin-top: 10px; 
    
  }
}
</style>
