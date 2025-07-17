<template>
  <div class="p-6 bg-white shadow-sm rounded-lg">
    <!-- Header -->
    <div class="mb-4 flex justify-between items-center">
      <h2 class="text-xl font-semibold text-gray-800 border-l-4 pl-1.5 border-[#5096ff]">Port Activity</h2>
      <button @click="addNewRow"
        class="bg-[#f7f7f7] text-gray-800 px-4 py-2 rounded-md border-2 border-[#ececec] transition-colors cursor-pointer">
        + Add New
      </button>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto">
      <table class="min-w-full rounded-xl overflow-hidden">
        <thead>
          <tr class="bg-[#f1f1f1] text-sm border border-[#f3f3f3] text-gray-700">
            <th class="relative px-3 py-2 text-left">
              <span>Day</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Activity Type</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>From Date &amp; Time</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Duration</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>%</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>To Date Time</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Remarks</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Deductions</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Actions</span>
            </th>
          </tr>
        </thead>

        <tbody>
          <PortActivityRow v-for="(row, index) in rows" :key="row.id" :row="row" :index="index"
            :is-out-of-order="isRowOutOfOrder(index)" :activity-types="activityTypes" @clone="cloneRow"
            @auto-adjust="autoAdjustRow" @confirm-delete="confirmDelete" @update-row="updateToDate"
            @update-deductions="updateDeductions" />
        </tbody>
      </table>
    </div>

    <!-- Delete Modal -->
    <DeleteModal v-if="showDeleteModal" @cancel="cancelDelete" @confirm="deleteRow" />
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import PortActivityRow from './PortActivityRow.vue'
import DeleteModal from './DeleteModal.vue'

// Props
const props = defineProps({
  selectedLayTime: Object
})

// Activity types
const activityTypes = [
  'Loading',
  'Unloading',
  'Waiting',
  'Berthing',
  'Unberthing',
  'Inspection',
  'Bunkering',
  'Maintenance'
]

// States
const rows = ref([])
const showDeleteModal = ref(false)
const deleteIndex = ref(-1)
let nextId = 1

// Helpers
const getCurrentDateTime = () => {
  const now = new Date()
  const offset = now.getTimezoneOffset()
  const localDate = new Date(now.getTime() - offset * 60000)
  return localDate.toISOString().slice(0, 16)
}

const addHoursToDateTime = (start, timeStr) => {
  const [h, m] = timeStr.split(':').map(Number)
  const date = new Date(start)
  date.setMinutes(date.getMinutes() + h * 60 + m)
  const offset = date.getTimezoneOffset()
  const local = new Date(date.getTime() - offset * 60000)
  return local.toISOString().slice(0, 16)
}

const parseDuration = (str) => {
  const [h, m] = str.split(':').map(Number)
  return h + m / 60
}

// Row Logic
const createNewRow = (from = null) => {
  const base = from || getCurrentDateTime()
  return {
    id: nextId++,
    activityType: '',
    fromDate: base,
    duration: '00:00',
    percentage: 0,
    toDate: base,
    remarks: '',
    deductions: '00:00'
  }
}

const addNewRow = () => {
  const last = rows.value[rows.value.length - 1]
  const fromDate = last?.toDate || getCurrentDateTime()
  rows.value.push(createNewRow(fromDate))
}

const updateToDate = (index) => {
  const row = rows.value[index]
  if (row.fromDate && row.duration) {
    row.toDate = addHoursToDateTime(row.fromDate, row.duration)
  } else {
    row.toDate = row.fromDate
  }
  updateDeductions(index)
}

const updateDeductions = (index) => {
  const row = rows.value[index]
  const hours = parseDuration(row.duration)
  const deducted = (hours * row.percentage) / 100
  const h = Math.floor(deducted)
  const m = Math.round((deducted - h) * 60)
  row.deductions = `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}`
}

const isRowOutOfOrder = (index) => {
  if (index === 0) return false
  const curr = new Date(rows.value[index].fromDate)
  const prev = new Date(rows.value[index - 1].fromDate)
  return curr < prev
}

const autoAdjustRow = (index) => {
  const row = rows.value[index]
  rows.value.splice(index, 1)
  const insertAt = rows.value.findIndex(r => new Date(r.fromDate) > new Date(row.fromDate))
  const pos = insertAt === -1 ? rows.value.length : insertAt
  rows.value.splice(pos, 0, row)
}

const cloneRow = (index) => {
  const row = rows.value[index]
  const clone = {
    ...row,
    id: nextId++,
    fromDate: row.toDate,
    toDate: row.toDate,
    remarks: ''
  }
  rows.value.splice(index + 1, 0, clone)
}

const confirmDelete = (index) => {
  deleteIndex.value = index
  showDeleteModal.value = true
}

const cancelDelete = () => {
  showDeleteModal.value = false
  deleteIndex.value = -1
}

const deleteRow = () => {
  if (deleteIndex.value >= 0) rows.value.splice(deleteIndex.value, 1)
  cancelDelete()
}

// Generate mock port activity data based on selected laytime row
function generateMockPortActivities(layTimeRow) {
  if (!layTimeRow) return []
  
  // Convert date format from "YYYY/MM/DD" to "YYYY-MM-DD" for proper ISO string
  const formatDateForInput = (dateStr) => {
    if (!dateStr) return getCurrentDateTime()
    // Convert "2025/03/30" to "2025-03-30"
    return dateStr.replace(/\//g, '-')
  }
  
  const laycanFromFormatted = formatDateForInput(layTimeRow.laycanFrom)
  const laycanToFormatted = formatDateForInput(layTimeRow.laycanTo)
  
  return [
    {
      id: nextId++,
      activityType: 'Loading',
      fromDate: `${laycanFromFormatted}T08:00`,
      duration: '04:00',
      percentage: 10,
      toDate: `${laycanFromFormatted}T12:00`,
      remarks: `Loaded ${layTimeRow.cargo}`,
      deductions: '00:24'
    },
    {
      id: nextId++,
      activityType: 'Waiting',
      fromDate: `${laycanFromFormatted}T12:00`,
      duration: '02:00',
      percentage: 0,
      toDate: `${laycanFromFormatted}T14:00`,
      remarks: 'Waiting for berth',
      deductions: '00:00'
    },
    {
      id: nextId++,
      activityType: 'Unloading',
      fromDate: `${laycanToFormatted}T09:00`,
      duration: '03:00',
      percentage: 5,
      toDate: `${laycanToFormatted}T12:00`,
      remarks: `Unloaded ${layTimeRow.cargo}`,
      deductions: '00:09'
    }
  ]
}

// Watch for changes to selectedLayTime and update rows
watch(
  () => props.selectedLayTime,
  (newVal) => {
    if (newVal) {
      rows.value = generateMockPortActivities(newVal)
    }
  }
)

// Init
addNewRow()
</script>
