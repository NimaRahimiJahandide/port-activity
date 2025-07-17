<template>
  <select
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
    class="w-full px-2 py-1 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
  >
    <option value="">Select...</option>
    <option v-for="type in options" :key="type" :value="type">
      {{ type }}
    </option>
  </select>
</template>

<script setup>
defineProps({
  modelValue: String,
  options: {
    type: Array,
    required: true
  }
})

defineEmits(['update:modelValue'])
</script>
