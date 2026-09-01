<script setup lang="ts">
import { computed, onMounted, reactive } from 'vue'
import { Heart } from '@lucide/vue'
import { useCoupleStore } from '../stores/couple'
import { daysBetween, formatDate } from '../utils/date'
const couple = useCoupleStore()
const form = reactive({ display_name: '', nickname: '', avatar_url: '', birthday: '' })
const days = computed(() => daysBetween(couple.couple?.started_date))
onMounted(async () => { if (!couple.couple) await couple.load(); Object.assign(form, { display_name: couple.myProfile?.display_name || '', nickname: couple.myProfile?.nickname || '', avatar_url: couple.myProfile?.avatar_url || '', birthday: couple.myProfile?.birthday || '' }) })
async function save() { if (couple.myProfile) await couple.saveProfile({ id: couple.myProfile.id, display_name: form.display_name, nickname: form.nickname || null, avatar_url: form.avatar_url || null, birthday: form.birthday || null }) }
</script>
<template><section class="page-stack"><header class="page-header"><div><span>Hồ sơ</span><h1>Chúng mình</h1></div></header><section class="couple-profile"><div v-for="profile in couple.profiles" :key="profile.id"><img :src="profile.avatar_url || '/favicon.svg'" :alt="profile.display_name" /><strong>{{ profile.nickname || profile.display_name }}</strong></div><Heart :size="28" fill="currentColor" /></section><section class="soft-card"><span>Bắt đầu</span><strong>{{ couple.couple ? formatDate(couple.couple.started_date) : 'Chưa cấu hình' }}</strong><p>Đã bên nhau {{ days }} ngày</p></section><form class="editor-card" @submit.prevent="save"><h2>Hồ sơ của tôi</h2><label>Tên hiển thị<input v-model="form.display_name" required /></label><label>Nickname<input v-model="form.nickname" /></label><label>Avatar URL<input v-model="form.avatar_url" type="url" /></label><label>Sinh nhật<input v-model="form.birthday" type="date" /></label><button class="primary-btn" type="submit">Lưu hồ sơ</button></form></section></template>
