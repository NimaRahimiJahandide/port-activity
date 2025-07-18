<template>
  <div class="relative inline-block w-full" ref="datepicker">
    <!-- Input field -->
    <div class="relative">
      <input type="text" :value="displayValue" @click="togglePicker" @keydown="handleKeydown" :class="inputClass"
        readonly :placeholder="placeholder" />
    </div>

    <!-- Calendar popup -->
    <div v-if="isOpen" :style="popupStyle"
      class="fixed bg-white rounded-xl shadow-lg border border-gray-100 overflow-hidden min-w-[420px] z-50"
      @click.stop
    >
      <section>
        <div class="flex">
          <!-- Left side - Calendar -->
          <div class="flex-1">
            <!-- Header -->
            <div class="flex items-center justify-between mb-2 pb-2 pt-2 border-b border-gray-100">
              <div class="flex gap-1">
                <button @click="previousYear"
                  class="p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-700 rounded-md transition cursor-pointer"
                  title="Previous Year">
                  <svg class="w-4 h-4" fill="none" stroke="#1990ff" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M11 19l-7-7 7-7M19 19l-7-7 7-7" />
                  </svg>
                </button>
                <button @click="previousMonth"
                  class="p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-700 rounded-md transition cursor-pointer"
                  title="Previous Month">
                  <svg class="w-4 h-4" fill="none" stroke="#1990ff" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                  </svg>
                </button>
              </div>
              <div class="text-base font-semibold text-gray-700">
                {{ formatMonthYear(currentMonth) }}
              </div>
              <div class="flex gap-1">
                <button @click="nextMonth"
                  class="p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-700 rounded-md transition cursor-pointer"
                  title="Next Month">
                  <svg class="w-4 h-4" fill="none" stroke="#1990ff" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                  </svg>
                </button>
                <button @click="nextYear"
                  class="p-2 text-gray-500 hover:bg-gray-100 hover:text-gray-700 rounded-md transition cursor-pointer"
                  title="Next Year">
                  <svg class="w-4 h-4" fill="none" stroke="#1990ff" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M13 5l7 7-7 7M5 5l7 7-7 7" />
                  </svg>
                </button>
              </div>
            </div>

            <!-- Days of week -->
            <div class="grid grid-cols-7 gap-0.5 mb-2">
              <div v-for="day in daysOfWeek" :key="day" class="text-center text-sm font-medium text-gray-500 py-2">
                {{ day }}
              </div>
            </div>

            <!-- Calendar grid -->
            <div class="grid grid-cols-7 gap-0.5">
              <button v-for="day in calendarDays" :key="day.key" @click="selectDate(day)" :class="getDayClass(day)"
                :disabled="day.disabled">
                {{ day.date }}
              </button>
            </div>
          </div>

          <!-- Right side - Time picker -->
          <div v-if="enableTimePicker"
            class="flex flex-col justify-start flex-shrink-0 w-[220px] border-l border-gray-200 ">
            <div class="mb-5">
              <div class="flex justify-center gap-2 mt-2 text-lg font-semibold text-gray-700">
                <span>{{ selectedHour.toString().padStart(2, '0') }}</span>:
                <span>{{ selectedMinute.toString().padStart(2, '0') }}</span>:
                <span>{{ selectedSecond.toString().padStart(2, '0') }}</span>
              </div>
            </div>
            <div class="flex gap-2 justify-center border-t border-gray-100 -mt-2 pt-2">
              <!-- Hours column -->
              <div class="flex flex-col items-center border-r border-gray-100 pr-2 w-[60px]">
                <!-- Selected hour at top -->
                <div class="rounded-lg bg-[#e6f7ff] text-blue-700 font-bold cursor-pointer text-center py-2 text-md w-full mb-1">
                  {{ selectedHour.toString().padStart(2, '0') }}
                </div>
                <!-- Scrollable list of other hours -->
                <div class="overflow-y-auto max-h-[220px] w-full rounded-lg scrollbar-hide" style="scrollbar-width: none; -ms-overflow-style: none;">
                  <div v-for="h in hoursList" :key="'h-' + h"
                    @click="selectHour(h)" class="rounded-lg"
                    :class="['hover:bg-blue-50', 'cursor-pointer text-center py-2 text-md']">
                    {{ h.toString().padStart(2, '0') }}
                  </div>
                </div>
              </div>
              <!-- Minutes column -->
              <div class="flex flex-col items-center border-r border-gray-100 pr-2 w-[60px]">
                <!-- Selected minute at top -->
                <div class="rounded-lg bg-[#e6f7ff] text-blue-700 font-bold cursor-pointer text-center py-2 text-md w-full mb-1">
                  {{ selectedMinute.toString().padStart(2, '0') }}
                </div>
                <!-- Scrollable list of other minutes -->
                <div class="overflow-y-auto max-h-[220px] w-full rounded-lg scrollbar-hide" style="scrollbar-width: none; -ms-overflow-style: none;">
                  <div v-for="m in minutesList" :key="'m-' + m"
                    @click="selectMinute(m)" class="rounded-lg"
                    :class="['hover:bg-blue-50', 'cursor-pointer text-center py-2 text-md']">
                    {{ m.toString().padStart(2, '0') }}
                  </div>
                </div>
              </div>
              <!-- Seconds column -->
              <div class="flex flex-col items-center w-[60px]">
                <!-- Selected second at top -->
                <div class="rounded-lg bg-[#e6f7ff] text-blue-700 font-bold cursor-pointer text-center py-2 text-md w-full mb-1">
                  {{ selectedSecond.toString().padStart(2, '0') }}
                </div>
                <!-- Scrollable list of other seconds -->
                <div class="overflow-y-auto max-h-[220px] w-full rounded-lg scrollbar-hide" style="scrollbar-width: none; -ms-overflow-style: none;">
                  <div v-for="s in secondsList" :key="'s-' + s"
                    @click="selectSecond(s)" class="rounded-lg"
                    :class="['hover:bg-blue-50', 'cursor-pointer text-center py-2 text-md']">
                    {{ s.toString().padStart(2, '0') }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Footer -->
        <div class="flex justify-between items-center py-2 border-t border-gray-200">
          <button @click="selectToday"
            class="text-[#1890ff] ml-4 py-2 rounded-md text-sm font-medium cursor-pointer transition">
            Now
          </button>
          <button @click="confirmSelection"
            class="bg-blue-500 text-white mr-4 px-3 py-2 rounded-md text-sm font-medium  transition cursor-pointer">
            OK
          </button>
        </div>
      </section>
    </div>
  </div>
</template>
<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'

const props = defineProps({
  modelValue: [String, Date],
  format: {
    type: String,
    default: 'dd/MM/yyyy HH:mm:ss'
  },
  enableTimePicker: {
    type: Boolean,
    default: true
  },
  placeholder: {
    type: String,
    default: 'Select date...'
  },
  inputClass: {
    type: String,
    default: 'w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent'
  }
})

const emit = defineEmits(['update:modelValue'])

// State
const isOpen = ref(false)
const datepicker = ref(null)
const currentMonth = ref(new Date())
const selectedDate = ref(null)
const selectedHour = ref(new Date().getHours())
const selectedMinute = ref(new Date().getMinutes())
const selectedSecond = ref(new Date().getSeconds())
const popupStyle = ref({})

// Days of week
const daysOfWeek = ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa']

// Computed
const displayValue = computed(() => {
  if (!props.modelValue) return ''
  const date = new Date(props.modelValue)
  return formatDate(date, props.format)
})

const calendarDays = computed(() => {
  const year = currentMonth.value.getFullYear()
  const month = currentMonth.value.getMonth()

  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const startDate = new Date(firstDay)
  startDate.setDate(startDate.getDate() - firstDay.getDay())

  const days = []
  const current = new Date(startDate)

  for (let i = 0; i < 42; i++) {
    const isCurrentMonth = current.getMonth() === month
    const isToday = isDateToday(current)
    const isSelected = selectedDate.value && isSameDate(current, selectedDate.value)

    days.push({
      key: `${current.getFullYear()}-${current.getMonth()}-${current.getDate()}`,
      date: current.getDate(),
      fullDate: new Date(current),
      isCurrentMonth,
      isToday,
      isSelected,
      disabled: !isCurrentMonth
    })

    current.setDate(current.getDate() + 1)
  }

  return days
})

// Computed arrays for time picker columns (excluding selected value)
const hoursList = computed(() => Array.from({length: 24}, (_, i) => i).filter(h => h !== selectedHour.value))
const minutesList = computed(() => Array.from({length: 60}, (_, i) => i).filter(m => m !== selectedMinute.value))
const secondsList = computed(() => Array.from({length: 60}, (_, i) => i).filter(s => s !== selectedSecond.value))

// Methods
const togglePicker = () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    nextTick(() => {
      positionPopup()
      initializeSelection()
    })
  }
}

