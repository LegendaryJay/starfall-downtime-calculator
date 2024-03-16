<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg" />
          </q-avatar>
          Starfall Downtime Calculator
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <CalculatorCard> </CalculatorCard>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import CalculatorCard from "./components/CalculatorCard.vue";

defineOptions({
  name: "App",
});
</script>
