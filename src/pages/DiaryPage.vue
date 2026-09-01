<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { Pencil, Trash2 } from '@lucide/vue'
import { useAuthStore } from '../stores/auth'
import { useCoupleStore } from '../stores/couple'
import { useDiariesStore } from '../stores/diaries'
import { useToast } from '../composables/useToast'
import { todayIso, formatDate } from '../utils/date'
import type { Diary, Mood } from '../types'

const moods: { value: Mood; label: string }[] = [{ value: 'happy', label: 'Vui' }, { value: 'love', label: 'Yêu' }, { value: 'normal', label: 'Bình yên' }, { value: 'sad', label: 'Buồn' }, { value: 'excited', label: 'Hào hứng' }]
const auth = useAuthStore(); const couple = useCoupleStore(); const store = useDiariesStore(); const toast = useToast()
const view = ref<'list' | 'calendar'>('list'); const search = ref(''); const editingId = ref<string | null>(null)
const form = reactive({ diary_date: todayIso(), title: '', content: '', mood: 'love' as Mood, weather: '', google_photos_url: '' })
const byDate = computed(() => [...store.items].sort((a, b) => a.diary_date.localeCompare(b.diary_date)))
function reset() { editingId.value = null; Object.assign(form, { diary_date: todayIso(), title: '', content: '', mood: 'love', weather: '', google_photos_url: '' }) }
function edit(item: Diary) { editingId.value = item.id; Object.assign(form, { ...item, mood: item.mood || 'love', weather: item.weather || '', google_photos_url: item.google_photos_url || '' }) }
async function reload() { if (couple.couple?.id) await store.load(couple.couple.id, search.value) }
async function submit() { if (!couple.couple?.id || !auth.user) return; const input = { couple_id: couple.couple.id, created_by: auth.user.id, diary_date: form.diary_date, title: form.title, content: form.content, mood: form.mood, weather: form.weather || null, google_photos_url: form.google_photos_url || null }; if (editingId.value) await store.update(editingId.value, input); else await store.create(input); toast.push('Đã lưu nhật ký', 'success'); reset(); await reload() }
async function remove(id: string) { await store.remove(id); toast.push('Đã xóa nhật ký', 'success') }
onMounted(async () => { if (!couple.couple) await couple.load(); await reload() })
</script>
<template>
  <section class="page-stack"><header class="page-header"><div><span>Nhật ký</span><h1>Hôm nay mình viết gì?</h1></div></header>
    <form class="editor-card" @submit.prevent="submit"><div class="form-row"><label>Tiêu đề<input v-model="form.title" required /></label><label>Ngày<input v-model="form.diary_date" type="date" required /></label></div><label>Nội dung<textarea v-model="form.content" rows="5" required /></label><div class="form-row"><label>Tâm trạng<select v-model="form.mood"><option v-for="mood in moods" :key="mood.value" :value="mood.value">{{ mood.label }}</option></select></label><label>Thời tiết<input v-model="form.weather" /></label></div><label>Google Photos URL<input v-model="form.google_photos_url" type="url" /></label><div class="actions"><button class="primary-btn" type="submit">{{ editingId ? 'Cập nhật' : 'Lưu nhật ký' }}</button><button type="button" class="ghost-btn" @click="reset">Làm mới</button></div></form>
    <div class="toolbar"><input v-model="search" placeholder="Tìm nhật ký" @input="reload" /><button :class="{ selected: view === 'list' }" @click="view = 'list'">List</button><button :class="{ selected: view === 'calendar' }" @click="view = 'calendar'">Calendar</button></div>
    <p v-if="store.loading" class="soft-card">Đang tải nhật ký...</p><p v-else-if="!store.items.length" class="soft-card empty">Chưa có nhật ký.</p>
    <section :class="view === 'calendar' ? 'calendar-list' : 'page-stack'"><article v-for="item in byDate" :key="item.id" class="soft-card diary-item"><span>{{ formatDate(item.diary_date) }} · {{ moods.find((mood) => mood.value === item.mood)?.label || 'Bình yên' }}</span><strong>{{ item.title }}</strong><p>{{ item.content }}</p><div class="actions"><button aria-label="Sửa nhật ký" @click="edit(item)"><Pencil :size="17" /></button><button aria-label="Xóa nhật ký" @click="remove(item.id)"><Trash2 :size="17" /></button></div></article></section>
  </section>
</template>