const positionPopup = () => {
  if (!datepicker.value) return

  const input = datepicker.value.querySelector('input')
  const popup = datepicker.value.querySelector('.calendar-popup')

  if (!input || !popup) return

  const rect = input.getBoundingClientRect()
  const viewportHeight = window.innerHeight
  const popupHeight = 400

  let top = rect.bottom + window.scrollY + 4
  let left = rect.left + window.scrollX

  // Check if there's enough space below
  if (rect.bottom + popupHeight > viewportHeight) {
    top = rect.top + window.scrollY - popupHeight - 4
  }

  // Ensure popup doesn't go off-screen horizontally
  const maxLeft = window.innerWidth - 440 // Updated width
  if (left > maxLeft) {
    left = maxLeft
  }
  if (left < 0) {
    left = 0
  }

  popupStyle.value = {
    position: 'fixed',
    top: `${top}px`,
    left: `${left}px`,
    zIndex: 9999
  }
}

const initializeSelection = () => {
  if (props.modelValue) {
    const date = new Date(props.modelValue)
    selectedDate.value = date
    currentMonth.value = new Date(date.getFullYear(), date.getMonth(), 1)
    selectedHour.value = date.getHours()
    selectedMinute.value = date.getMinutes()
    selectedSecond.value = date.getSeconds()
  } else {
    const now = new Date()
    selectedHour.value = now.getHours()
    selectedMinute.value = now.getMinutes()
    selectedSecond.value = now.getSeconds()
  }
}

