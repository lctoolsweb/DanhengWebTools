<template>
  <div>
    <!-- 滚动公告 -->
    <div class="scrolling-notice" v-if="showNotice">
      <marquee behavior="scroll" direction="left">{{ t('main.notice') }}</marquee>
    </div>

    <!-- 原有的组件内容 -->
    <div class="commuse">
      

      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">{{ t('monster.monster1') }}</div>
        <a-cascader allow-search v-model="value2" :options="options" placeholder="" filterable />
      </div>

      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">{{ t('monster.monster2') }}</div>
        <a-cascader allow-search v-model="value3" :options="options2" placeholder="" filterable />
      </div>
      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">{{ t('monster.number') }}</div>
        <a-input-number v-model="amount" placeholder="" mode="button" size="large" class="input-demo" />
      </div>
      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">{{ t('monster.level') }}</div>
        <a-input-number v-model="level" placeholder="" mode="button" size="large" class="input-demo" />
      </div>
      <div class="commuse-item">
        <div class="text-slate-900 dark:text-slate-100">{{ t('monster.radius') }}</div>
        <a-input-number v-model="radius" placeholder="" mode="button" size="large" class="input-demo" />
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
import monster from './json/monster.json'
import stage from './json/stage.json'
import { useAppStore } from '@/store/modules/app'
import { useI18n } from 'vue-i18n'
import axios from 'axios'
import qs from 'qs'
const { text, isSupported, copy } = useClipboard()
const appStore = useAppStore()
const { t, locale } = useI18n()

var value2 = ref(1004010)
var value3 = ref(1)

const value = computed(() => {
  return `spawn ${value2.value} ${value3.value} x${amount.value} lv${level.value} r${radius.value}`
})

const options = ref([] as { label: string; value: number }[]);
const options2 = ref([] as { label: string; value: number }[]);
const amount = ref(1)
const level = ref(1)
const radius = ref(1)

const message = Message

function copyvalue() {
  copy(value.value)
  if (isSupported) {
    message.success(`已复制${value.value}`)
  }
}

const send: any = inject("send")

const showNotice = ref(true)
const noticeContent = 'LunarCore及其他任何衍生工具都是免费软件，如果你是付费购买的，那你就被骗了，请及时退款并举报。'

onMounted(() => {
  // 根据浏览器语言设置初始语言
  locale.value = navigator.language.includes('zh') ? 'zh' : 'en'

  // 导入中文和英文的 JSON 数据
  import('./json/zh/monster.json').then((monsterzh) => {
    import('./json/en/monster.json').then((monsteren) => {
      options.value = locale.value === 'zh' ? monsterzh.default : monsteren.default;
    });
  });

  import('./json/zh/stage.json').then((stagezh) => {
    import('./json/en/stage.json').then((stageen) => {
      options2.value = locale.value === 'zh' ? stagezh.default : stageen.default;
    });
  });

  setTimeout(() => {
    showNotice.value = true
  }, 1000);
});
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

  >div {
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
