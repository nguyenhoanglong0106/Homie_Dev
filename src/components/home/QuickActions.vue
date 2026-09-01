<script setup lang="ts">
import { BookOpen, Camera, Map, PlusCircle } from '@lucide/vue'
import { googlePhotosAlbumUrl } from '../../services/supabase'

const openAlbum = () => window.open(googlePhotosAlbumUrl, '_blank', 'noopener,noreferrer')
</script>
<template>
  <section class="quick-grid">
    <RouterLink to="/diary" class="quick-action"><BookOpen :size="20" /> Viết nhật ký</RouterLink>
    <RouterLink to="/map" class="quick-action"><Map :size="20" /> Xem bản đồ</RouterLink>
    <button class="quick-action" type="button" @click="openAlbum"><Camera :size="20" /> Album ảnh</button>
    <RouterLink to="/timeline" class="quick-action"><PlusCircle :size="20" /> Thêm kỷ niệm</RouterLink>
  </section>
</template>
