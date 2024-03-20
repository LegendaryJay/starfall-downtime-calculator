<template>
  <div v-if="hasContent">
    <strong>{{ label }}:</strong>
    {{ fullContent }}
  </div>
</template>

<script setup>
import { computed, defineProps } from "vue";

const props = defineProps({
  label: {
    type: String,
    default: "",
  },
  content: {
    default: "",
  },
  format: {
    type: Function,
    default: (string) => string,
  },
  unitLabel: {
    type: String,
    default: "",
  },
});

const fullContent = computed(() =>
  hasContent.value ? `${props.format(props.content)} ${props.unitLabel}` : ""
);

const hasContent = computed(() => {
  const contentValue = props.content;
  return (
    contentValue !== null && contentValue !== undefined && contentValue !== ""
  );
});
</script>
