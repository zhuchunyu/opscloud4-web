<template>
  <div style="margin-bottom: 10px">
    <!--    版本1-->
    <el-tag  v-if="type === 'version1' && version.versionName !== '-'">
      <b style="font-size: 15px">{{ version.versionName }}</b>
    </el-tag>
    <!--    版本2-->
    <el-tag v-if="type === 'version2'">
      <b style="font-size: 15px; margin-right: 15px" v-if="version.versionName !== '-'">{{ version.versionName }}</b>
      <span style="font-size: 12px">副本数 {{ version.pods.length }}/{{ replicas }}</span>
    </el-tag>
    <span style="margin-left: 15px; font-size: 12px"><i class="fab fa-docker"></i>{{ version.image }}</span>
  </div>
</template>

<script>
export default {
  name: 'DeployVersion',
  props: ['version', 'type', 'replicas']
}
</script>

<style scoped>

</style>
