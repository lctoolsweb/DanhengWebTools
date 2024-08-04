<template>
  <div>
    <div class="scrolling-notice" v-if="showNotice">
      <marquee behavior="scroll" direction="left">{{ t('main.notice') }}</marquee>
    </div>

    <div class="commuse">
      <div class="commuse-item">
        <div class="text-neutral-400 dark:text-slate-100">{{ t('commuse.item') }}</div>
        <a-cascader allow-search v-model="value2" :options="options" placeholder="{{ t('main.item') }}" filterable />
      </div>

      <div class="commuse-item">
        <div class="text-stone-700 dark:text-slate-100">{{ t('commuse.number') }}</div>
        <a-input-number v-model="num" placeholder="{{ t('main.number') }}" mode="button" size="large" class="input-demo" />
      </div>

      <div class="generate">
        <a-input v-model="value" placeholder="" disabled />
        <a-button type="primary" shape="round" size="large" @click="copyvalue">{{ t('main.copy') }}</a-button>
        <a-button type="primary" shape="round" size="large" @click="execute">执行</a-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useClipboard } from '@vueuse/core';
import { Message } from '@arco-design/web-vue';
import axios from 'axios';
import {useI18n} from "vue-i18n";
const { t, locale } = useI18n();

const value2 = ref(1);
const num = ref(1000);

const value = computed(() => {
  return `give ${value2.value} x${num.value}`;
});

const message = Message;

const options = ref([] as { label: string; value: number }[]);

function copyvalue() {
  const { copy, isSupported } = useClipboard();
  copy(value.value);
  if (isSupported) {
    message.success(t('main.success', { value: value.value }));
  }
}

const API_BASE_URL = import.meta.env.VITE_DHWT_API_SERVER;

const execute = async () => {
  //读取localStorage中存储的uid
  const uid = localStorage.getItem('uid');

  if (!uid) {
    message.info('用户未登录，请先前往“远程”页面执行一次命令，然后重试');
    return;
  }

  const command = value.value;

  try {
    // 发送请求到后端
    const res = await axios.post(`${API_BASE_URL}/api/submit`, {
      keyType: 'PEM',  
      uid: uid,
      command: command
    });

    if (res.data.code !== 0) {
      throw new Error('命令提交失败: ' + res.data.message);
    }
    const responseMessage = res.data.data.message;
    message.success(`命令提交成功：${res.data.data.message}`);
  } catch (err: unknown) {
    const errorMessage = err instanceof Error ? err.message : '请求失败';
    message.error(errorMessage);
    console.error(err);
  }
};

const showNotice = ref(true);

onMounted(() => {
  locale.value = navigator.language.includes('zh') ? 'zh' : 'en';

  import('./json/zh/commuse.json').then((commusezh) => {
    import('./json/en/commuse.json').then((commuseen) => {
      options.value = locale.value === 'zh' ? commusezh.default : commuseen.default;

      setTimeout(() => {
        showNotice.value = true;
      }, 1000);
    });
  });
});
</script>

<style lang="less" scoped>
.scrolling-notice {
  color: #BEBEBE;
  padding: 8px;
  font-size: 14px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  border-radius: 10px;
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
