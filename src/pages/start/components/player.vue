<template>
  <a-card class="wider-card content-container">
    <template v-slot:title>
      玩家管理
      <span style="float: right;">在线人数: {{ onlineCount }}</span>
    </template>
    <div style="margin-bottom: 20px;">
      <a-input 
        v-model="searchUID" 
        placeholder="输入UID搜索" 
        style="width: 300px; margin-right: 10px;" 
      />
      <a-button type="primary" @click="handleSearch">搜索</a-button>
    </div>
    <div class="table-container">
      <a-table 
        :columns="columns" 
        :data="filteredData" 
        row-key="uid" 
        :row-class-name="rowClassName" 
        @rowClick="onRowClick"
      />
    </div>
    <a-modal v-model:visible="isModalVisible" title="任务信息" :footer="null">
      <div>
        <div class="scrolling-notice" v-if="showNotice">
          <marquee behavior="scroll" direction="left">{{ noticeContent }}</marquee>
        </div>

        <div class="commuse">
          <div class="commuse-item">
            <div class="text-slate-900 dark:text-slate-100">任务类型:</div>
            <div class="select-container">
              <a-select v-model="selectedType" placeholder="请选择类型">
                <a-option v-for="(type, index) in typeOptions" :key="index" :value="type">
                  {{ type }}
                </a-option>
              </a-select>
            </div>
          </div>

          <div class="commuse-item">
            <div class="text-slate-900 dark:text-slate-100">主任务:</div>
            <div class="select-container">
              <a-select v-model="selectedMainMission" placeholder="请选择主任务" :disabled="!selectedType">
                <a-option v-for="(mission, index) in mainMissionOptions" :key="index" :value="mission.value">
                  {{ mission.label }}
                </a-option>
              </a-select>
            </div>
          </div>

          <div class="commuse-item">
            <div class="text-slate-900 dark:text-slate-100">子任务:</div>
            <div class="select-container">
              <a-select v-model="selectedSubMission" placeholder="请选择子任务" :disabled="!selectedMainMission">
                <a-option v-for="(subMission, index) in subMissionOptions" :key="index" :value="subMission.value">
                  {{ subMission.label }}
                </a-option>
              </a-select>
            </div>
          </div>

          <div class="button-group">
            <a-button type="primary" shape="round" size="large" @click="completeMainMission">完成主任务</a-button>
            <a-button type="primary" shape="round" size="large" @click="completeSubMission">完成子任务</a-button>
          </div>
        </div>
      </div>
    </a-modal>

    <!-- 遮罩弹窗 -->
    <div v-if="isMaskVisible" class="mask">
      <div class="mask-content">
        監修中...
      </div>
    </div>
  </a-card>
</template>

<script>
import { reactive, ref, onMounted, computed, watch } from 'vue';
import axios from 'axios';
import { Message } from '@arco-design/web-vue';
import MainMission from './json/MainMission.json';
import SubMission from './json/SubMission.json';

