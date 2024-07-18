<template>
  <div class="commuse">
    <div class="commuse-item">
      <div class="label">Name</div>
      <a-input v-model="userName" placeholder="请输入姓名" class="input-demo" />
    </div>
    <div class="generate">
      <a-button type="primary" shape="round" size="large" @click="handleSubmit">确定</a-button>
      <a-button type="primary" shape="round" size="large" @click="copyResult" style="margin-left: 10px;">复制</a-button>
    </div>
    <div class="result">
      <a-textarea v-model="result" placeholder="生成的内容将显示在这里" readonly class="result-textarea"></a-textarea>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { Button as AButton, Input as AInput, Message } from '@arco-design/web-vue'
import { useClipboard } from '@vueuse/core'
import notesData from './json/note.json' 

const userName = ref('')
const result = ref('')

const { copy } = useClipboard()

const handleSubmit = () => {
  if (!userName.value) {
    Message.error('请输入姓名')
    return
  }

  // 从导入的 JSON 数据中随机选择一个 note
  const randomNoteObj = notesData[Math.floor(Math.random() * notesData.length)]
  const randomNote = randomNoteObj.note.replace('xxx', userName.value)
  result.value = randomNote
}

const copyResult = () => {
  if (result.value) {
    copy(result.value)
    Message.success('内容已复制')
  } else {
    Message.warning('没有内容可复制')
  }
}
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

.result {
  margin-top: 20px;
}

.result-textarea {
  width: 100%;
  height: 150px;
  padding: 10px;
  box-sizing: border-box;
  resize: none;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  color: #333;
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
    margin-top: 10px; 
    text-align: center; 
  }

  .generate > .arco-input {
    margin-bottom: 10px; 
  }

  .generate > .arco-button { 
    display: block; 
    width: 100%; 
    margin-top: 10px; 
    margin-left: 0 !important;
  }

  .result-textarea {
    width: 100%;
    height: 150px;
  }
}
</style>
