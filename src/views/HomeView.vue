<template>
  <div class="common-layout">
    <el-container>
      <el-container>
        <HomeAside :collapsed="collapsed" @toggle="toggleSidebar" />
        <HomeMain />
      </el-container>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import HomeAside from '@/components/HomeAside.vue'
import HomeMain from '@/components/HomeMain.vue'

const collapsed = ref(false)

const toggleSidebar = () => {
  collapsed.value = !collapsed.value
}
</script>

<style scoped lang="scss">
.common-layout {
  height: 100vh;
}
</style>
