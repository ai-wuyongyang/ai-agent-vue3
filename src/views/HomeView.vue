<template>
  <div class="common-layout">
    <el-container>
      <el-container>
        <HomeAside :collapsed="collapsed" @toggle="toggleSidebar" />
        <el-main>Main</el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import HomeAside from '@/components/HomeAside.vue'

const collapsed = ref(false)

const toggleSidebar = () => {
  collapsed.value = !collapsed.value
}
</script>

<style scoped lang="scss">
.common-layout {
  height: 100vh;
}

.el-main {
  background: #e9eef3;
  text-align: center;
  line-height: 400px;
}
</style>
