<script setup lang="ts">
import { computed, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { Heart } from '@lucide/vue'
import { useAuthStore } from '../stores/auth'
import { hasSupabaseConfig } from '../services/supabase'

const email = ref('')
const password = ref('')
const loading = ref(false)
const auth = useAuthStore()
const route = useRoute()
const router = useRouter()
const canSubmit = computed(() => email.value.includes('@') && password.value.length >= 6)

async function submit() {
  if (!canSubmit.value) return
  loading.value = true
  try {
    await auth.login(email.value, password.value)
    await router.replace(String(route.query.redirect || '/'))
  } finally { loading.value = false }
}
</script>
<template>
  <main class="login-page">
    <section class="login-card">
      <div class="login-mark"><Heart :size="30" /></div>
      <h1>Nhật ký của chúng mình</h1>
      <p>Không gian riêng để giữ lại kỷ niệm, nhật ký và vị trí của hai người.</p>
      <div v-if="!hasSupabaseConfig" class="config-warning">Hãy điền Supabase URL và anon key trong file .env trước khi đăng nhập.</div>
      <form @submit.prevent="submit">
        <label>Email<input v-model.trim="email" type="email" autocomplete="email" required /></label>
        <label>Mật khẩu<input v-model="password" type="password" autocomplete="current-password" minlength="6" required /></label>
        <label class="check"><input type="checkbox" checked /> Ghi nhớ phiên đăng nhập</label>
        <button class="primary-btn" type="submit" :disabled="!canSubmit || loading">{{ loading ? 'Đang đăng nhập...' : 'Đăng nhập' }}</button>
        <p v-if="auth.error" class="form-error">{{ auth.error }}</p>
      </form>
    </section>
  </main>
</template>
