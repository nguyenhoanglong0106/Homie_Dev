<script setup lang="ts">
import { MapPinned } from '@lucide/vue'
defineProps<{ label: string }>()
</script>
<template>
  <RouterLink to="/map" class="soft-card compact"><MapPinned :size="18" /><div><span>Khoảng cách hiện tại</span><strong>{{ label }}</strong></div></RouterLink>
</template>
