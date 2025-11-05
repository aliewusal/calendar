<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'

const props = defineProps<{
  initDate?: string
}>()

const emits = defineEmits<{
  (e: 'getDate', value: string): void
}>()

const lang = ref<string>('ru')

const weeks = computed(() => {
  return lang.value === 'ru'
    ? ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вск']
    : ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
})

const headerDate = computed(() => {
  return new Date(year.value, month.value).toLocaleString(lang.value, {
    month: 'long',
    year: 'numeric',
  })
})

const selectedDate = ref<Date>(new Date())

const year = ref<number>(new Date().getFullYear())
const month = ref<number>(new Date().getMonth())

const days = computed<Array<{ date: Date | undefined; index: number }>>(() => {
  const ds = []
  const firstMonthDate = new Date(year.value, month.value, 1)
  const lastMonthDate = new Date(year.value, month.value + 1, 0).getDate()
  const startWeekDay = (firstMonthDate.getDay() + 6) % 7
  let index = 0
  for (let i = 0; i < startWeekDay; i++) {
    ds.push({
      date: undefined,
      index: ++index,
    })
  }
  for (let i = 1; i <= lastMonthDate; i++) {
    ds.push({
      date: new Date(year.value, month.value, i),
      index: ++index,
    })
  }
  return ds
})

onMounted(() => {
  init()
})

watch(props, () => {
  init()
})

function next() {
  if (month.value == 11) {
    year.value = year.value + 1
    month.value = 0
  } else {
    month.value = month.value + 1
  }
}

function prev() {
  if (month.value == 0) {
    year.value = year.value - 1
    month.value = 11
  } else {
    month.value = month.value - 1
  }
}

function setDate(date?: Date) {
  if (date) {
    selectedDate.value = date
    const day = date.getDate()
    const m = month.value < 9 ? `0${month.value + 1}` : month.value + 1
    const d = day < 10 ? `0${day}` : day
    emits('getDate', `${year.value}-${m}-${d}`)
  }
}

function viewSelectedDate(date: Date) {
  return date.toLocaleString(lang.value, { month: 'long', year: 'numeric', day: '2-digit' })
}

function init() {
  if (props.initDate) {
    const d = new Date(props.initDate)
    year.value = d.getFullYear()
    month.value = d.getMonth()
    selectedDate.value = d
  }
}

function selectedClass(date?: Date): boolean {
  if (!date) return false
  return (
    `${year.value}-${month.value}-${date.getDate()}` ===
    `${selectedDate.value.getFullYear()}-${selectedDate.value.getMonth()}-${selectedDate.value.getDate()}`
  )
}
</script>

<template>
  <div class="container">
    <div class="header">
      <button @click="prev" class="arrow-left" />
      <div class="">{{ headerDate }}</div>
      <button @click="next" class="arrow-right" />
    </div>
    <div class="weeks mt-1">
      <div v-for="week in weeks" :key="week">
        {{ week }}
      </div>
    </div>
    <div class="days">
      <div v-for="day in days" :key="day.index">
        <div
          @click="setDate(day.date)"
          class="day"
          :class="{ 'selected-day': selectedClass(day.date) }"
        >
          {{ day.date ? day.date.getDate() : day.date }}
        </div>
      </div>
    </div>
    <div class="mt-1">
      {{ lang === 'ru' ? 'Выбраная дата' : 'Selected date' }}: {{ viewSelectedDate(selectedDate) }}
    </div>
    <div class="mt-1">
      <button @click="lang = 'ru'">RU</button>
      <button @click="lang = 'en'">EN</button>
    </div>
  </div>
</template>

<style scoped>
.container {
  width: 450px;
  text-align: center;
}
.header {
  display: grid;
  grid-template-columns: auto 1fr auto;
  text-align: center;
}

.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-auto-rows: 50px;
  place-items: center;
}

.weeks {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-auto-rows: 50px;
  place-items: center;
}

.selected-day {
  border: blue solid 1px;
}

.day {
  cursor: pointer;
  display: flex;
  flex: 1;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
}
.arrow-right::before {
  content: '►';
}
.arrow-left::before {
  content: '◄';
}
.mt-1 {
  margin-top: 16px;
}
</style>
