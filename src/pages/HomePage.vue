<script setup lang="ts">
import { computed, onMounted } from 'vue'
import LoveCounter from '../components/home/LoveCounter.vue'
import UpcomingDateCard from '../components/home/UpcomingDateCard.vue'
import FavoriteMemoryCard from '../components/home/FavoriteMemoryCard.vue'
import CurrentDistanceCard from '../components/home/CurrentDistanceCard.vue'
import QuickActions from '../components/home/QuickActions.vue'
import { useCoupleStore } from '../stores/couple'
import { useMemoriesStore } from '../stores/memories'
import { useDiariesStore } from '../stores/diaries'
import { useSpecialDatesStore } from '../stores/specialDates'
import { useLocationStore } from '../stores/location'
import { countdownLabel, daysBetween, formatDate, greeting } from '../utils/date'
import { formatDistance } from '../utils/distance'

const couple = useCoupleStore()
const memories = useMemoriesStore()
const diaries = useDiariesStore()
const dates = useSpecialDatesStore()
const locations = useLocationStore()
const names = computed(() => couple.profiles.map((profile) => profile.nickname || profile.display_name).join(' & ') || 'Hai chúng ta')
const favorite = computed(() => memories.items.find((item) => item.is_favorite) ?? memories.items[0] ?? null)
const upcoming = computed(() => dates.items.map((item) => ({ item, label: countdownLabel(item.event_date, item.repeat_yearly) }))[0])
const latestDiary = computed(() => diaries.items[0])
const latestMemory = computed(() => memories.items[0])

onMounted(async () => {
  if (!couple.couple) await couple.load()
  if (couple.couple?.id) {
    await Promise.all([memories.load(couple.couple.id), diaries.load(couple.couple.id), dates.load(couple.couple.id), locations.load(couple.couple.id)])
  }
})
</script>
<template>
  <section class="page-stack">
    <header class="home-hero">
      <div class="avatar-pair"><img v-for="profile in couple.profiles" :key="profile.id" :src="profile.avatar_url || '/favicon.svg'" :alt="profile.display_name" /></div>
      <p>{{ greeting() }}</p><h1>{{ names }}</h1>
    </header>
    <LoveCounter :days="daysBetween(couple.couple?.started_date)" :names="couple.couple?.name || 'Chúng mình'" />
    <QuickActions />
    <div class="grid-two">
      <UpcomingDateCard v-if="upcoming" :title="upcoming.item.title" :countdown="upcoming.label" />
      <CurrentDistanceCard :label="formatDistance(locations.distanceMeters)" />
    </div>
    <FavoriteMemoryCard :memory="favorite" />
    <section class="soft-card"><span>Gần đây</span><strong>{{ latestMemory?.title || 'Chưa có kỷ niệm nào' }}</strong><p>{{ latestMemory ? formatDate(latestMemory.memory_date) : 'Hãy lưu lại khoảnh khắc đầu tiên.' }}</p></section>
    <section class="soft-card"><span>Nhật ký mới nhất</span><strong>{{ latestDiary?.title || 'Chưa có nhật ký' }}</strong><p>{{ latestDiary ? formatDate(latestDiary.diary_date) : 'Một dòng nhỏ cũng đủ làm ngày này đẹp hơn.' }}</p></section>
  </section>
</template>
