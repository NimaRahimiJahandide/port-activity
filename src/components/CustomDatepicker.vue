<template>
    <div class="custom-datepicker" ref="datepicker">
      <!-- Input field -->
      <div class="relative">
        <input
          type="text"
          :value="displayValue"
          @click="togglePicker"
          @keydown="handleKeydown"
          :class="inputClass"
          readonly
          :placeholder="placeholder"
        />
        <span class="absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 pointer-events-none">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/>
          </svg>
        </span>
      </div>
  
      <!-- Calendar popup -->
      <div 
        v-if="isOpen"
        class="calendar-popup"
        :style="popupStyle"
      >
        <div class="calendar-container">
          <div class="calendar-layout">
            <!-- Left side - Calendar -->
            <div class="calendar-section">
              <!-- Header -->
              <div class="calendar-header">
                <button @click="previousMonth" class="nav-button">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/>
                  </svg>
                </button>
                <div class="month-year">
                  {{ formatMonthYear(currentMonth) }}
                </div>
                <button @click="nextMonth" class="nav-button">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
                  </svg>
                </button>
              </div>
  
              <!-- Days of week -->
              <div class="days-header">
                <div v-for="day in daysOfWeek" :key="day" class="day-name">
                  {{ day }}
                </div>
              </div>
  
              <!-- Calendar grid -->
              <div class="calendar-grid">
                <button
                  v-for="day in calendarDays"
                  :key="day.key"
                  @click="selectDate(day)"
                  :class="getDayClass(day)"
                  :disabled="day.disabled"
                >
                  {{ day.date }}
                </button>
              </div>
            </div>
  
            <!-- Right side - Time picker (if enabled) -->
            <div v-if="enableTimePicker" class="time-picker">
              <div class="time-picker-header">
                <h3 class="time-picker-title">Time</h3>
              </div>
              <div class="time-inputs">
                <div class="time-input-group">
                  <label class="time-label">Hours</label>
                  <input
                    v-model="selectedHour"
                    type="number"
                    min="0"
                    max="23"
                    class="time-input"
                    @change="updateTime"
                  />
                </div>
                <div class="time-separator">:</div>
                <div class="time-input-group">
                  <label class="time-label">Minutes</label>
                  <input
                    v-model="selectedMinute"
                    type="number"
                    min="0"
                    max="59"
                    class="time-input"
                    @change="updateTime"
                  />
                </div>
                <div class="time-separator">:</div>
                <div class="time-input-group">
                  <label class="time-label">Seconds</label>
                  <input
                    v-model="selectedSecond"
                    type="number"
                    min="0"
                    max="59"
                    class="time-input"
                    @change="updateTime"
                  />
                </div>
              </div>
            </div>
          </div>
  
          <!-- Footer -->
          <div class="calendar-footer">
            <button @click="selectToday" class="today-button">
              Today
            </button>
            <button @click="confirmSelection" class="ok-button">
              OK
            </button>
          </div>
        </div>
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
    const classes = ['calendar-day']
    
    if (day.disabled) classes.push('disabled')
    if (day.isToday) classes.push('today')
    if (day.isSelected) classes.push('selected')
    if (!day.isCurrentMonth) classes.push('other-month')
    
    return classes.join(' ')
  }
  
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
  .custom-datepicker {
    position: relative;
    display: inline-block;
    width: 100%;
  }
  
  .calendar-popup {
    position: fixed;
    background: white;
    border-radius: 12px;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
    border: 1px solid #e5e7eb;
    overflow: hidden;
    min-width: 420px;
  }
  
  .calendar-container {
    padding: 20px;
  }
  
  .calendar-layout {
    display: flex;
    gap: 20px;
  }
  
  .calendar-section {
    flex: 1;
  }
  
  .calendar-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 20px;
  }
  
  .nav-button {
    padding: 8px;
    border: none;
    background: none;
    color: #6b7280;
    cursor: pointer;
    border-radius: 6px;
    transition: all 0.2s;
  }
  
  .nav-button:hover {
    background: #f3f4f6;
    color: #374151;
  }
  
  .month-year {
    font-size: 16px;
    font-weight: 600;
    color: #374151;
  }
  
  .days-header {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 2px;
    margin-bottom: 8px;
  }
  
  .day-name {
    text-align: center;
    font-size: 14px;
    font-weight: 500;
    color: #6b7280;
    padding: 8px 0;
  }
  
  .calendar-grid {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 2px;
  }
  
  .calendar-day {
    aspect-ratio: 1;
    border: none;
    background: none;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 14px;
    font-weight: 500;
    color: #374151;
    border-radius: 8px;
    transition: all 0.2s;
    min-height: 40px;
  }
  
  .calendar-day:hover:not(.disabled) {
    background: #f3f4f6;
  }
  
  .calendar-day.disabled {
    color: #d1d5db;
    cursor: not-allowed;
  }
  
  .calendar-day.other-month {
    color: #9ca3af;
  }
  
  .calendar-day.today {
    background: #dbeafe;
    color: #1e40af;
    font-weight: 600;
  }
  
  .calendar-day.selected {
    background: #3b82f6;
    color: white;
    font-weight: 600;
  }
  
  .calendar-day.selected:hover {
    background: #2563eb;
  }
  
  .time-picker {
    flex: 0 0 140px;
    border-left: 1px solid #e5e7eb;
    padding-left: 20px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
  }
  
  .time-picker-header {
    margin-bottom: 20px;
  }
  
  .time-picker-title {
    font-size: 14px;
    font-weight: 600;
    color: #374151;
    margin: 0;
  }
  
  .time-inputs {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }
  
  .time-input-group {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .time-label {
    font-size: 12px;
    color: #6b7280;
    font-weight: 500;
    min-width: 40px;
  }
  
  .time-input {
    width: 50px;
    padding: 6px 8px;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    text-align: center;
    font-size: 13px;
    font-weight: 500;
  }
  
  .time-input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }
  
  .time-separator {
    font-size: 14px;
    font-weight: 600;
    color: #374151;
    margin: 0;
    text-align: center;
  }
  
  .calendar-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
    padding-top: 20px;
    border-top: 1px solid #e5e7eb;
  }
  
  .today-button {
    background: none;
    border: 1px solid #d1d5db;
    color: #6b7280;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
    font-weight: 500;
    transition: all 0.2s;
  }
  
  .today-button:hover {
    background: #f9fafb;
    border-color: #9ca3af;
  }
  
  .ok-button {
    background: #3b82f6;
    color: white;
    border: none;
    padding: 8px 20px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 14px;
    font-weight: 500;
    transition: all 0.2s;
  }
  
  .ok-button:hover {
    background: #2563eb;
  }
  
  .ok-button:focus {
    outline: none;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.3);
  }
  </style>