<template>
  <q-card>
    <q-card-section class="bg-secondary text-white">
      <div class="">{{ label }}</div>
    </q-card-section>
    <slot></slot>
  </q-card>
</template>

<script setup>
import { defineProps } from "vue";

const props = defineProps({
  label: String,
});
</script>
