<template>
    <tr class="border-b border-[#f3f3f3]" :class="{ 'bg-red-50 border-red-300': isOutOfOrder }">
        <!-- Day -->
        <td class="px-3 py-2 text-sm">
            {{ formatDay(row.fromDate) }}
        </td>

        <!-- Activity Type -->
        <td class="px-3 py-2">
            <ActivityTypeSelect v-model="row.activityType" :options="activityTypes" />
        </td>

        <!-- From Date & Time -->
        <td class="px-3 py-2">
            <div class="relative flex items-center">
                <Datepicker
                    v-model="row.fromDate"
                    :enable-time-picker="true"
                    :format="'dd/MM/yyyy HH:mm:ss'"
                    @update:model-value="onDateTimeChange"
                    input-class="w-full px-2 py-1 rounded text-base border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500 shadow-sm pr-10"
                    menu-class="z-50 bg-white rounded-lg shadow-lg border border-gray-200 p-2"
                    :clearable="false"
                    teleport="body"
                    calendar-cell-class="w-10 h-10 flex items-center justify-center rounded-lg text-base font-medium hover:bg-blue-50 transition-colors"
                    calendar-cell-selected-class="bg-blue-500 text-white font-bold"
                    calendar-cell-today-class="border border-blue-400"
                    calendar-header-class="text-gray-500 font-semibold text-base"
                    time-column-class="bg-white"
                    time-cell-class="w-12 h-10 flex items-center justify-center rounded-lg text-base font-medium hover:bg-blue-50 transition-colors"
                    time-cell-selected-class="bg-blue-500 text-white font-bold"
                    ok-button-class="bg-blue-500 text-white rounded px-4 py-1 ml-2 hover:bg-blue-600 transition-colors"
                />
                <span class="absolute right-2 text-gray-400 pointer-events-none">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/></svg>
                </span>
            </div>
        </td>

        <!-- Duration -->
        <td class="px-3 py-2">
            <span class="block w-full px-2 py-1 rounded text-sm border border-gray-300 bg-gray-50 text-gray-700 shadow-sm">
                {{ row.duration || '--:--' }}
            </span>
        </td>

        <!-- % -->
        <td class="px-3 py-2">
            <CustomDropdown v-model="row.percentage" @change="emitUpdateDeductions" />
        </td>

        <!-- To Date & Time -->
        <td class="px-3 py-2 text-sm">
            {{ formatDateTime(row.toDate) }}
        </td>

        <!-- Remarks -->
        <td class="px-3 py-2">
            <input type="text" v-model="row.remarks" placeholder="Enter remarks..."
                class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
        </td>

        <!-- Deductions -->
        <td class="px-3 py-2 text-sm">
            {{ row.deductions }}
        </td>

        <!-- Actions -->
        <td class="px-3 py-2">
            <div class="flex items-center space-x-2">
                <button v-if="isOutOfOrder" @click="$emit('auto-adjust', index)"
                    class="bg-yellow-500 hover:bg-yellow-600 text-white px-2 py-1 rounded text-xs" title="Auto Adjust">
                    ⚡
                </button>
                <button @click="$emit('clone', index)" class="text-white px-2 py-1 rounded text-xs cursor-pointer"
                    title="Clone">
                    <img class="size-4" src="../../icons/copy.svg" alt="clone">
                </button>
                <button @click="$emit('confirm-delete', index)"
                    class=" text-white px-2 py-1 rounded text-xs cursor-pointer" title="Delete">
                    <img class="size-4" src="../../icons/trash.svg" alt="delete">
                </button>
            </div>
        </td>
    </tr>
</template>

<script setup>
import { toRefs } from 'vue'
import ActivityTypeSelect from './ActivityTypeSelect.vue'
import CustomDropdown from './CustomDropdown .vue'
import Datepicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'

const props = defineProps({
    row: Object,
    index: Number,
    isOutOfOrder: Boolean,
    activityTypes: Array
})

const emit = defineEmits(['clone', 'auto-adjust', 'confirm-delete', 'update-row', 'update-deductions'])

const emitUpdate = () => {
    emit('update-row', props.index)
}

const emitUpdateDeductions = () => {
    emit('update-deductions', props.index)
}

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

const onDateTimeChange = (val) => {
    // val is a Date object or string depending on the picker
    if (!val) {
        props.row.duration = ''
        emitUpdate()
        return
    }
    const date = new Date(val)
    // Set duration as HH:mm:ss
    const hh = String(date.getHours()).padStart(2, '0')
    const mm = String(date.getMinutes()).padStart(2, '0')
    const ss = String(date.getSeconds()).padStart(2, '0')
    props.row.duration = `${hh}:${mm}:${ss}`
    emitUpdate()
}
</script>

<style scoped>
::v-deep(.dp__menu) {
  border-radius: 0.75rem;
  box-shadow: 0 10px 32px 0 rgba(0,0,0,0.10);
  border: 1px solid #e5e7eb;
  padding: 0.5rem 0.75rem;
}
::v-deep(.dp__calendar_header) {
  font-size: 1rem;
  font-weight: 600;
  color: #6b7280;
}
::v-deep(.dp__cell_inner) {
  border-radius: 0.5rem;
  font-size: 1rem;
  font-weight: 500;
  transition: background 0.2s;
}
::v-deep(.dp__cell_inner.dp__active) {
  background: #3b82f6;
  color: #fff;
  font-weight: 700;
}
::v-deep(.dp__cell_inner.dp__today) {
  border: 1.5px solid #3b82f6;
}
::v-deep(.dp__time_col) {
  background: #fff;
}
::v-deep(.dp__time_input) {
  border-radius: 0.5rem;
  font-size: 1rem;
  font-weight: 500;
}
::v-deep(.dp__button.dp__action_ok) {
  background: #3b82f6;
  color: #fff;
  border-radius: 0.5rem;
  padding: 0.25rem 1.25rem;
  margin-left: 0.5rem;
  transition: background 0.2s;
}
::v-deep(.dp__button.dp__action_ok:hover) {
  background: #2563eb;
}
</style>
