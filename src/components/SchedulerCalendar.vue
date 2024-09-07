<template>
  <div class="scheduler-calendar">
    <h2 class="text-2xl font-bold mb-4">Calendar View</h2>
    <div class="flex justify-between items-center mb-4">
      <h3 class="text-xl font-semibold">{{ currentMonth }} {{ currentYear }}</h3>
      <div>
        <button @click="previousWeek" class="mr-2 px-2 py-1 bg-gray-200 rounded">&lt;</button>
        <button @click="nextWeek" class="px-2 py-1 bg-gray-200 rounded">&gt;</button>
      </div>
    </div>
    <div class="grid grid-cols-7 gap-2">
      <div v-for="day in weekDays" :key="day" class="text-center font-semibold">
        {{ day }}
      </div>
      <div v-for="(day, index) in week" :key="index" class="w-32 border rounded p-2 h-96 overflow-y-auto">
        <div class="text-sm font-semibold mb-2">{{ formatDate(day) }}</div>
        <div v-for="hour in businessHours" :key="hour" class="relative">
          <div class="absolute w-full border-t border-gray-200" :style="{ top: `${(hour - 7) * 60}px` }"></div>
          <div class="absolute text-xs text-gray-500" :style="{ top: `${(hour - 7) * 60}px` }">
            {{ formatHour(hour) }}
          </div>
          <div v-for="slot in getSlotsForDay(day, hour)" :key="slot.start" 
               class="absolute bg-blue-200 rounded p-1 text-xs" 
               :style="getSlotStyle(slot)">
            {{ slot.start }} - {{ slot.end }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { format, addDays, startOfWeek, endOfWeek, eachDayOfInterval } from 'date-fns'

interface Slot {
  start: string;
  end: string;
}

interface DaySchedule {
  enabled: boolean;
  slots: Slot[];
}

interface WeekSchedule {
  [key: string]: DaySchedule;
}

const props = defineProps<{
  schedule: WeekSchedule;
}>()

const currentDate = ref(new Date())
const weekDays = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
const businessHours = Array.from({ length: 13 }, (_, i) => i + 7);

const week = computed(() => {
  const start = startOfWeek(currentDate.value)
  const end = endOfWeek(currentDate.value)
  return eachDayOfInterval({ start, end })
})

const currentMonth = computed(() => format(currentDate.value, 'MMMM'))
const currentYear = computed(() => format(currentDate.value, 'yyyy'))

function previousWeek() {
  currentDate.value = addDays(currentDate.value, -7)
}

function nextWeek() {
  currentDate.value = addDays(currentDate.value, 7)
}

function formatDate(date: Date) {
  return format(date, 'd')
}

function formatHour(hour: number): string {
  const date = new Date();
  date.setHours(hour, 0, 0, 0);
  return date.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit', hour12: false });
}


function getSlotsForDay(date: Date, hour: number) {
  const day = weekDays[date.getDay()]
  if (!props.schedule[day]?.enabled) return []
  
  return props.schedule[day].slots.filter(slot => {
    const [startHour] = slot.start.split(':').map(Number)
    return startHour === hour
  })
}

function getSlotStyle(slot: Slot) {
  const [startHour, startMinute] = slot.start.split(':').map(Number)
  const [endHour, endMinute] = slot.end.split(':').map(Number)
  
  const top = (startHour - 7) * 60 + startMinute
  const height = (endHour - startHour) * 60 + (endMinute - startMinute)
  
  return {
    top: `${top}px`,
    height: `${height}px`,
    left: '20px',
    right: '0'
  }
}

watch(() => props.schedule, () => {
  // Trigger re-render when schedule changes
}, { deep: true })
</script>

<style scoped>
.scheduler-calendar {
  height: 800px;
  overflow-y: auto;
}
</style>