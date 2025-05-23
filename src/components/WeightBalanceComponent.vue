<script setup lang="ts">
import { DataTableCreateSummary, NCard, NDataTable, NInputNumber } from 'naive-ui'
import { ref, computed, h } from 'vue';
import { weightBalance } from '@/assets/defaults.json'
import { DataTableColumn } from 'naive-ui';
import { formatNumber } from '@/helpers';

class Line {
  name: string
  weight?: number
  arm: number

  constructor(name: string, weight: number | undefined, arm: number) {
    this.name = name
    this.weight = weight
    this.arm = arm
  }

  public get torque() {
    return this.weight && this.arm ? this.weight * this.arm : undefined
  }
}

const lines = ref(weightBalance.map(x => new Line(x.name, x.weight, x.arm)))

const totalWeight = computed(() => lines.value.reduce((prev, current) => prev + (current.weight ?? 0), 0))
const totalTorque = computed(() => lines.value.reduce((prev, current) => prev + (current.torque ?? 0), 0))

const columns: DataTableColumn[] = [
  {
    title: '',
    key: 'name'
  },
  {
    title: 'Vægt',
    key: 'weight',
    align: 'right',
    render: (row) => h(NInputNumber, {
      value: row.weight,
      showButton: false,
      onUpdateValue: (value) => row.weight = value
    })
  },
  {
    title: 'Arm',
    key: 'arm',
    align: 'right',
    render: (row) => h(NInputNumber, {
      value: row.arm,
      showButton: false,
      onUpdateValue: (value) => row.arm = value
    })
  },
  {
    title: 'Moment',
    key: 'torque',
    align: 'right'
  }
]

const summary: DataTableCreateSummary = (data) => {
  return {
    name: {
      value: 'Total'
    },
    weight: {
      value: formatNumber(totalWeight.value)
    },
    arm: {
      value: formatNumber(totalTorque.value / totalWeight.value)
    },
    torque: {
      value: formatNumber(totalTorque.value),
    }
  }
}
</script>

<template>
  <n-card title="Vægt og balance">
    <n-data-table :columns="columns" :data="lines" :summary="summary"></n-data-table>
  </n-card>
</template>