<template>
  <main class="min-h-screen bg-gray-100 p-8">
    <LayTimesTable @row-selected="onLayTimeRowSelected" />
    <PortActivityTable ref="portActivityTable" :selected-lay-time="selectedLayTime" />
  </main>
</template>

<script setup>
import { ref } from 'vue'
import LayTimesTable from '../components/LayTimesTable.vue'
import PortActivityTable from '../components/PortActivityTable.vue'

const portActivityTable = ref(null)
const selectedLayTime = ref(null)

const onLayTimeRowSelected = (row) => {
  selectedLayTime.value = row
  // Optional: You can also directly call methods on PortActivityTable if needed
  // portActivityTable.value?.updateFromLayTime(row)
}
</script>