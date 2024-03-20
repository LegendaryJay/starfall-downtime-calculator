<template>
  <q-card>
    <q-card-section class="bg-primary text-white">
      <div class="text-h6">{{ label }}</div>
    </q-card-section>
    <q-card-section>
      <slot></slot>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { defineProps } from "vue";

const props = defineProps({
  label: String,
});
</script>
