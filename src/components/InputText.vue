<template>
  <div class="mb-2">
    <input
      :type="type"
      :name="name"
      :placeholder="placeholder"
      :value="modelValue"
      @input="inputValue"
      class="w-full bg-dark-900 text-white-700 placeholder-dark-500 rounded-md px-4 py-2 h-10 flex-grow border-dark-border focus:border-green-500 focus:ring-green-500"
    />
  </div>
</template>
<script lang="ts">
  import { defineComponent } from 'vue'

  export default defineComponent({
    props: {
      type: {
        type: String,
        default: 'text',
      },
      name: {
        type: String,
        default: '',
      },
      placeholder: {
        type: String,
        default: 'placeholder',
      },
      modelValue: {
        type: String,
        required: true,
        default: '',
      },
    },
    emits: ['update:modelValue'],
    setup(prop, { emit }) {
      const inputValue = (e: Event) => {
        emit('update:modelValue', (e.target as HTMLInputElement).value)
      }
      return {
        inputValue,
      }
    },
  })
</script>
