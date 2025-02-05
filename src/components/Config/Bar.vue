<template>
  <div>
    <div>
      <label class="mr-2 text-sm font-medium uppercase">Table</label>
      <Select v-model="modelValue.table" :options="tableList"></Select>
    </div>
    <div v-if="columnList.length">
      <label class="mr-2 text-sm font-medium uppercase">Column</label>
      <Select v-model="modelValue.column" :options="columnList"></Select>
    </div>
    <div>
      <label class="mr-2 text-sm font-medium uppercase">Measure</label>
      <Select v-model="modelValue.measure" :options="measureList"></Select>
    </div>
  </div>
</template>

<script lang="ts">
  import { toRefs } from '@vueuse/core'
  import { defineComponent, computed, ref, watch, PropType } from 'vue'
  import { Column } from '../../interface'
  import { state, supabaseClientState } from '../../store'
  import * as d3 from 'd3'

  interface Config {
    table: string
    column: string
    measure: string
    data: number[]
    xLabel: string[]
  }
  export default defineComponent({
    props: {
      modelValue: {
        type: Object as PropType<Config>,
        required: true,
      },
    },
    emits: ['update:modelValue'],
    setup(prop, { emit }) {
      const { modelValue } = toRefs(prop)
      const temp = {
        ...{
          table: '',
          column: '',
          measure: 'count',
          data: [],
          xLabel: [],
        },
        ...modelValue.value,
      }
      emit('update:modelValue', temp)

      // measures
      const tableList = computed(() => Object.keys(state.tables))
      const columnList = computed(() => {
        if (!modelValue.value.table) {
          return []
        } else {
          modelValue.value.column = modelValue.value.column
            ? modelValue.value.column
            : '*'
          const tbl = state.tables[modelValue.value.table].columns as Column[]

          return tbl
            .filter((a) => a.format != 'uuid')
            .filter((a) => a.type == 'string' || a.type == 'boolean')
            .map((a) => a.title)
        }
      })
      const measureList = computed(() => {
        if (columnList.value.length) {
          return ['max', 'min', 'count', 'average', 'sum']
        } else {
          return ['count']
        }
      })

      // labels
      const calculated = computed(() => {
        return d3.rollup(
          dataJSON.value,
          (v) => v.length,
          (d) => d[modelValue.value.column]
        )
        // switch (modelValue.value.measure) {
        //   case 'count':
        //     return dataJSON.value.length
        //     break
        //   case 'min':
        //     return d3.min(dataJSON.value.map((a) => a[modelValue.value.column]))
        //     break
        //   case 'max':
        //     return d3.max(dataJSON.value.map((a) => a[modelValue.value.column]))
        //     break
        //   case 'average':
        //     return d3.mean(
        //       dataJSON.value.map((a) => a[modelValue.value.column])
        //     )
        //     break
        //   case 'sum':
        //     return d3.sum(dataJSON.value.map((a) => a[modelValue.value.column]))
        //     break
        //   default:
        //     return 0
        // }
      })

      // fetch Supabase Data
      const dataJSON = ref([])
      const fetchData = async () => {
        if (modelValue.value.table) {
          const { data, error } = await supabaseClientState
            .supabase()
            .from(modelValue.value.table)
            .select('*')
          dataJSON.value = data as []
        }
      }

      watch(
        () => modelValue.value.table,
        () => {
          modelValue.value.measure = 'count'
          fetchData()
        }
      )

      watch(calculated, (n, o) => {
        console.log(n)
        if (n) {
          modelValue.value.data = [...n.values()]
          modelValue.value.xLabel = [...n.keys()]
        }
        // modelValue.value.data = n
      })
      return {
        tableList,
        columnList,
        measureList,
      }
    },
  })
</script>
