<template>
  <el-main class="content">
    <el-card class="card">
      <keep-alive>
        <router-view v-if="$route.meta.keepAlive"></router-view>
      </keep-alive>
      <router-view v-if="!$route.meta.keepAlive"></router-view>
    </el-card>
  </el-main>
</template>

<script>
import Tab from "@/views/home/Tab.vue";
export default {
//   name: "Content",
  components: {
    Tab,
  },
};
</script>

<style lang="less" scoped>
.content {
  width: 100%;
  height: 100%;
  // background-color: #fff; // #acb0b6
  background-color: #f3f6fb;
}
.card {
  background-color: #fff; // #acb0b6
  box-shadow: none;
  padding: 0;
  border: 0;
  width: 100%;
  height: 100%;
  & /deep/ .el-card__body {
    height: 100%;
  }
}
</style>