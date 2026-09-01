<script setup lang="ts">
import { ExternalLink, LogOut, Moon, SunMedium } from '@lucide/vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '../stores/auth'
import { useTheme } from '../composables/useTheme'
import { googlePhotosAlbumUrl } from '../services/supabase'
const auth = useAuthStore(); const router = useRouter(); const { mode, setTheme } = useTheme()
async function logout() { await auth.logout(); await router.replace('/login') }
const openAlbum = () => window.open(googlePhotosAlbumUrl, '_blank', 'noopener,noreferrer')
</script>
<template><section class="page-stack"><header class="page-header"><div><span>Cài đặt</span><h1>Không gian riêng tư</h1></div></header><section class="soft-card"><strong>Google Photos</strong><p>Ảnh kỷ niệm chính được quản lý trong shared album, app chỉ lưu link gắn với kỷ niệm/nhật ký.</p><button class="primary-btn" @click="openAlbum"><ExternalLink :size="18" /> Album của chúng mình</button><button class="ghost-btn" @click="openAlbum">Thêm ảnh vào album</button></section><section class="soft-card"><strong>Giao diện</strong><div class="segmented"><button :class="{ selected: mode === 'light' }" @click="setTheme('light')"><SunMedium :size="16" /> Sáng</button><button :class="{ selected: mode === 'dark' }" @click="setTheme('dark')"><Moon :size="16" /> Tối</button><button :class="{ selected: mode === 'system' }" @click="setTheme('system')">System</button></div></section><section class="soft-card"><strong>Vị trí realtime</strong><p>Vị trí realtime hoạt động tốt nhất khi ứng dụng đang mở. PWA trên iOS/Android không đảm bảo GPS nên khi trình duyệt bị đóng hoặc app bị suspend.</p></section><button class="danger-btn" @click="logout"><LogOut :size="18" /> Đăng xuất</button></section></template>