const previousMonth = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear(), currentMonth.value.getMonth() - 1, 1)
}

const nextMonth = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear(), currentMonth.value.getMonth() + 1, 1)
}

const previousYear = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear() - 1, currentMonth.value.getMonth(), 1)
}

const nextYear = () => {
  currentMonth.value = new Date(currentMonth.value.getFullYear() + 1, currentMonth.value.getMonth(), 1)
}

const selectDate = (day) => {
  if (day.disabled) return
  selectedDate.value = new Date(day.fullDate)
  updateTime()
}

const selectToday = () => {
  const today = new Date()
  selectedDate.value = today
  currentMonth.value = new Date(today.getFullYear(), today.getMonth(), 1)
  selectedHour.value = today.getHours()
  selectedMinute.value = today.getMinutes()
  selectedSecond.value = today.getSeconds()
  updateTime()
}

const updateTime = () => {
  if (!selectedDate.value) return

  const newDate = new Date(selectedDate.value)
  newDate.setHours(parseInt(selectedHour.value) || 0)
  newDate.setMinutes(parseInt(selectedMinute.value) || 0)
  newDate.setSeconds(parseInt(selectedSecond.value) || 0)

  selectedDate.value = newDate
}

const confirmSelection = () => {
  if (selectedDate.value) {
    emit('update:modelValue', selectedDate.value)
  }
  isOpen.value = false
}

const getDayClass = (day) => {
  const classes = [
    'aspect-square min-h-[40px]',
    'flex items-center justify-center',
    'text-sm font-medium',
    'rounded-lg transition',
  ];

  if (day.disabled) {
    classes.push('text-gray-300', 'cursor-not-allowed');
  } else if (day.isSelected) {
    classes.push('bg-[#1890ff]', 'text-white', 'font-semibold', 'hover:bg-blue-600', 'cursor-pointer');
  } else if (day.isToday) {
    classes.push('bg-[#e6f7ff]', 'text-blue-900', 'font-semibold', 'cursor-pointer');
  } else {
    classes.push('text-gray-700', 'hover:bg-gray-100', 'cursor-pointer');
  }

  if (!day.isCurrentMonth && !day.disabled) {
    classes.push('text-gray-400');
  }

  return classes.join(' ');
};


const formatMonthYear = (date) => {
  return date.toLocaleDateString('en-US', { month: 'long', year: 'numeric' })
}

const formatDate = (date, format) => {
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const year = date.getFullYear()
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return format
    .replace('dd', day)
    .replace('MM', month)
    .replace('yyyy', year)
    .replace('HH', hours)
    .replace('mm', minutes)
    .replace('ss', seconds)
}

const isDateToday = (date) => {
  const today = new Date()
  return isSameDate(date, today)
}

const isSameDate = (date1, date2) => {
  return date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
}

const handleKeydown = (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault()
    togglePicker()
  } else if (e.key === 'Escape') {
    isOpen.value = false
  }
}

const handleClickOutside = (event) => {
  if (datepicker.value && !datepicker.value.contains(event.target)) {
    isOpen.value = false
  }
}

const selectHour = (h) => {
  selectedHour.value = h
  updateTime()
}
const selectMinute = (m) => {
  selectedMinute.value = m
  updateTime()
}
const selectSecond = (s) => {
  selectedSecond.value = s
  updateTime()
}


// Lifecycle
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  window.addEventListener('scroll', positionPopup)
  window.addEventListener('resize', positionPopup)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  window.removeEventListener('scroll', positionPopup)
  window.removeEventListener('resize', positionPopup)
})

// Watch for model value changes
watch(() => props.modelValue, (newValue) => {
  if (newValue) {
    const date = new Date(newValue)
    selectedDate.value = date
    currentMonth.value = new Date(date.getFullYear(), date.getMonth(), 1)
  }
})
</script>
<style scoped>
.scrollbar-hide::-webkit-scrollbar {
  display: none;
}
.scrollbar-hide {
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;     /* Firefox */
}
</style>