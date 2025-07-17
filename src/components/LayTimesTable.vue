<template>
  <div class="p-6 bg-white rounded-lg shadow-sm mb-6">
    <!-- Header -->
    <div class="mb-4 flex justify-between items-center">
      <h2 class="text-xl font-semibold text-gray-800 border-l-4  pl-1.5 border-[#5096ff]">Lay Times</h2>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto">
      <table class="min-w-full rounded-xl overflow-hidden">
        <thead>
          <tr class="bg-[#f1f1f1] text-sm font-medium border border-[#f3f3f3] text-gray-700">
            <th class="relative px-3 py-2 text-left">
              <span>Port Name</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Cargo</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>F</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>BL Code</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Quantity</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>L/D Rate</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Term</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Dem Rate</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Des Rate/D</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Allowed</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Used</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Deduction</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Balance</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Laycan From</span>
              <div class="absolute right-0 top-[6px] bottom-[6px] w-px bg-gray-300"></div>
            </th>
            <th class="relative px-3 py-2 text-left">
              <span>Laycan To</span>
            </th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(row, index) in rows" :key="row.id" :class="{ 'bg-blue-50': selectedRowIndex === index }"
            @click="selectRow(index)" class="hover:bg-gray-50 border border-[#f3f3f3] cursor-pointer">
            <!-- Port Name -->
            <td class="px-3 py-2">
              <div class="flex items-center text-sm gap-1.5">
                <span>{{ row.portName }}</span>
                <img v-if="row.countryFlag" :src="row.countryFlag" :alt="row.portName" class="w-4 h-3 mr-2" />
              </div>
            </td>

            <!-- Cargo -->
            <td class="px-3 py-2 text-sm ">
              <span>{{ row.cargo }}</span>
            </td>

            <!-- F -->
            <td class="px-3 py-2">
              <span>{{ row.f }}</span>
            </td>

            <!-- BL Code -->
            <td class="px-3 py-2">
              <span>{{ row.blCode }}</span>
            </td>

            <!-- Quantity -->
            <td class="px-3 py-2">
              <span>{{ row.quantity }}</span>
            </td>

            <!-- L/D Rate -->
            <td class="px-3 py-2">
              <span>{{ row.ldRate }}</span>
            </td>

            <!-- Term -->
            <td class="px-3 py-2">
              <span>{{ row.term }}</span>
            </td>

            <!-- Dem Rate -->
            <td class="px-3 py-2">
              <span>{{ row.demRate }}</span>
            </td>

            <!-- Des Rate/D -->
            <td class="px-3 py-2">
              <span>{{ row.desRate }}</span>
            </td>

            <!-- Allowed -->
            <td class="px-3 py-2">
              <span>{{ row.allowed }}</span>
            </td>

            <!-- Used -->
            <td class="px-3 py-2">
              <span class="text-sm text-gray-600">{{ formatTime(row.used) }}</span>
            </td>

            <!-- Deduction -->
            <td class="px-3 py-2">
              <span class="text-sm text-gray-600">{{ formatTime(row.deduction) }}</span>
            </td>

            <!-- Balance -->
            <td class="px-3 py-2">
              <span class="text-sm text-gray-600">{{ formatTime(row.balance) }}</span>
            </td>

            <!-- Laycan From -->
            <td class="px-3 py-2">
              <span>{{ row.laycanFrom }}</span>
            </td>

            <!-- Laycan To -->
            <td class="px-3 py-2">
              <span>{{ row.laycanTo }}</span>
            </td>

          </tr>
        </tbody>
      </table>
    </div>

    <!-- Delete Modal -->
    <DeleteModal v-if="showDeleteModal" @cancel="cancelDelete" @confirm="deleteRow" />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import DeleteModal from './DeleteModal.vue'

const emit = defineEmits(['row-selected'])

// Sample country flags (you can replace with actual flag images)
const countryFlags = {
  'USA': '../images/usa.webp',
  'Brazil': '../images/brazil.webp',
  'Singapore': '../images/singapore.webp',
  'Germany': '../images/germany.webp',
  'Russia': '../images/russia.webp'
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

const selectRow = (index) => {
  selectedRowIndex.value = index
  emit('row-selected', rows.value[index])
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
      laycanFrom: '2025/03/30',
      laycanTo: '2025/04/05',
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
      laycanFrom: '2025/05/08',
      laycanTo: '2025/05/12',
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
      laycanFrom: '2025/06/30',
      laycanTo: '2025/06/03',
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