<template>
  <div ref="target" class="mb-2">
    <button
      class="w-full text-white-700 bg-dark-900 mr-2 rounded-md px-4 py-2 h-10 border border-dark-border ring ring-transparent focus:ring-green-500 focus:outline-none flex justify-between relative"
      @click.prevent="isOpen = !isOpen"
    >
      <span>{{ modelValue }} </span>
      <i-entypo:select-arrows class="mt-0.5 p-2px"></i-entypo:select-arrows>
    </button>
    <ul
      v-if="isOpen"
      class="absolute mt-1 w-56 bg-dark-900 rounded-md overflow-hidden border-dark-border border z-50"
    >
      <li
        class="py-1 px-4 items-center text-white-800 border-l-2 border-transparent hover:bg-dark-600 hover:text-white cursor-pointer"
        v-for="option in options"
        :key="option"
        @click="selectAction(option)"
      >
        {{ option }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
  import { defineComponent, PropType, ref } from 'vue'
  import { onClickOutside } from '@vueuse/core'

  export default defineComponent({
    props: {
      modelValue: String || Number,
      options: Object as PropType<string[]>,
    },
    emits: ['update:modelValue'],
    setup(prop, { emit }) {
      const isOpen = ref(false)
      const target = ref(null)
      const targetParent = ref(null)
      const selectAction = (title: string) => {
        isOpen.value = false
        emit('update:modelValue', title)
      }
      onClickOutside(target, (event) => {
        isOpen.value = false
        // targetParent.value == event.srcElement ? '' : (isOpen.value = false)
      })

      return {
        isOpen,
        target,
        targetParent,
        selectAction,
      }
    },
  })
</script>

<style></style>
