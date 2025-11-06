<script lang="ts" setup>
import { onMounted, ref } from 'vue';
import Compress from './Compress/index.vue'

const route = ref('compress') // 默认显示compress
const enterAction = ref({})

onMounted(() => {
  window.utools?.onPluginEnter((action) => {
    route.value = action.code
    enterAction.value = action
  })
  window.utools?.onPluginOut((isKill) => {
    route.value = ''
  })
})
</script>

<template>
  <Compress :enterAction="enterAction"></Compress>
</template>
