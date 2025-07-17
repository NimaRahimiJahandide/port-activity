<template>
  <div class="p-6 bg-white rounded-lg shadow-sm mb-6">
    <!-- Header -->
    <div class="mb-4 flex justify-between items-center">
      <h2 class="text-xl font-semibold text-gray-800">Lay Times</h2>
      <button
        @click="addNewRow"
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-md transition-colors"
      >
        + Add New
      </button>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto">
      <table class="min-w-full border-collapse border border-gray-300">
        <thead>
          <tr class="bg-gray-50 text-sm font-medium text-gray-700">
            <th class="border px-3 py-2 text-left">Port Name</th>
            <th class="border px-3 py-2 text-left">Cargo</th>
            <th class="border px-3 py-2 text-left">F</th>
            <th class="border px-3 py-2 text-left">BL Code</th>
            <th class="border px-3 py-2 text-left">Quantity</th>
            <th class="border px-3 py-2 text-left">L/D Rate</th>
            <th class="border px-3 py-2 text-left">Term</th>
            <th class="border px-3 py-2 text-left">Dem Rate</th>
            <th class="border px-3 py-2 text-left">Des Rate/D</th>
            <th class="border px-3 py-2 text-left">Allowed</th>
            <th class="border px-3 py-2 text-left">Used</th>
            <th class="border px-3 py-2 text-left">Deduction</th>
            <th class="border px-3 py-2 text-left">Balance</th>
            <th class="border px-3 py-2 text-left">Laycan From</th>
            <th class="border px-3 py-2 text-left">Laycan To</th>
            <th class="border px-3 py-2 text-left">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr 
            v-for="(row, index) in rows" 
            :key="row.id"
            :class="{ 'bg-blue-50 border-blue-300': selectedRowIndex === index }"
            @click="selectRow(index)"
            class="hover:bg-gray-50 cursor-pointer"
          >
            <!-- Port Name -->
            <td class="border px-3 py-2">
              <div class="flex items-center">
                <img 
                  v-if="row.countryFlag" 
                  :src="row.countryFlag" 
                  :alt="row.portName"
                  class="w-4 h-3 mr-2"
                />
                <input 
                  type="text" 
                  v-model="row.portName" 
                  class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                  placeholder="Port Name"
                />
              </div>
            </td>

            <!-- Cargo -->
            <td class="border px-3 py-2">
              <select 
                v-model="row.cargo" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">Select...</option>
                <option value="Crude Oil">Crude Oil</option>
                <option value="Soybeans">Soybeans</option>
                <option value="Chemicals">Chemicals</option>
                <option value="LNG">LNG</option>
                <option value="Container">Container</option>
                <option value="Bulk">Bulk</option>
              </select>
            </td>

            <!-- F -->
            <td class="border px-3 py-2">
              <select 
                v-model="row.f" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">-</option>
                <option value="FOB">FOB</option>
                <option value="CIF">CIF</option>
                <option value="CFR">CFR</option>
                <option value="L">L</option>
              </select>
            </td>

            <!-- BL Code -->
            <td class="border px-3 py-2">
              <input 
                type="text" 
                v-model="row.blCode" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="BL Code"
              />
            </td>

            <!-- Quantity -->
            <td class="border px-3 py-2">
              <input 
                type="number" 
                v-model="row.quantity" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="0"
              />
            </td>

            <!-- L/D Rate -->
            <td class="border px-3 py-2">
              <input 
                type="number" 
                step="0.01"
                v-model="row.ldRate" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="0.00"
              />
            </td>

            <!-- Term -->
            <td class="border px-3 py-2">
              <select 
                v-model="row.term" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">-</option>
                <option value="CIF">CIF</option>
                <option value="FOB">FOB</option>
                <option value="CFR">CFR</option>
                <option value="SHINC">SHINC</option>
              </select>
            </td>

            <!-- Dem Rate -->
            <td class="border px-3 py-2">
              <input 
                type="number" 
                v-model="row.demRate" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="0"
              />
            </td>

            <!-- Des Rate/D -->
            <td class="border px-3 py-2">
              <input 
                type="number" 
                v-model="row.desRate" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="0"
              />
            </td>

            <!-- Allowed -->
            <td class="border px-3 py-2">
              <input 
                type="number" 
                v-model="row.allowed" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="0"
              />
            </td>

            <!-- Used -->
            <td class="border px-3 py-2 bg-gray-50">
              <span class="text-sm text-gray-600">{{ formatTime(row.used) }}</span>
            </td>

            <!-- Deduction -->
            <td class="border px-3 py-2 bg-gray-50">
              <span class="text-sm text-gray-600">{{ formatTime(row.deduction) }}</span>
            </td>

            <!-- Balance -->
            <td class="border px-3 py-2 bg-gray-50">
              <span class="text-sm text-gray-600">{{ formatTime(row.balance) }}</span>
            </td>

            <!-- Laycan From -->
            <td class="border px-3 py-2">
              <input 
                type="date" 
                v-model="row.laycanFrom" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </td>

            <!-- Laycan To -->
            <td class="border px-3 py-2">
              <input 
                type="date" 
                v-model="row.laycanTo" 
                class="w-full px-2 py-1 border rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </td>

            <!-- Actions -->
            <td class="border px-3 py-2">
              <div class="flex items-center space-x-2">
                <button 
                  @click.stop="cloneRow(index)"
                  class="bg-green-500 hover:bg-green-600 text-white px-2 py-1 rounded text-xs" 
                  title="Clone"
                >
                  📋
                </button>
                <button 
                  @click.stop="confirmDelete(index)"
                  class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded text-xs" 
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

    <!-- Delete Modal -->
    <DeleteModal
      v-if="showDeleteModal"
      @cancel="cancelDelete"
      @confirm="deleteRow"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import DeleteModal from './DeleteModal.vue'

