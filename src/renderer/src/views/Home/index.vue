<template>
  <h3>Home Page</h3>
  <IUpload v-model="fileList" />
</template>
<script setup lang="ts">
import { ref } from 'vue'
import IUpload from '../../components/Upload/index.vue'

const fileList = ref([])
</script>
