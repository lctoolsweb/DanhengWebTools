<template>
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
</template>


<script setup>
import { ref, watch, onMounted, computed } from 'vue';
import axios from 'axios';
import { Message } from '@arco-design/web-vue';
import MainMission from './json/MainMission.json';
import SubMission from './json/SubMission.json';

const showNotice = ref(true);
const noticeContent = 'cxfm666及其他任何衍生工具都是免费软件，如果你是付费购买的，那你就被骗了，请及时退款并举报。';

const selectedType = ref(null);
const selectedMainMission = ref(null);
const selectedSubMission = ref(null);

const typeOptions = ref([]);
const mainMissionOptions = ref([]);
const subMissionOptions = ref([]);

onMounted(async () => {
  
  setTimeout(() => {
    showNotice.value = true;
  }, 1000);

  
  const address = localStorage.getItem('address');
  const adminpass = localStorage.getItem('adminpass');
  const uid = localStorage.getItem('uid');

  if (!address || !adminpass || !uid) {
    Message.info('页面正在建设中，敬请期待Ψ(￣∀￣)Ψ');
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
});


const completeMainMission = async () => {
  const address = localStorage.getItem('address');
  const adminpass = localStorage.getItem('adminpass');
  const uid = localStorage.getItem('uid');

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
  const uid = localStorage.getItem('uid');

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
  flex-direction: column;
  align-items: flex-start;
  color: #000;
  margin: 18px 0;
  margin-bottom: 20px; 

  > div {
    margin-bottom: 10px;

    &:nth-child(1) {
      color: #6b6a6a !important;
    }
  }
}


.select-container {
  width: 100%;
}

.button-group {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
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

  .button-group {
    flex-direction: column;
    align-items: center;
    margin-top: 20px;
  }

  .button-group button {
    width: 100%;
    margin-top: 10px;
  }
}
</style>

