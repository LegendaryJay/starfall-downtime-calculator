<template>
  <q-card>
    <q-card-section class="bg-secondary text-white">
      <div class="">{{ item.label }}</div>
    </q-card-section>
    <q-separator />

    <q-list dense bordered separator>
      <q-item :disable="typeof item.dc == 'undefined'" clickable v-ripple>
        <q-item-section>
          <q-item-label overline>DC</q-item-label>
          {{ item.dc }}
          <q-popup-edit
            :cover="false"
            v-model="item.dc"
            title="Modify DC"
            buttons
            v-slot="scope"
          >
            <q-input type="number" v-model="scope.value" dense autofocus />
          </q-popup-edit>
        </q-item-section>
      </q-item>
      <q-item clickable v-ripple>
        <q-item-section>
          <q-item-label overline>Time</q-item-label>
          {{ item.time }}
          <q-popup-edit
            :cover="false"
            v-model="item.time"
            title="Modify Time"
            buttons
            v-slot="scope"
          >
            <q-input type="number" v-model="scope.value" dense autofocus />
          </q-popup-edit>
        </q-item-section>
        <q-item-section>
          <q-item-label overline>Unit of Time</q-item-label>
          {{ item.timeUnit }}s
          <q-popup-edit
            :cover="false"
            v-model="item.timeUnit"
            title="Modify Unit of time"
            buttons
            v-slot="scope"
          >
            <q-select
              v-model="scope.value"
              :options="['Day', 'Week']"
              label="Standard"
            />
          </q-popup-edit>
        </q-item-section>
      </q-item>
      <q-item clickable v-ripple>
        <q-item-section>
          <q-item-label overline>
            Gold per
            {{ item.timeUnit }}
          </q-item-label>
          {{ item.daily }}
          <q-popup-edit
            v-model="item.daily"
            :title="'Modify Gold Per ' + item.timeUnit"
            buttons
            :cover="false"
            v-slot="scope"
          >
            <q-input type="number" v-model="scope.value" dense autofocus />
          </q-popup-edit>
        </q-item-section>
      </q-item>
      <q-item>
        <q-item-section>
          <q-item-label overline>Total Base Gold</q-item-label>
          {{ gold }}
        </q-item-section>
      </q-item>
    </q-list>
  </q-card>
</template>

<script setup>
import { ref, watch, computed } from "vue";

const props = defineProps({
  modelValue: Object,
});
const emit = defineEmits(["update:modelValue"]);

const item = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

const gold = computed(() => item.value.daily * item.value.time);

watch(
  () => props.modelValue,
  (newVal) => {
    item.value = { ...newVal };
  },
  { deep: true }
);
</script>
