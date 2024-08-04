<template>
    <div style="padding: 20px;">
      <!-- 提交表单卡片，宽度自适应全屏 -->
      <a-card title="提交表单" style="margin-bottom: 20px; width: 100%;">
        <a-form
          @submit="handleSubmit"
          :labelCol="{ span: 4 }"
          :wrapperCol="{ span: 20 }"
          layout="horizontal"
        >
          <!-- 昵称 -->
          <a-form-item label="昵称">
            <a-input v-model="formData.nickname" placeholder="请输入昵称" />
          </a-form-item>
          <!-- 描述 -->
          <a-form-item label="描述">
            <a-input v-model="formData.discribe" placeholder="请输入描述" />
          </a-form-item>
          <!-- 指令 -->
          <a-form-item label="指令">
            <a-input v-model="formData.command" placeholder="请输入指令" />
          </a-form-item>
          <!-- 提交按钮 -->
          <a-form-item :wrapperCol="{ offset: 4, span: 20 }">
            <a-button type="primary" html-type="submit">提交</a-button>
          </a-form-item>
        </a-form>
      </a-card>
  
      <!-- 数据展示部分 -->
      <a-row :gutter="20" class="data-display">
        <a-col
          v-for="item in data"
          :key="item.id"
          :xs="24"
          :sm="12"
          :md="8"
          :lg="6"
          :xl="4"
        >
          <a-card class="data-card">
            <template #title>
              ID: {{ item.id }}
            </template>
            <p>指令: {{ item.command }}</p>
            <p>描述: {{ item.discribe }}</p>
            <p>上传者: {{ item.nickname }}</p>
            <a-button @click="copyToClipboard(item.command)">复制指令</a-button>
          </a-card>
        </a-col>
      </a-row>
    </div>
  </template>
  
  <script>
  import { Message } from '@arco-design/web-vue';
  
  export default {
    data() {
      return {
        formData: {
          nickname: '',
          discribe: '',
          command: '',
        },
        data: [],
      };
    },
    mounted() {
      this.fetchData();
    },
    methods: {
      async handleSubmit() {
        try {
          await fetch('https://srgm.dreamplace.cn/submit', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify(this.formData),
          });
          Message.success('提交成功');
          this.fetchData();
          this.formData = { nickname: '', discribe: '', command: '' };
        } catch (error) {
          Message.error('提交失败');
        }
      },
      async fetchData() {
        try {
          const response = await fetch('https://srgm.dreamplace.cn/data');
          this.data = await response.json();
        } catch (error) {
          Message.error('获取数据失败');
        }
      },
      copyToClipboard(command) {
        navigator.clipboard.writeText(command).then(() => {
          Message.success('指令已复制到剪切板');
        });
      },
    },
  };
  </script>
  
  <style>
  /* 数据展示卡片布局样式 */
  .data-display {
    margin-top: 20px;
  }
  
  .data-card {
    width: 100%;
    height: 100%;
  }
  </style>
  