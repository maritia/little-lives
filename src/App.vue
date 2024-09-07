<template>
  <div class="flex">
    <BookingScheduler @update:schedule="updateSchedule" />
    <SchedulerCalendar :schedule="schedule" />
    
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import BookingScheduler from './components/BookingScheduler.vue'
import SchedulerCalendar from './components/SchedulerCalendar.vue'


interface TimeSlot {
  start: string;
  end: string;
}

interface DaySchedule {
  enabled: boolean;
  slots: TimeSlot[];
}

interface WeekSchedule {
  [key: string]: DaySchedule;
}

const schedule = ref<WeekSchedule>({})

function updateSchedule(newSchedule: WeekSchedule) {
  schedule.value = newSchedule
}

</script>

<style scoped>
/* Tailwind CSS is used, so minimal custom styling needed */
</style>
