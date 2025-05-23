<script lang="ts" setup>
import { formatNumber } from '@/helpers';
import { NCard, NFlex, NForm, NFormItem, NInputNumber, NDivider, NCheckbox, NCheckboxGroup, NTable, NTag, NRadioGroup, NRadioButton } from 'naive-ui';
import { computed, ref, watch } from 'vue';

class Factor {
  name: string
  enabled: boolean
  multiplier: number
  value?: number
  maxValue?: number
  perUnit?: string

  constructor(name: string, multiplier: number, perUnit?: string, maxValue?: number) {
    this.name = name
    this.multiplier = multiplier
    this.perUnit = perUnit
    this.maxValue = maxValue
  }

  public get addedDistance(): number | undefined {
    if (this.perUnit) {
      return this.value ? Math.round(initialStartDistance.value * this.multiplier * this.value) : undefined
    } else {
      return Math.round(initialStartDistance.value * this.multiplier)
    }
  }
}

const initialStartDistance = ref<number>(1600)
const isShortGrass = ref(false)
const isLongGrass = ref(false)
const isWet = ref(false)
const isWetSnow = ref(false)
const isFrozenSnow = ref(false)
const unit = ref<'m' | 'ft'>('ft')

const factors = ref<Factor[]>([
  new Factor('Fast kort græs (5-10 cm)', 0.1),
  new Factor('Langt græs (10-20 cm)', 0.25),
  new Factor('Vand/sjap (max. 2,5 cm)', 0.2, 'cm', 2.5),
  new Factor('Våd sne (max. 5 cm)', 0.1, 'cm', 5),
  new Factor('Frossen sne (max. 10 cm)', 0.05, 'cm', 10)
])

const shortGrassDistance = computed(() => initialStartDistance.value * 0.1)
const calibratedStartDistance = computed(() => {
  let result = initialStartDistance.value

  const addedDistances = factors.value.filter(x => x.enabled).reduce((prev, current) => {
    return prev + (current.multiplier * initialStartDistance.value)
  }, 0)

  return result + addedDistances
})

watch(unit, (value) => {
  if (value === 'm') {
    console.log('changed to meters')
    initialStartDistance.value = Math.round(initialStartDistance.value / 3.2808399)
  }

  else if (value === 'ft') {
    initialStartDistance.value = Math.round(initialStartDistance.value * 3.2808399)
  }
})
</script>

<template>
  <n-card title="Start og landing">
    <n-flex>
      <n-form label-placement="left" label-width="auto">
        <n-form-item label="Enhed">
          <n-radio-group v-model:value="unit">
            <n-radio-button label="Meter" value="m" />
            <n-radio-button label="Fod" value="ft" />
          </n-radio-group>
        </n-form-item>
        <n-form-item label="Startdistance (til 50') jf. POH">
          <n-input-number v-model:value="initialStartDistance" />
        </n-form-item>
      </n-form>
      <n-table>
        <thead>
          <tr>
            <th></th>
            <th>Faktor</th>
            <th></th>
            <th style="text-align: right;">Distance</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="factor in factors" :key="factor.name">
            <td>
              <n-checkbox v-model:checked="factor.enabled" :label="factor.name"></n-checkbox>
            </td>
            <td>
              +{{ factor.multiplier * 100 }}%{{ factor.perUnit ? `/${factor.perUnit}` : '' }}
            </td>
            <td style="width: 300px">
              <n-input-number
                v-if="factor.perUnit"
                v-model:value="factor.value"
                :disabled="!factor.enabled"
                :show-button="false"
                :max="factor.maxValue"
              >
                <template #suffix>
                  {{ factor.perUnit }}
                </template>
              </n-input-number>
            </td>
            <td align="right">
              <div v-if="factor.enabled && factor.addedDistance">
                {{ Math.sign(factor.addedDistance) ? '+' : '-' }} {{ factor.addedDistance }}
              </div>
            </td>
          </tr>
        </tbody>
        <tfoot>
          <tr>
            <th colspan="3">Total</th>
            <th style="text-align: right;">
              {{ calibratedStartDistance }} fod
            </th>
          </tr>
        </tfoot>
      </n-table>
    </n-flex>
  </n-card>
</template>