const emit = defineEmits(['row-selected'])

// Sample country flags (you can replace with actual flag images)
const countryFlags = {
  'USA': 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIHZpZXdCb3g9IjAgMCAyNCAxOCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIGZpbGw9IiNGRkZGRkYiLz48cGF0aCBkPSJNMCAwSDI0VjE4SDAiIGZpbGw9IiNGRkZGRkYiLz48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIGZpbGw9IiNGRkZGRkYiLz48cGF0aCBkPSJNMCAwSDI0VjE4SDAiIGZpbGw9IiNGRkZGRkYiLz48L3N2Zz4=',
  'Brazil': 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIHZpZXdCb3g9IjAgMCAyNCAxOCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIGZpbGw9IiMwMEJDMDAiLz48L3N2Zz4=',
  'Singapore': 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIHZpZXdCb3g9IjAgMCAyNCAxOCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iOSIgZmlsbD0iI0ZGRkZGRiIvPjxyZWN0IHk9IjkiIHdpZHRoPSIyNCIgaGVpZ2h0PSI5IiBmaWxsPSIjRkYwMDAwIi8+PC9zdmc+',
  'Germany': 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIHZpZXdCb3g9IjAgMCAyNCAxOCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iNiIgZmlsbD0iIzAwMDAwMCIvPjxyZWN0IHk9IjYiIHdpZHRoPSIyNCIgaGVpZ2h0PSI2IiBmaWxsPSIjRkYwMDAwIi8+PHJlY3QgeT0iMTIiIHdpZHRoPSIyNCIgaGVpZ2h0PSI2IiBmaWxsPSIjRkZEQjAwIi8+PC9zdmc+',
  'Russia': 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMTgiIHZpZXdCb3g9IjAgMCAyNCAxOCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMjQiIGhlaWdodD0iNiIgZmlsbD0iI0ZGRkZGRiIvPjxyZWN0IHk9IjYiIHdpZHRoPSIyNCIgaGVpZ2h0PSI2IiBmaWxsPSIjMDA1N0IwIi8+PHJlY3QgeT0iMTIiIHdpZHRoPSIyNCIgaGVpZ2h0PSI2IiBmaWxsPSIjRkYwMDAwIi8+PC9zdmc+'
}

