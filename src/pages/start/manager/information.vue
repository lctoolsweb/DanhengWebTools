<template>
  <div>
    <a-card bordered>
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
  </div>
</template>

<script>
import axios from 'axios';
import { Message } from '@arco-design/web-vue';
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';

export default {
  name: 'ServerStatus',
  setup() {
    const serverInfo = ref({});
    const intervalId = ref(null);
    const countdownPercent = ref(1);
    const countdownIntervalId = ref(null);

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
        const dispatchServer = import.meta.env.VITE_DANHENG_DISPATCH_SERVER;
        const adminKey = import.meta.env.VITE_DANHENG_ADMIN_KEY;

        const authResponse = await axios.post(`${dispatchServer}/muip/auth_admin`, {
          admin_key: adminKey,
          key_type: 'PEM'
        });

        if (authResponse.data.code !== 0) {
          Message.error('授权失败');
          return;
        }

        const sessionId = authResponse.data.data.sessionId;

        const serverResponse = await axios.get(`${dispatchServer}/muip/server_information`, {
          params: { SessionId: sessionId }
        });

        if (serverResponse.data.code === 0) {
          serverInfo.value = serverResponse.data.data;
        } else {
          Message.error('获取服务器信息失败');
        }
      } catch (error) {
        Message.error('请求失败');
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
      countdownPercent
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
