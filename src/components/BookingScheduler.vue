<template>
  <div class="booking-scheduler">
    <h2 class="text-2xl font-bold mb-4">Booking Scheduler</h2>
    
    <!-- Visit Duration -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700">Visit Duration</label>
      <select v-model="visitDuration" class="mt-1 block w-full pl-3 pr-1 py-2 text-base border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md">
        <option v-for="duration in durations" :key="duration" :value="duration">{{ duration }} min</option>
      </select>
    </div>

    <!-- Number of Bookings -->
    <div class="mb-4">
      <label class="block text-sm font-medium text-gray-700">No. of Booking / Session</label>
      <input v-model.number="bookingsPerSlot" type="number" min="1" class="mt-1 focus:ring-indigo-500 focus:border-indigo-500 block w-full shadow-sm sm:text-sm border-gray-300 rounded-md">
    </div>

    <!-- Allow Video Tour Call -->
    <div class="mb-4">
      <label class="inline-flex items-center">
        <input v-model="allowVideoCall" type="checkbox" class="form-checkbox h-5 w-5 text-indigo-600">
        <span class="ml-2 text-gray-700">Allow video tour call</span>
      </label>
    </div>

    <!-- Availability Schedule -->
    <div class="mb-4">
      <h3 class="text-lg font-semibold mb-2">Availability</h3>
      <p class="text-sm text-gray-600 mb-2">Set your weekly recurring schedule</p>
      <div v-for="day in weekDays" :key="day" class="mb-2">
        <div class="flex items-center">
          <input :id="day" v-model="schedule[day].enabled" type="checkbox" class="h-4 w-4 text-indigo-600 focus:ring-indigo-500 border-gray-300 rounded">
          <label :for="day" class="ml-2 block text-sm text-gray-900">{{ day }}</label>
        </div>
        <div v-if="schedule[day].enabled" class="ml-6 mt-2">
          <div v-for="(slot, index) in schedule[day].slots" :key="index" class="flex items-center space-x-2 mb-2">
            <select @input="setEndSlot($event, day, index)" v-model="slot.start" class="block w-24 pl-3 pr-1 py-2 text-base border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md">
              <option v-for="time in timeOptions" :key="time" :value="time">{{ time }}</option>
            </select>
            <span>-</span>
            <span>{{ slot.end }}</span>
            <button @click="removeTimeSlot(day, index)" class="text-red-600 hover:text-red-800">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
              </svg>
            </button>
          </div>
          <button @click="addTimeSlot(day)" class="mt-2 inline-flex items-center px-2 py-1 border border-transparent text-xs font-medium rounded shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
            Add Time Slot
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue'

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

const visitDuration = ref<number>(15)
const durations = [15, 30, 45, 60, 90]
const bookingsPerSlot = ref<number>(4)
const allowVideoCall = ref<boolean>(false)
const weekDays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
const timeOptions = generateTimeOptions()

const schedule = reactive<WeekSchedule>(
  weekDays.reduce((acc, day) => {
    acc[day] = {
      enabled: !['Sat', 'Sun'].includes(day),
      slots: generateDefaultSlots()
    }
    return acc
  }, {} as WeekSchedule)
)

const emit = defineEmits<{
  (e: 'update:schedule', schedule: WeekSchedule): void
}>()

watch(schedule, (newSchedule) => {
  emit('update:schedule', newSchedule)
}, { deep: true })

function generateDefaultSlots(): TimeSlot[] {

  return [{ start: '09:00', end: new Date(2000, 0, 1, 9, 0 + visitDuration.value).toTimeString().slice(0, 5) }]
}

function generateTimeOptions(): string[] {
  const options: string[] = []
  for (let hour = 7; hour <= 19; hour++) {
    for (let minute = 0; minute < 60; minute += 15) {
      const time = `${hour.toString().padStart(2, '0')}:${minute.toString().padStart(2, '0')}`
      options.push(time)
    }
  }
  return options
}

function addTimeSlot(day: string): void {
  if (schedule[day].slots.length >= bookingsPerSlot.value) {
    return
  }
  const lastSlot = schedule[day].slots[schedule[day].slots.length - 1]
  const newStart = lastSlot?.end || '09:00'
  const [hours, minutes] = newStart.split(':').map(Number)
  const newEnd = new Date(2000, 0, 1, hours, minutes + visitDuration.value).toTimeString().slice(0, 5)
  schedule[day].slots.push({ start: newStart, end: newEnd })
}

function removeTimeSlot(day: string, index: number): void {
  schedule[day].slots.splice(index, 1)
  if (schedule[day].slots.length === 0) {
    schedule[day].enabled = false
  }
}

function setEndSlot (e: Event, day: string, index: number): void {
    schedule[day].slots[index] = ({
      start: e?.target?.value,
      end: calculateEndTime((e.target as HTMLSelectElement).value)
    })
}

function calculateEndTime (startTime: string): string {
  const durationMinutes = visitDuration.value
  const [hours, minutes] = startTime.split(':').map(Number)
  return new Date(2000, 0, 1, hours, minutes + durationMinutes).toTimeString().slice(0, 5)
}
</script>
<style scoped>
.booking-scheduler {
  width: 40vw;
}
</style>