// States
const rows = ref([])
const selectedRowIndex = ref(-1)
const showDeleteModal = ref(false)
const deleteIndex = ref(-1)
let nextId = 1

// Helper functions
const formatTime = (timeString) => {
  return timeString || '00:00:00'
}

const getCurrentDate = () => {
  const now = new Date()
  return now.toISOString().split('T')[0]
}

// Row logic
const createNewRow = () => {
  return {
    id: nextId++,
    portName: '',
    cargo: '',
    f: '',
    blCode: '',
    quantity: 0,
    ldRate: 0,
    term: '',
    demRate: 0,
    desRate: 0,
    allowed: 0,
    used: '00:00:00',
    deduction: '00:00:00',
    balance: '00:00:00',
    laycanFrom: getCurrentDate(),
    laycanTo: getCurrentDate(),
    countryFlag: null
  }
}

const addNewRow = () => {
  rows.value.push(createNewRow())
}

const selectRow = (index) => {
  selectedRowIndex.value = index
  emit('row-selected', rows.value[index])
}

const cloneRow = (index) => {
  const row = rows.value[index]
  const clone = {
    ...row,
    id: nextId++
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
  if (deleteIndex.value >= 0) {
    rows.value.splice(deleteIndex.value, 1)
    if (selectedRowIndex.value === deleteIndex.value) {
      selectedRowIndex.value = -1
    } else if (selectedRowIndex.value > deleteIndex.value) {
      selectedRowIndex.value--
    }
  }
  cancelDelete()
}

// Initialize with sample data
const initializeSampleData = () => {
  rows.value = [
    {
      id: nextId++,
      portName: 'USA',
      cargo: 'Crude Oil',
      f: 'FOB',
      blCode: 'BL001',
      quantity: 100000,
      ldRate: 5.000,
      term: 'CIF',
      demRate: 15000,
      desRate: 12000,
      allowed: 5,
      used: '054:00:00',
      deduction: '014:00:00',
      balance: '014:00:00',
      laycanFrom: '2025-03-30',
      laycanTo: '2025-04-05',
      countryFlag: countryFlags['USA']
    },
    {
      id: nextId++,
      portName: 'Brazil',
      cargo: 'Soybeans',
      f: 'CIF',
      blCode: 'BL002',
      quantity: 80000,
      ldRate: 4.000,
      term: 'FOB',
      demRate: 12000,
      desRate: 9000,
      allowed: 6,
      used: '064:00:00',
      deduction: '014:00:00',
      balance: '014:00:00',
      laycanFrom: '2025-05-08',
      laycanTo: '2025-05-12',
      countryFlag: countryFlags['Brazil']
    },
    {
      id: nextId++,
      portName: 'Singapore',
      cargo: 'Chemicals',
      f: 'FOB',
      blCode: 'BL003',
      quantity: 60000,
      ldRate: 4.500,
      term: 'CFR',
      demRate: 10000,
      desRate: 8000,
      allowed: 4,
      used: '044:00:00',
      deduction: '000:12:00',
      balance: '000:00:00',
      laycanFrom: '2025-06-30',
      laycanTo: '2025-06-03',
      countryFlag: countryFlags['Singapore']
    }
  ]
}

// Expose methods for parent component
defineExpose({
  getSelectedRow: () => selectedRowIndex.value >= 0 ? rows.value[selectedRowIndex.value] : null,
  selectRowById: (id) => {
    const index = rows.value.findIndex(row => row.id === id)
    if (index >= 0) {
      selectRow(index)
    }
  }
})

// Initialize
initializeSampleData()
</script>