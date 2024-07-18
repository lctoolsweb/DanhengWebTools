<template>
  <div class="full-screen">
    <a-page-header
      :style="{ width: '100vw', background: theme === 'dark' ? 'black' : 'white' }"
      title="Ciallo～(∠・ω< )⌒☆Tools"
    >
      <template #extra>
        <div class="extra-container">
          <span :class="latencyClass">{{ latency }} ms</span>

          <a href="https://github.com/lctoolsweb/LunarCoreTools/" target="_blank" rel="noopener noreferrer" class="github-link">
            <img src="@/components/icon/BiGithub.svg" alt="GitHub Icon" :class="{ 'dark-icon': theme === 'dark' }" class="icon" />
          </a>

          <a href="http://qm.qq.com/cgi-bin/qm/qr?_wv=1027&k=J54m8XB3Ig8VJ41ICO8KW029aSz4fFB-&authKey=tq9%2BSsg2M30Jy1v9OwJEa%2B%2FmarnH2AslQYQsv24BkkeqY39%2FtlpZsrIkqO01SScg&noverify=0&group_code=124750952" target="_blank" rel="noopener noreferrer" class="qq-link">
            <img src="@/components/icon/BiTencentQq.svg" alt="Tencent QQ Icon" :class="{ 'dark-icon': theme === 'dark' }" class="icon" />
          </a>

          <button @click="toggleTheme" class="theme-toggle">
            <img v-if="theme === 'light'" src="@/components/icon/IonMdMoon.svg" :class="{ 'dark-icon': theme === 'dark' }" alt="Moon Icon" class="icon" />
            <img v-else src="@/components/icon/IonMdSunny.svg" :class="{ 'dark-icon': theme === 'dark' }" alt="Sunny Icon" class="icon" />
          </button>
        </div>
      </template>
    </a-page-header>
  </div>
</template>



<script>
export default {
  data() {
    return {
      theme: 'light',
      latency: 0, // 初始化延迟为 0
      initialFetch: true // 标志是否是第一次获取延迟值
    };
  },
  computed: {
    latencyClass() {
      if (this.latency < 100) {
        return 'latency-green';
      } else if (this.latency < 200) {
        return 'latency-orange';
      } else {
        return 'latency-red';
      }
    }
  },
  mounted() {
    // 在组件加载完成后获取延迟
    this.fetchLatency();
    setTimeout(() => {
      this.fetchLatency();
      setInterval(this.fetchLatency, 10000); // 每10秒钟获取一次延迟
    }, 1000); // 第一次获取之后的1秒获取第二次
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light';
      if (this.theme === 'dark') {
        document.body.setAttribute('arco-theme', 'dark');
      } else {
        document.body.removeAttribute('arco-theme');
      }
    },
    fetchLatency() {
      const start = Date.now();
      fetch(`${import.meta.env.VITE_DANHENG_DISPATCH_SERVER}/muip/auth_admin`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          admin_key: import.meta.env.VITE_DANHENG_ADMIN_KEY,
          key_type: 'pem'
        })
      })
      .then(response => response.json())
      .then(() => {
        const end = Date.now();
        const latency = end - start; // 计算延迟时间
        this.latency = latency; // 更新延迟
      })
      .catch(error => {
        console.error('Error fetching latency:', error);
      });
    }
  }
};

</script>

<style>
.full-screen {
  width: 100vw;
  overflow-x: hidden;
}

.a-page-header {
  background: white; 
}
.extra-container {
  display: flex;
  align-items: center; 
}
.github-link,
.qq-link {
  text-decoration: none;
  color: inherit;
}
.latency {
  margin-right: 10px;
  font-weight: bold;
}
button {
  background: none;
  border: none;
  cursor: pointer;
}
.icon {
  width: 24px;
  height: 24px;
}
.dark-icon {
  filter: invert(1);
}
.latency-green {
  color: green;
}
.latency-orange {
  color: orange;
}
.latency-red {
  color: red;
}

</style>
