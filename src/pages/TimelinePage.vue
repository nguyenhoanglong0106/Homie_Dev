<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { Heart, Pencil, Plus, Search, Trash2 } from '@lucide/vue'
import { useAuthStore } from '../stores/auth'
import { useCoupleStore } from '../stores/couple'
import { useMemoriesStore } from '../stores/memories'
import { useToast } from '../composables/useToast'
import { todayIso, formatDate } from '../utils/date'
import type { Memory } from '../types'

const auth = useAuthStore(); const couple = useCoupleStore(); const store = useMemoriesStore(); const toast = useToast()
const search = ref(''); const year = ref(''); const editingId = ref<string | null>(null)
const form = reactive({ title: '', content: '', memory_date: todayIso(), cover_image_url: '', google_photos_url: '', location_name: '', latitude: '', longitude: '', is_favorite: false })
const years = computed(() => [...new Set(store.items.map((item) => item.memory_date.slice(0, 4)))])
const grouped = computed(() => years.value.map((itemYear) => ({ year: itemYear, items: store.items.filter((item) => item.memory_date.startsWith(itemYear)) })))

function reset() { editingId.value = null; Object.assign(form, { title: '', content: '', memory_date: todayIso(), cover_image_url: '', google_photos_url: '', location_name: '', latitude: '', longitude: '', is_favorite: false }) }
function edit(item: Memory) { editingId.value = item.id; Object.assign(form, { ...item, cover_image_url: item.cover_image_url || '', google_photos_url: item.google_photos_url || '', location_name: item.location_name || '', latitude: item.latitude?.toString() || '', longitude: item.longitude?.toString() || '' }) }
async function reload() { if (couple.couple?.id) await store.load(couple.couple.id, search.value, year.value) }
async function submit() {
  if (!couple.couple?.id || !auth.user) return
  const input = { couple_id: couple.couple.id, created_by: auth.user.id, title: form.title, content: form.content, memory_date: form.memory_date, cover_image_url: form.cover_image_url || null, google_photos_url: form.google_photos_url || null, location_name: form.location_name || null, latitude: form.latitude ? Number(form.latitude) : null, longitude: form.longitude ? Number(form.longitude) : null, is_favorite: form.is_favorite }
  if (editingId.value) await store.update(editingId.value, input); else await store.create(input)
  toast.push('Đã lưu kỷ niệm', 'success'); reset(); await reload()
}
async function remove(id: string) { await store.remove(id); toast.push('Đã xóa kỷ niệm', 'success') }
onMounted(async () => { if (!couple.couple) await couple.load(); await reload() })
</script>
<template>
  <section class="page-stack">
    <header class="page-header"><div><span>Timeline</span><h1>Kỷ niệm của chúng mình</h1></div></header>
    <form class="editor-card" @submit.prevent="submit">
      <div class="form-row"><label>Tiêu đề<input v-model="form.title" required maxlength="120" /></label><label>Ngày<input v-model="form.memory_date" type="date" required /></label></div>
      <label>Nội dung<textarea v-model="form.content" rows="4" required /></label>
      <div class="form-row"><label>Ảnh bìa URL<input v-model="form.cover_image_url" type="url" /></label><label>Google Photos URL<input v-model="form.google_photos_url" type="url" /></label></div>
      <div class="form-row"><label>Địa điểm<input v-model="form.location_name" /></label><label>Lat<input v-model="form.latitude" inputmode="decimal" /></label><label>Lng<input v-model="form.longitude" inputmode="decimal" /></label></div>
      <label class="check"><input v-model="form.is_favorite" type="checkbox" /> Đánh dấu yêu thích</label>
      <div class="actions"><button class="primary-btn" type="submit"><Plus :size="18" /> {{ editingId ? 'Cập nhật' : 'Thêm kỷ niệm' }}</button><button type="button" class="ghost-btn" @click="reset">Làm mới</button></div>
    </form>
    <div class="toolbar"><Search :size="18" /><input v-model="search" placeholder="Tìm tiêu đề hoặc nội dung" @input="reload" /><select v-model="year" @change="reload"><option value="">Tất cả năm</option><option v-for="item in years" :key="item" :value="item">{{ item }}</option></select></div>
    <p v-if="store.loading" class="soft-card">Đang tải timeline...</p><p v-else-if="!store.items.length" class="soft-card empty">Chưa có kỷ niệm nào. Hãy lưu lại khoảnh khắc đầu tiên.</p>
    <section v-for="group in grouped" :key="group.year" class="timeline-year"><h2>{{ group.year }}</h2><article v-for="item in group.items" :key="item.id" class="timeline-item"><time>{{ formatDate(item.memory_date) }}</time><RouterLink :to="`/memories/${item.id}`"><strong>{{ item.title }}</strong><p>{{ item.content }}</p></RouterLink><div><button aria-label="Sửa kỷ niệm" @click="edit(item)"><Pencil :size="17" /></button><button aria-label="Yêu thích" @click="store.update(item.id, { is_favorite: !item.is_favorite })"><Heart :fill="item.is_favorite ? 'currentColor' : 'none'" :size="17" /></button><button aria-label="Xóa kỷ niệm" @click="remove(item.id)"><Trash2 :size="17" /></button></div></article></section>
  </section>
</template>
