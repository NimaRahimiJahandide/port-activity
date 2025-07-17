<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white p-6 rounded-lg shadow-lg max-w-md w-full mx-4">
      <h3 class="text-lg font-semibold mb-4">Confirm Delete</h3>
      <p class="text-gray-600 mb-6">
        Are you sure you want to delete this row? This action cannot be undone.
      </p>
      <div class="flex justify-between">
        <button
          @click="$emit('cancel')"
          class="px-4 py-2 bg-gray-300 hover:bg-gray-400 text-gray-800 rounded transition-colors"
        >
          Cancel
        </button>
        <button
          @click="$emit('confirm')"
          class="px-4 py-2 bg-red-500 hover:bg-red-600 text-white rounded transition-colors"
        >
          OK
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineEmits(['cancel', 'confirm'])
</script>
