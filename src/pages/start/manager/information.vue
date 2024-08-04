<template>
  <div>
    <!-- 服务器状态页面 -->
    <a-card v-if="!hasError" bordered>
      <template #title>
        <div class="card-title">
          <span>服务器状态</span>
          <a-progress
            size="mini"
            status="warning"
            :percent="countdownPercent"
            style="margin-left: 20px; width: 100px;"
          />
        </div>
      </template>
      <a-row :gutter="16">
        <a-col :span="12">
          <div>
            <a-progress
              type="circle"
              :percent="cpuUsagePercent"
              :style="{ width: '100px' }"
              :color="{
                '0%': 'rgb(var(--primary-6))',
                '100%': 'rgb(var(--success-6))',
              }"
              :format="percent => `${(percent * 100).toFixed(0)}%`"
            />
            <p><strong>CPU 使用率：</strong>{{ (cpuUsagePercent * 100).toFixed(0) }}%</p>
            <p><strong>CPU 型号：</strong>{{ serverInfo.cpuModel }}</p>
            <p><strong>系统版本：</strong>{{ serverInfo.systemVersion }}</p>
          </div>
        </a-col>

        <a-col :span="12">
          <div>
            <a-progress
              type="circle"
              :percent="memoryUsagePercent"
              :style="{ width: '100px' }"
              :color="{
                '0%': 'rgb(var(--primary-6))',
                '100%': 'rgb(var(--success-6))',
              }"
              :format="percent => `${(percent * 100).toFixed(0)}%`"
            />
            <p><strong>内存使用率：</strong>{{ (memoryUsagePercent * 100).toFixed(0) }}%</p>
            <p><strong>最大内存：</strong>{{ serverInfo.maxMemory }} MB</p>
            <p><strong>已用内存：</strong>{{ serverInfo.usedMemory }} MB</p>
            <p><strong>程序使用内存：</strong>{{ serverInfo.programUsedMemory }} MB</p>
          </div>
        </a-col>
      </a-row>
    </a-card>

    <!-- 错误页面 -->
    <a-result v-if="hasError" status="error" title="获取服务器状态失败">
      <template #icon>
        <IconFaceFrownFill />
      </template>
      <template #subtitle>{{ errorMessage }}</template>

      <template #extra>
        <a-button type="primary" @click="refresh">Refresh</a-button>
      </template>
      <a-typography style="background: var(--color-fill-2); padding: 24px;">
        <a-typography-paragraph>Try:</a-typography-paragraph>
        <ul>
          <li>检查您的网络</li>
          <li>将上方报错信息反馈给服务器管理员</li>
        </ul>
      </a-typography>
    </a-result>
  </div>
</template>

<script>
import axios from 'axios';
import { Message } from '@arco-design/web-vue';
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';
import { IconFaceFrownFill } from '@arco-design/web-vue/es/icon';

export default {
  name: 'ServerStatus',
  components: {
    IconFaceFrownFill,
  },
  setup() {
    const serverInfo = ref({});
    const intervalId = ref(null);
    const countdownPercent = ref(1);
    const countdownIntervalId = ref(null);
    const hasError = ref(false);
    const errorMessage = ref('');

    const cpuUsagePercent = computed(() => {
      return serverInfo.value.cpuUsage ? Number((serverInfo.value.cpuUsage / 100).toFixed(2)) : 0;
    });

    const memoryUsagePercent = computed(() => {
      return serverInfo.value.maxMemory && serverInfo.value.usedMemory
        ? Number((serverInfo.value.usedMemory / serverInfo.value.maxMemory).toFixed(2))
        : 0;
    });

    const fetchServerInfo = async () => {
      try {
        const apiServer = import.meta.env.VITE_DHWT_API_SERVER;
        const serverResponse = await axios.get(`${apiServer}/api/status`);

        if (serverResponse.data.code === 0) {
          serverInfo.value = serverResponse.data.data;
          hasError.value = false;
        } else {
          throw new Error('获取服务器信息失败');
        }
      } catch (error) {
        hasError.value = true;
        errorMessage.value = error.toJSON ? JSON.stringify(error.toJSON()) : error.message;
        Message.error(error.message || '请求失败');
      }
    };

    const startCountdown = () => {
      let countdown = 10;
      countdownPercent.value = 1;
      countdownIntervalId.value = setInterval(() => {
        countdown -= 1;
        countdownPercent.value = countdown / 10;
        if (countdown <= 0) {
          countdown = 10;
        }
      }, 1000);
    };

    const refresh = () => {
      hasError.value = false;
      errorMessage.value = '';
      fetchServerInfo();
    };

    onMounted(() => {
      fetchServerInfo();
      intervalId.value = setInterval(fetchServerInfo, 10000);
      startCountdown();
    });

    onBeforeUnmount(() => {
      clearInterval(intervalId.value);
      clearInterval(countdownIntervalId.value);
    });

    return {
      serverInfo,
      cpuUsagePercent,
      memoryUsagePercent,
      countdownPercent,
      hasError,
      errorMessage,
      refresh
    };
  }
};
</script>

<style scoped>
.card-title {
  display: flex;
  align-items: center;
}
</style>
