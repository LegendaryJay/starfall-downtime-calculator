<template>
  <q-card>
    <q-card-section class="bg-primary text-white">
      <div class="text-h6">Details</div>
    </q-card-section>
    <q-separator></q-separator>
    <div class="q-pa-md q-gutter-md row">
      <div
        v-for="(activity, index) in activities"
        :key="index"
        class="col-auto"
      >
        <EditActivitiesItem
          :modelValue="activity"
          @update:modelValue="(newValue) => updateActivity(index, newValue)"
        />
      </div>
    </div>
  </q-card>
</template>

<script setup>
import { ref } from "vue";
import EditActivitiesItem from "./EditActivitiesItem.vue";
</script>
