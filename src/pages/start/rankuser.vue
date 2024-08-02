<script setup lang="ts">
import Header from '@/components/Header/index.vue'
import startnav from './components/startnav.vue'
import rankuser from './components/rankuser.vue'


</script>
<template>
  <Header />
  <div class="cont">
    <startnav />
    <div class="selectcom">
      <rankuser />
    </div>
  </div>

</template>

<style lang="less" scoped>
@import url('./style.less');
</style>
