<template>
  <CardSection label="Trial Results" v-if="hasResults">
    <q-card-section>
      <div class="q-pr-md row q-gutter-md row">
        <InfoCard label="Time" class="col-12 col-sm" v-if="hasTime">
          <q-card-section>
            <ResultItem
              label="Base"
              :content="trialResults.time.originalRequired"
              :format="formatNumber"
              :unitLabel="trialResults.time.unit + 's'"
            />
            <ResultItem
              label="Avg"
              :content="trialResults.time.average"
              :format="formatNumber"
              :unitLabel="trialResults.time.unit + 's'"
            />
            <ResultItem
              label="Min"
              :content="trialResults.time.min"
              :format="formatNumber"
              :unitLabel="trialResults.time.unit + 's'"
            />
            <ResultItem
              label="Max"
              :content="trialResults.time.max"
              :format="formatNumber"
              :unitLabel="trialResults.time.unit + 's'"
            />
            <ResultItem
              label="Successful Trials"
              :content="trialResults.time.trialSuccessRate"
              :format="formatToPercent"
            />
          </q-card-section>
        </InfoCard>
        <InfoCard label="Cost" class="col-12 col-sm" v-if="hasCost">
          <q-card-section>
            <ResultItem
              label="Base"
              :content="trialResults.cost.originalRequired"
              :format="formatNumber"
              unitLabel="Gold"
            />

            <ResultItem
              label="Min"
              :content="trialResults.cost.min"
              :format="formatNumber"
              unitLabel="Gold"
            />
            <ResultItem
              label="Max"
              :content="trialResults.cost.max"
              :format="formatNumber"
              unitLabel="Gold"
            />
          </q-card-section>
        </InfoCard>
        <InfoCard label="Rolls" class="col-12 col-sm" v-if="hasRolls">
          <q-card-section>
            <ResultItem label="DC" :content="trialResults.rolls.dcRequired" />
            <ResultItem
              label="Success Chance"
              :content="trialResults.rolls.successChance"
              :format="formatToPercent"
            />
            <ResultItem
              label="Avg Skill Roll"
              :content="trialResults.rolls.skill"
              :format="formatAndAdd(trialResults.rolls.skillBonus)"
            />
            <ResultItem
              label="Avg Tool Roll"
              :content="trialResults.rolls.tool"
              :format="formatAndAdd(trialResults.rolls.toolBonus)"
            />
          </q-card-section>
        </InfoCard>
      </div>

      <trialResultGraphs v-if="hasData" :trialResults="trialResults" />
    </q-card-section>
  </CardSection>
</template>

<script setup>
import { ref, watch, computed, toRefs } from "vue";

import CardSection from "./CardSection.vue";
import ResultItem from "./ResultItem.vue";
import InfoCard from "./InfoCard.vue";
import trialResultGraphs from "./trialResultGraphs.vue";

const props = defineProps({
  trialResults: {
    type: Object,
    default: () => ({}),
  },
});

const { trialResults } = toRefs(props);

function formatNumber(value) {
  return parseFloat(value.toFixed(2));
}

function formatToPercent(number) {
  return `${formatNumber(number * 100)}%`;
}

function formatAndAdd(bonus) {
  return (number) => formatNumber(number + bonus);
}

const hasTime = computed(
  () =>
    !!trialResults.value.time &&
    ["average", "min", "max", "originalRequired"].some(
      (key) => key in trialResults.value.time
    )
);
const hasCost = computed(
  () =>
    !!trialResults.value.cost &&
    ["average", "min", "max", "originalRequired"].some(
      (key) => key in trialResults.value.cost
    )
);
const hasRolls = computed(
  () =>
    !!trialResults.value.rolls &&
    [
      "dcRequired",
      "successChance",
      "skill",
      "tool",
      "skillBonus",
      "toolBonus",
    ].some((key) => key in trialResults.value.rolls)
);
const hasData = computed(
  () =>
    !!trialResults.value.time &&
    typeof trialResults.value.time.timeData !== "undefined"
);
const hasResults = computed(
  () => hasTime.value || hasCost.value || hasRolls.value
);
</script>
