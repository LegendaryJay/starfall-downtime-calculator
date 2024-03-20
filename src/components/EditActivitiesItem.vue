<template>
  <info-card :label="item.label">
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
          {{ item.costPerTime }}
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
      <q-item clickable v-ripple>
        <q-item-section>
          <q-item-label overline> Total Base Gold </q-item-label>
          {{ item.cost }}
          <q-popup-edit
            v-model="item.cost"
            :title="'Modify Base Price'"
            buttons
            :cover="false"
            v-slot="scope"
          >
            <q-input type="number" v-model="scope.value" dense autofocus />
          </q-popup-edit>
        </q-item-section>
      </q-item>
    </q-list>
  </info-card>
</template>

<script setup>
import { ref, watch, computed } from "vue";
import InfoCard from "./InfoCard.vue";

const props = defineProps({
  modelValue: Object,
});
const emit = defineEmits(["update:modelValue"]);

const item = computed({
  get: () => props.modelValue,
  set: (value) => emit("update:modelValue", value),
});

watch(
  () => props.modelValue,
  (newVal) => {
    item.value = { ...newVal };
  },
  { deep: true }
);
</script>
