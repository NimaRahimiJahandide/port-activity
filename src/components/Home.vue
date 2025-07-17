<template>
  <div class="p-6 bg-white">
    <div class="mb-4 flex justify-between items-center">
      <h2 class="text-xl font-semibold text-gray-800">Port Activity</h2>
      <button
        @click="addNewRow"
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-md transition-colors"
      >
        + Add New
      </button>
    </div>

    <div class="overflow-x-auto">
      <table class="min-w-full border-collapse border border-gray-300">
        <thead>
          <tr class="bg-gray-50">
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Day</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Activity Type</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">From Date & Time</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Duration</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">%</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">To Date & Time</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Remarks</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Deductions</th>
            <th class="border border-gray-300 px-3 py-2 text-left text-sm font-medium text-gray-700">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(row, index) in rows"
            :key="row.id"
            :class="{ 'bg-red-50 border-red-300': isRowOutOfOrder(index) }"
          >
            <!-- Day -->
            <td class="border border-gray-300 px-3 py-2 text-sm">
              {{ formatDay(row.fromDate) }}
            </td>

            <!-- Activity Type -->
            <td class="border border-gray-300 px-3 py-2">
              <select
                v-model="row.activityType"
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">Select...</option>
                <option v-for="type in activityTypes" :key="type" :value="type">
                  {{ type }}
                </option>
              </select>
            </td>

            <!-- From Date & Time -->
            <td class="border border-gray-300 px-3 py-2">
              <input
                type="datetime-local"
                v-model="row.fromDate"
                @change="updateToDate(index)"
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </td>

            <!-- Duration -->
            <td class="border border-gray-300 px-3 py-2">
              <input
                type="time"
                v-model="row.duration"
                @change="updateToDate(index)"
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </td>

            <!-- % -->
            <td class="border border-gray-300 px-3 py-2">
              <select
                v-model="row.percentage"
                @change="updateDeductions(index)"
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="0">0%</option>
                <option value="50">50%</option>
                <option value="100">100%</option>
              </select>
            </td>

            <!-- To Date & Time -->
            <td class="border border-gray-300 px-3 py-2 text-sm bg-gray-50">
              {{ formatDateTime(row.toDate) }}
            </td>

            <!-- Remarks -->
            <td class="border border-gray-300 px-3 py-2">
              <input
                type="text"
                v-model="row.remarks"
                placeholder="Enter remarks..."
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </td>

            <!-- Deductions -->
            <td class="border border-gray-300 px-3 py-2 text-sm bg-gray-50">
              {{ row.deductions }}
            </td>

            <!-- Actions -->
            <td class="border border-gray-300 px-3 py-2">
              <div class="flex items-center space-x-2">
                <button
                  v-if="isRowOutOfOrder(index)"
                  @click="autoAdjustRow(index)"
                  class="bg-yellow-500 hover:bg-yellow-600 text-white px-2 py-1 rounded text-xs transition-colors"
                  title="Auto Adjust"
                >
                  ⚡
                </button>
                <button
                  @click="cloneRow(index)"
                  class="bg-green-500 hover:bg-green-600 text-white px-2 py-1 rounded text-xs transition-colors"
                  title="Clone"
                >
                  📋
                </button>
                <button
                  @click="confirmDelete(index)"
                  class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded text-xs transition-colors"
                  title="Delete"
                >
                  🗑️
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Delete Confirmation Modal -->
    <div
      v-if="showDeleteModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white p-6 rounded-lg shadow-lg max-w-md w-full mx-4">
        <h3 class="text-lg font-semibold mb-4">Confirm Delete</h3>
        <p class="text-gray-600 mb-6">Are you sure you want to delete this row? This action cannot be undone.</p>
        <div class="flex justify-between">
          <button
            @click="cancelDelete"
            class="px-4 py-2 bg-gray-300 hover:bg-gray-400 text-gray-800 rounded transition-colors"
          >
            Cancel
          </button>
          <button
            @click="deleteRow"
            class="px-4 py-2 bg-red-500 hover:bg-red-600 text-white rounded transition-colors"
          >
            OK
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// Activity types dropdown options
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

// Reactive data
const rows = ref([])
const showDeleteModal = ref(false)
const deleteIndex = ref(-1)
let nextId = 1