export default {
  setup() {
    const columns = [
      { title: 'Head Icon ID', dataIndex: 'headIcon', key: 'headIcon' },
      { title: 'UID', dataIndex: 'uid', key: 'uid' },
      { title: 'Name', dataIndex: 'name', key: 'name' },
      { title: 'Level', dataIndex: 'level', key: 'level' }
    ];

    const data = reactive([]);
    const searchUID = ref('');
    const searchQuery = ref('');
    const isModalVisible = ref(false);
    const isMaskVisible = ref(true);
    const selectedUID = ref('');

    const fetchStatus = async () => {
      try {
        const response = await axios.get('https://lctweb.ates.top/api/status');
        const players = response.data.onlinePlayers.map(player => ({
          ...player,
          uid: String(player.uid), 
          headIcon: player.headIcon,
        }));
        data.push(...players);
      } catch (error) {
        console.error('Error fetching status:', error);
      }
    };

    const filteredData = computed(() => {
      if (!searchQuery.value) return data;
      return data.filter(player => player.uid.includes(searchQuery.value));
    });

    const onlineCount = computed(() => {
      return filteredData.value.length;
    });

    const handleSearch = () => {
      searchQuery.value = searchUID.value;
    };

    const rowClassName = (record) => {
      return record.uid === searchQuery.value ? 'highlight-row' : '';
    };

    const onRowClick = (record) => {
      selectedUID.value = record.uid;
      localStorage.setItem('uid', record.uid); // 保存 UID 到 localStorage
      isModalVisible.value = true; // 显示弹出容器
      fetchMissionData(record.uid);
    };

    onMounted(fetchStatus);

    const showNotice = ref(true);
    const noticeContent = '';
    const selectedType = ref(null);
    const selectedMainMission = ref(null);
    const selectedSubMission = ref(null);
    const typeOptions = ref([]);
    const mainMissionOptions = ref([]);
    const subMissionOptions = ref([]);

    const fetchMissionData = async (uid) => {
      const address = 'https://lctweb.ates.top/api/command';
      const adminpass = 'mhfude';


      if (!address || !adminpass || !uid) {
        Message.info('用户未登录，请重试');
        return;
      }

      try {
        const response = await axios.post(address, new URLSearchParams({
          command: 'mission running',
          adminpass,
          uid
        }), {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        });

        const responseData = response.data;
        if (responseData.retcode !== 0) {
          Message.error('获取任务信息失败');
          return;
        }

        const missionInfo = responseData.data.missioninfo;

        const categorizedMissions = {};

        missionInfo.forEach(mission => {
          const mainMissionId = mission.mainMission.toString();
          const typename = MainMission[mainMissionId]?.typename || '未知类型';

          if (!categorizedMissions[typename]) {
            categorizedMissions[typename] = [];
          }

          categorizedMissions[typename].push({
            value: mainMissionId,
            label: MainMission[mainMissionId]?.text || '未知任务',
            children: mission.subMissions.map(subMission => ({
              value: subMission.toString(),
              label: SubMission[subMission.toString()] || '未知子任务'
            }))
          });
        });

        typeOptions.value = Object.keys(categorizedMissions);

        watch(selectedType, (newType) => {
          mainMissionOptions.value = categorizedMissions[newType] || [];
          selectedMainMission.value = null;
          selectedSubMission.value = null;
        });

        watch(selectedMainMission, (newMainMission) => {
          const selectedMission = mainMissionOptions.value.find(mission => mission.value === newMainMission);
          subMissionOptions.value = selectedMission ? selectedMission.children : [];
          selectedSubMission.value = null;
        });
      } catch (error) {
        Message.error('请求失败');
        console.error(error);
      }
    };

    const completeMainMission = async () => {
      const address = localStorage.getItem('address');
      const adminpass = localStorage.getItem('adminpass');
      const uid = selectedUID.value;

      if (!address || !adminpass || !uid) {
        Message.info('用户未登录，请重试');
        return;
      }

      if (!selectedMainMission.value) {
        Message.info('请选择一个主任务');
        return;
      }

      console.log('Selected Main Mission ID:', selectedMainMission.value);

      try {
        const response = await axios.post(address, new URLSearchParams({
          command: `mission finish ${selectedMainMission.value}`,
          adminpass,
          uid
        }), {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        });

        const responseData = response.data;
        if (responseData.retcode === 0) {
          Message.success('主任务完成成功');
        } else {
          Message.error('主任务完成失败');
        }
      } catch (error) {
        Message.error('请求失败');
        console.error(error);
      }
    };

    const completeSubMission = async () => {
      const address = localStorage.getItem('address');
      const adminpass = localStorage.getItem('adminpass');
      const uid = selectedUID.value;

      if (!address || !adminpass || !uid) {
        Message.info('用户未登录，请重试');
        return;
      }

      if (!selectedSubMission.value) {
        Message.info('请选择一个子任务');
        return;
      }

      console.log('Selected Sub Mission ID:', selectedSubMission.value);

      try {
        const response = await axios.post(address, new URLSearchParams({
          command: `mission finish ${selectedSubMission.value}`,
          adminpass,
          uid
        }), {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        });

        const responseData = response.data;
        if (responseData.retcode === 0) {
          Message.success('子任务完成成功');
        } else {
          Message.error('子任务完成失败');
        }
      } catch (error) {
        Message.error('请求失败');
        console.error(error);
      }
    };

    return {
      columns,
      data,
      searchUID,
      searchQuery,
      handleSearch,
      filteredData,
      onlineCount,
      isModalVisible,
      isMaskVisible,
      rowClassName,
      onRowClick,
      showNotice,
      noticeContent,
      selectedType,
      selectedMainMission,
      selectedSubMission,
      typeOptions,
      mainMissionOptions,
      subMissionOptions,
      completeMainMission,
      completeSubMission
    };
  }
};
</script>

<style>
.mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.mask-content {
  color: white;
  font-size: 24px;
}

.content-container {
  position: relative;
}

.table-container {
  width: 100%;
}

.wider-card {
  width: 100%;
}

.highlight-row {
  background-color: yellow;
}
</style>
