<template>
  <el-aside :width="asideWidth" :class="['home-aside', { 'is-collapsed': collapsed }]">
    <div class="side-header">
      <button class="logo-button" type="button" @click="emit('toggle')">
        <el-icon size="20">
          <ElementPlus />
        </el-icon>
      </button>
      <button v-if="!collapsed" class="toggle-button" type="button" @click="emit('toggle')">
        <el-icon size="20">
          <Fold />
        </el-icon>
      </button>
    </div>
    <div v-if="!collapsed" class="side-main"></div>
    <div v-if="!collapsed" class="side-footer"></div>
  </el-aside>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { ElementPlus, Fold } from '@element-plus/icons-vue'

const props = defineProps<{
  collapsed: boolean
}>()

const emit = defineEmits<{
  toggle: []
}>()

const asideWidth = computed(() => (props.collapsed ? '64px' : '240px'))
</script>

<style scoped lang="scss">
.home-aside {
  height: 100vh;
  background: #f7f7f9;
  border-right: 1px solid #e8e8ec;
  transition:
    width 0.2s ease,
    padding 0.2s ease;
  overflow: hidden;

  .side-header {
    box-sizing: border-box;
    height: 52px;
    width: 100%;
    padding: 0 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
}

.is-collapsed {
  .side-header {
    justify-content: center;
    padding: 0;
  }
}

.logo-button,
.toggle-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  border: 0;
  background: transparent;
  color: #303133;
  cursor: pointer;
}

.logo-button {
  width: 36px;
  height: 36px;
}

.toggle-button {
  width: 32px;
  height: 32px;
  border-radius: 6px;
}

.logo-button:hover,
.toggle-button:hover {
  background: #ececf1;
}
</style>
