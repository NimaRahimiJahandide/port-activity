<template>
  <div class="custom-dropdown" ref="dropdown">
    <div 
      class="dropdown-trigger w-full px-2 py-1 rounded text-sm"
      :class="{ 'open': isOpen }"
      @click="toggle"
      @keydown="handleKeydown"
      tabindex="0"
    >
      <span class="dropdown-value">{{ displayValue }}</span>
      <div class="dropdown-arrow"></div>
    </div>
    <div 
      class="dropdown-menu"
      :class="{ 'open': isOpen }"
    >
      <div 
        v-for="option in options" 
        :key="option"
        class="dropdown-item"
        :class="{ 'selected': option === modelValue }"
        @click="selectItem(option)"
      >
        {{ option }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

const props = defineProps({
  modelValue: String,
  options: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['update:modelValue'])

const isOpen = ref(false)
const dropdown = ref(null)

const displayValue = computed(() => {
  return props.modelValue || 'Select...'
})

const toggle = () => {
  isOpen.value = !isOpen.value
  if (isOpen.value) {
    nextTick(() => {
      positionDropdown()
    })
  }
}

const positionDropdown = () => {
  if (!dropdown.value) return
  
  const trigger = dropdown.value.querySelector('.dropdown-trigger')
  const menu = dropdown.value.querySelector('.dropdown-menu')
  
  if (!trigger || !menu) return
  
  const rect = trigger.getBoundingClientRect()
  const viewportHeight = window.innerHeight
  const menuHeight = 200 // max-height از CSS
  
  // موقعیت پایه
  let top = rect.bottom + window.scrollY
  let left = rect.left + window.scrollX
  
  // بررسی اینکه آیا فضای کافی در پایین هست یا نه
  if (rect.bottom + menuHeight > viewportHeight) {
    // اگر فضای کافی نیست، در بالا نمایش بده
    top = rect.top + window.scrollY - menuHeight
  }
  
  // تنظیم موقعیت و عرض
  menu.style.top = `${top}px`
  menu.style.left = `${left}px`
  menu.style.width = `${rect.width}px`
}

const selectItem = (option) => {
  emit('update:modelValue', option)
  isOpen.value = false
}

const handleKeydown = (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault()
    toggle()
  } else if (e.key === 'Escape') {
    isOpen.value = false
  }
}

const handleClickOutside = (event) => {
  if (dropdown.value && !dropdown.value.contains(event.target)) {
    isOpen.value = false
  }
}

const handleScroll = () => {
  if (isOpen.value) {
    positionDropdown()
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  window.addEventListener('scroll', handleScroll, true)
  window.addEventListener('resize', handleScroll)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
  window.removeEventListener('scroll', handleScroll, true)
  window.removeEventListener('resize', handleScroll)
})
</script>

<style scoped>
.custom-dropdown {
  position: relative;
  display: inline-block;
  width: 100%;
}

.dropdown-trigger {
  background: white;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.2s ease;
}

.dropdown-trigger:hover {
  border-color: #9ca3af;
}

.dropdown-trigger.open { 
  color: #cfcece;
}

.dropdown-arrow {
  width: 0;
  height: 0;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid #6b7280;
  transition: transform 0.2s ease;
}

.dropdown-trigger.open .dropdown-arrow {
  border-top: 4px solid #cfcece;
  transform: rotate(180deg);
}

.dropdown-menu {
  position: fixed;
  background: white;
  border-radius: 4px;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  filter: drop-shadow(0 25px 25px rgb(0 0 0 / 0.15));
  z-index: 9999;
  overflow-y: auto;
  display: none;
  min-width: 80px;
  font-weight: 700;
}

.dropdown-menu.open {
  display: block;
}

.dropdown-item {
  padding: 8px 12px;
  cursor: pointer;
  font-size: 14px;
  color: #374151;
  transition: background-color 0.15s ease;
  white-space: nowrap;
}

.dropdown-item:hover {
  background-color: #f3f4f6; /* Light gray background on hover */
}

.dropdown-item.selected {
  background-color: #dbeafe; /* Light blue background for selected item */
  color: #1e40af;
}

.dropdown-item.selected:hover {
  background-color: #bfdbfe; /* Slightly darker blue on hover for selected item */
}
</style>