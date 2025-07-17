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
            <input type="datetime-local" v-model="row.fromDate" @change="emitUpdate"
                class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
        </td>

        <!-- Duration -->
        <td class="px-3 py-2">
            <input type="time" v-model="row.duration" @change="emitUpdate"
                class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
        </td>

        <!-- % -->
        <td class="px-3 py-2">
            <select v-model="row.percentage" @change="emitUpdateDeductions"
                class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
                <option value="0">0%</option>
                <option value="50">50%</option>
                <option value="100">100%</option>
            </select>
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
                <button @click="$emit('clone', index)"
                    class="text-white px-2 py-1 rounded text-xs cursor-pointer" title="Clone">
                    <img class="size-4" src="../../icons/copy.svg" alt="clone">
                </button>
                <button @click="$emit('confirm-delete', index)"
                    class=" text-white px-2 py-1 rounded text-xs cursor-pointer" title="Delete">
                    <img class="size-4"src="../../icons/trash.svg" alt="delete">
                </button>
            </div>
        </td>
    </tr>
</template>

<script setup>
import { toRefs } from 'vue'
import ActivityTypeSelect from './ActivityTypeSelect.vue'

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
</script>
