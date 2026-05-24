<template>
  <div class="common-layout">
    <el-container>
      <el-container>
        <el-aside width="240px">Aside</el-aside>
        <el-main>Main</el-main>
      </el-container>
    </el-container>
  </div>
</template>

<style scoped lang="scss">
.common-layout {
  height: 100vh;
}

.el-header {
  background: #409eff;
  color: white;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background: #d3dce6;
  height: 100vh;
  text-align: center;
  line-height: 400px;
}

.el-main {
  background: #e9eef3;
  text-align: center;
  line-height: 400px;
}
</style>