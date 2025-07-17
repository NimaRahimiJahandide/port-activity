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
            <CustomDatepicker
                v-model="row.fromDate"
                :enable-time-picker="true"
                :format="'dd/MM/yyyy HH:mm:ss'"
                @update:model-value="onDateTimeChange"
                input-class="w-full px-2 py-1 rounded text-base border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500 shadow-sm pr-10"
                placeholder="Select date and time..."
            />
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
    return date.toLocaleString('en-US', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
    })
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
</script>

<style scoped>
/* Custom styles for the row can be added here if needed */
</style>