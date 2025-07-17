<template>
    <tr class="border-b border-[#f3f3f3] " :class="{ 'bg-red-100': isOutOfOrder }">
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
            <CustomDatepicker
                v-model="row.fromDate"
                :enable-time-picker="true"
                :format="'dd/MM/yyyy'"
                @update:model-value="onDateTimeChange"
                input-class="w-full py-1 rounded text-base focus:outline-none focus:border-none pr-10"
                placeholder="Select date and time..."
            />
            <span class="block font-bold text-xs text-gray-400">
                {{ formatWeekday(row.fromDate) }}, {{ formatTimeOnly(row.fromDate) }}
            </span>
        </td>

        <!-- Duration -->
        <td class="px-3 py-2">
            <span class="block w-full px-2 py-1 rounded text-sm text-gray-700">
                {{ formatDuration(row.duration) }}
            </span>
        </td>

        <!-- % -->
        <td class="px-3 py-2">
            <CustomDropdown v-model="row.percentage" @change="emitUpdateDeductions" />
        </td>

        <!-- To Date & Time -->
        <td class="px-3 py-2 text-sm">
            <span v-html="formatDateTimeMultiline(row.toDate)"></span>
        </td>

        <!-- Remarks -->
        <td class="px-3 py-2">
            <input type="text" v-model="row.remarks" placeholder="Enter remarks..."
                class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
        </td>

        <!-- Deductions -->
        <td class="px-3 py-2 text-sm">
            {{ formatDuration(row.deductions) }}
        </td>

        <!-- Actions -->
        <td class="px-3 py-2">
            <div class="flex items-center justify-end space-x-2">
                <button v-if="isOutOfOrder" @click="$emit('auto-adjust', index)"
                    class="bg-yellow-500 hover:bg-yellow-600 text-white px-2 py-1 rounded text-xs cursor-pointer" title="Auto Adjust">
                    ✅
                </button>
                <!-- Only show Clone for non-first rows -->
                <button v-if="index > 0" @click="$emit('clone', index)" class="text-white px-2 py-1 rounded text-xs cursor-pointer"
                    title="Clone">
                    <img class="size-4" src="../../icons/copy.svg" alt="clone">
                </button>
                <button @click="$emit('confirm-delete', index)"
                    class=" text-white px-2 py-1 rounded text-xs cursor-pointer" title="Delete">
                    <img class="size-5" src="../../icons/trash.svg" alt="delete">
                </button>
            </div>
        </td>
    </tr>
</template>

<script setup>
import { toRefs } from 'vue'
import ActivityTypeSelect from './ActivityTypeSelect.vue'
import CustomDropdown from './CustomDropdown .vue'
import CustomDatepicker from './CustomDatepicker.vue'

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
    // Format: dd/MM/yyyy, HH:mm:ss (24-hour, no AM/PM)
    const day = String(date.getDate()).padStart(2, '0')
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const year = date.getFullYear()
    const hours = String(date.getHours()).padStart(2, '0')
    const minutes = String(date.getMinutes()).padStart(2, '0')
    const seconds = String(date.getSeconds()).padStart(2, '0')
    return `${day}/${month}/${year}, ${hours}:${minutes}:${seconds}`
}

const formatDateTimeMultiline = (dateTimeString) => {
    if (!dateTimeString) return ''
    const date = new Date(dateTimeString)
    const day = String(date.getDate()).padStart(2, '0')
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const year = date.getFullYear()
    const hours = String(date.getHours()).padStart(2, '0')
    const minutes = String(date.getMinutes()).padStart(2, '0')
    const seconds = String(date.getSeconds()).padStart(2, '0')
    return `${day}/${month}/${year}<br>${hours}:${minutes}:${seconds}`
}

const formatDay = (dateTimeString) => {
    if (!dateTimeString) return ''
    const date = new Date(dateTimeString)
    return date.toLocaleDateString('en-US', { weekday: 'short' })
}

const onDateTimeChange = (val) => {
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

const formatTimeOnly = (dateTimeString) => {
    if (!dateTimeString) return ''
    const date = new Date(dateTimeString)
    return date.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit'})
}

const formatWeekday = (dateTimeString) => {
    if (!dateTimeString) return ''
    const date = new Date(dateTimeString)
    return date.toLocaleDateString('en-US', { weekday: 'short' })
}

const formatDuration = (durationStr) => {
    if (!durationStr) return '--:--'
    // Accepts duration as HH:mm:ss or HH:mm
    const parts = durationStr.split(':').map(Number)
    let hours = 0, minutes = 0
    if (parts.length === 3) {
        hours = parts[0]
        minutes = parts[1]
    } else if (parts.length === 2) {
        hours = parts[0]
        minutes = parts[1]
    } else {
        return durationStr
    }
    const days = Math.floor(hours / 24)
    const remHours = hours % 24
    return `${days.toString().padStart(2, '0')}d ${remHours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`
}
</script>

<style scoped>
/* Custom styles for the row can be added here if needed */
</style>