// Helper functions
const formatDateTime = (dateTimeString) => {
  if (!dateTimeString) return ''
  const date = new Date(dateTimeString)
  return date.toLocaleString('en-US', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const formatDay = (dateTimeString) => {
  if (!dateTimeString) return ''
  const date = new Date(dateTimeString)
  return date.toLocaleDateString('en-US', { weekday: 'short' })
}

const getCurrentDateTime = () => {
  const now = new Date()
  const offset = now.getTimezoneOffset()
  const localDate = new Date(now.getTime() - offset * 60 * 1000)
  return localDate.toISOString().slice(0, 16)
}

const addHoursToDateTime = (dateTimeString, hours) => {
  if (!dateTimeString || !hours) return dateTimeString
  const date = new Date(dateTimeString)
  const [hoursStr, minutesStr] = hours.split(':')
  const totalMinutes = parseInt(hoursStr) * 60 + parseInt(minutesStr)
  date.setMinutes(date.getMinutes() + totalMinutes)
  const offset = date.getTimezoneOffset()
  const localDate = new Date(date.getTime() - offset * 60 * 1000)
  return localDate.toISOString().slice(0, 16)
}

const calculateDurationInHours = (fromDate, toDate) => {
  if (!fromDate || !toDate) return '00:00'
  const from = new Date(fromDate)
  const to = new Date(toDate)
  const diffMs = to - from
  const diffHours = Math.floor(diffMs / (1000 * 60 * 60))
  const diffMinutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60))
  return `${diffHours.toString().padStart(2, '0')}:${diffMinutes.toString().padStart(2, '0')}`
}

const parseDuration = (durationStr) => {
  if (!durationStr) return 0
  const [hours, minutes] = durationStr.split(':').map(Number)
  return hours + minutes / 60
}

// Row operations
const createNewRow = (fromDate = null) => {
  const currentDateTime = fromDate || getCurrentDateTime()
  return {
    id: nextId++,
    activityType: '',
    fromDate: currentDateTime,
    duration: '00:00',
    percentage: 0,
    toDate: currentDateTime,
    remarks: '',
    deductions: '00:00'
  }
}

const addNewRow = () => {
  let fromDate = getCurrentDateTime()
  
  if (rows.value.length > 0) {
    // Set from date to the last row's to date
    const lastRow = rows.value[rows.value.length - 1]
    fromDate = lastRow.toDate || getCurrentDateTime()
  }
  
  const newRow = createNewRow(fromDate)
  rows.value.push(newRow)
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
  if (row.duration && row.percentage) {
    const durationHours = parseDuration(row.duration)
    const deductionHours = (durationHours * row.percentage) / 100
    const hours = Math.floor(deductionHours)
    const minutes = Math.round((deductionHours - hours) * 60)
    row.deductions = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`
  } else {
    row.deductions = '00:00'
  }
}

const isRowOutOfOrder = (index) => {
  if (index === 0) return false
  const currentRow = rows.value[index]
  const previousRow = rows.value[index - 1]
  
  if (!currentRow.fromDate || !previousRow.fromDate) return false
  
  return new Date(currentRow.fromDate) < new Date(previousRow.fromDate)
}

const autoAdjustRow = (index) => {
  const row = rows.value[index]
  rows.value.splice(index, 1)
  
  // Find the correct position to insert
  let insertIndex = 0
  for (let i = 0; i < rows.value.length; i++) {
    if (new Date(row.fromDate) > new Date(rows.value[i].fromDate)) {
      insertIndex = i + 1
    } else {
      break
    }
  }
  
  rows.value.splice(insertIndex, 0, row)
}

const cloneRow = (index) => {
  const originalRow = rows.value[index]
  const clonedRow = {
    ...originalRow,
    id: nextId++,
    fromDate: originalRow.toDate,
    toDate: originalRow.toDate,
    remarks: ''
  }
  
  rows.value.splice(index + 1, 0, clonedRow)
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
  if (deleteIndex.value >= 0) {
    rows.value.splice(deleteIndex.value, 1)
  }
  showDeleteModal.value = false
  deleteIndex.value = -1
}

// Initialize with one row
addNewRow()
</script>

<style scoped>
/* Additional custom styles if needed */
.bg-red-50 {
  background-color: #fef2f2;
}

.border-red-300 {
  border-color: #fca5a5;
}
</style>