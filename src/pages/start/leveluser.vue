<script setup lang="ts">
import Header from '@/components/Header/index.vue'
import startnav from './components/startnav.vue'
import leveluser from './components/leveluser.vue'


</script>
<template>
  <Header />
  <div class="cont">
    <startnav />
    <div class="selectcom">
      <leveluser />
    </div>
  </div>

</template>

<style lang="less" scoped>
@import url('./style.less');
</style>
