<template>
  <div class="q-pa-md">
    <div class="q-gutter-md">
      <CalculatorCard @update-averages="handleAveragesUpdate"></CalculatorCard>
      <TrialResultsCard :trial-results="averages"></TrialResultsCard>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import CalculatorCard from "./CalculatorCard.vue";
import TrialResultsCard from "./TrialResultsCard.vue";

const averages = ref({});

const handleAveragesUpdate = (newAverages) => {
  averages.value = newAverages;
};
</script>
