<template>
  <d2-container>
    <h1>帮助文档</h1>
    <document-zone mount-zone="WORKORDER_APPLICATION_SCALE_REPLICAS"></document-zone>
  </d2-container>
</template>

<script>

import DocumentZone from '@/components/opscloud/sys/DocumentZone.vue'

export default {
  components: {
    DocumentZone
  }
}
</script>
