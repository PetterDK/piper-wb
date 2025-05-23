<script setup>
import FuelComponent from './components/FuelComponent.vue';
import PerformanceComponent from './components/PerformanceComponent.vue';
import WeightBalanceComponent from './components/WeightBalanceComponent.vue';
import { darkTheme, NConfigProvider, NFlex } from 'naive-ui';

</script>

<template>
  <n-config-provider :theme="darkTheme">
    <header>
      <h1>OY-CGV beregninger</h1>
    </header>
    <main>
      <n-flex>
        <FuelComponent />
        <WeightBalanceComponent />
        <PerformanceComponent />
      </n-flex>
    </main>
  </n-config-provider>
</template>
