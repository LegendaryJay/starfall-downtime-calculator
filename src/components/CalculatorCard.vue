<template>
  <CardSection label="Calculator">
    <div>
      <q-card-section>
        <q-btn-toggle
          v-model="calcMode"
          push
          toggle-color="primary"
          :options="[
            { label: 'Craft Magic Item', value: 'craftMagicItem' },
            { label: 'Learn Language/Tool', value: 'learnLanguageTool' },
            { label: 'Learn Skill', value: 'learnSkill' },
          ]"
        />
      </q-card-section>
      <q-card-section class="row">
        <div class="col-12 col-sm-6">
          <q-select
            v-model="itemRarity"
            :options="rarityOptions"
            outlined
            v-if="calcMode === 'craftMagicItem'"
          />
          <q-btn-toggle
            v-model="isExpertise"
            class="q-mt-md"
            push
            toggle-color="primary"
            color="white"
            text-color="primary"
            :options="[
              { label: 'proficiency', value: false },
              { label: 'Expertise', value: true },
            ]"
            v-if="calcMode === 'learnSkill'"
          />
          <q-btn-toggle
            v-model="isConsumable"
            class="q-mt-md"
            push
            toggle-color="primary"
            color="white"
            text-color="primary"
            :options="[
              { label: 'Item', value: false },
              { label: 'Consumable', value: true },
            ]"
            v-if="calcMode === 'craftMagicItem'"
          />
          <q-input
            v-model.number="intMod"
            label="Intelligence Modifier"
            type="number"
            class="q-mt-md"
          />
          <q-input
            v-model.number="skillMod"
            label="Skill Check Bonus"
            type="number"
            class="q-mt-md"
            v-if="calcMode !== 'learnLanguageTool'"
          />
          <q-input
            v-model.number="toolMod"
            label="Tool Check Bonus (Skill Modifier + Proficiency)"
            type="number"
            class="q-mt-md"
            v-if="calcMode === 'craftMagicItem'"
          />
        </div>
        <div class="col-12 col-sm-6 q-pa-md">
          <EditActivitiesItem
            v-model:model-value="currentActivity"
          ></EditActivitiesItem>
        </div>
      </q-card-section>
      <q-card-section class="row">
        <div class="col-12 col-sm-2">
          <q-input
            dense
            filled
            v-model.number="trialCountLimit"
            label="Trials"
            type="number"
          />
        </div>
        <div class="col-12 col-sm-10">
          <q-btn class="bg-primary fit text-white" @click="calculate">
            Calculate
          </q-btn>
        </div>
      </q-card-section>
    </div>
  </CardSection>
</template>

<script setup>
import { ref, computed } from "vue";
import TrialResultsCard from "./TrialResultsCard.vue";
import EditActivitiesItem from "./EditActivitiesItem.vue";
import { activities } from "app/data/data";
import CardSection from "./CardSection.vue";

const emit = defineEmits(["updateAverages"]);

const trialCountLimit = ref(100000);

const averages = ref({});

const rarityOptions = activities.value.slice(0, 4);

const itemRarity = ref(rarityOptions[0]); // Default value can be adjusted
const isExpertise = ref(false);
const isConsumable = ref(false);
const intMod = ref(0);
const skillMod = ref(0);
const toolMod = ref(0);
const calcMode = ref("craftMagicItem");

const currentActivity = computed(() => {
  if (calcMode.value == "craftMagicItem") {
    return itemRarity.value;
  }

  if (calcMode.value == "learnLanguageTool") {
    return activities.value[4];
  }

  if (calcMode.value == "learnSkill") {
    if (isExpertise.value) {
      return activities.value[6];
    }
    return activities.value[5];
  }
  return undefined;
});

function roll() {
  return Math.floor(Math.random() * 20) + 1;
}

const initMagicItemTrials = function () {
  let baseTime = Math.max(currentActivity.value.time - intMod.value, 1);
  let skillBonus = skillMod.value;
  let toolBonus = toolMod.value;
  let dc = currentActivity.value.dc;
  let cost = isConsumable.value
    ? currentActivity.value.cost * 0.5
    : currentActivity.value.cost;
  let costOffsetValue = cost / baseTime;
  let timeUnit = currentActivity.value.timeUnit;

  let returnItem = {
    time: {
      unit: timeUnit,
      originalRequired: baseTime,
    },
    cost: {
      originalRequired: cost,
    },
    rolls: {
      dcRequired: dc,
      skillBonus,
      toolBonus,
    },
  };

  return runTrials(
    magicItemTime,
    baseTime,
    skillBonus,
    toolBonus,
    dc,
    cost,
    costOffsetValue,
    returnItem
  );
};
const initSkillTrials = function () {
  let baseTime = currentActivity.value.time - intMod.value;
  let skillBonus = skillMod.value;
  let toolBonus = null;
  let dc = currentActivity.value.dc;
  let cost = 0;
  let costOffsetValue = currentActivity.value.costPerTime;
  let timeUnit = currentActivity.value.timeUnit;

  let returnItem = {
    time: {
      unit: timeUnit,
      originalRequired: baseTime,
    },
    cost: {
      originalRequired: currentActivity.value.cost,
    },
    rolls: {
      dcRequired: dc,
      skillBonus,
      toolBonus: null,
    },
  };

  return runTrials(
    learnSkillTime,
    baseTime,
    skillBonus,
    toolBonus,
    dc,
    cost,
    costOffsetValue,
    returnItem
  );
};

let magicItemTime = function (skillBonus, toolBonus, dc) {
  let timeSuccessCount = 0;
  let rollSuccessCount = 0;
  let costOffsetCount = 0;

  let toolRoll = roll();
  let skillRoll = roll();

  let nat1 = toolRoll == 1 || skillRoll == 1;
  let nat20 = toolRoll == 20 || skillRoll == 20;

  if (nat1) {
    timeSuccessCount--;
    costOffsetCount++;
  }

  if (nat20) {
    timeSuccessCount++;
  }

  if (toolRoll + toolBonus >= dc && skillRoll + skillBonus >= dc) {
    timeSuccessCount++;
    rollSuccessCount++;
  }

  return {
    timeSuccessCount,
    rollCount: 1,
    toolRollTotal: toolRoll,
    skillRollTotal: skillRoll,
    rollSuccessCount,
    costOffsetCount,
  };
};
function learnSkillTime(skillBonus, toolBonus, dc) {
  let rollTotal = 0;
  let rollSuccessCount = 0;
  let timeSuccessCount = 0;
  let costOffsetCount = 1;
  let day;
  for (day = 0; day < 5; day++) {
    let rawRoll = roll();
    let success = rawRoll + skillBonus >= dc;

    rollTotal += rawRoll;
    rollSuccessCount += success ? 1 : 0;

    if (rollSuccessCount >= 3) {
      timeSuccessCount = 1;
      costOffsetCount = 0;
      break;
    } else if (day - rollSuccessCount >= 3) {
      break;
    }
  }
  return {
    timeSuccessCount,
    rollCount: day,
    toolRollTotal: null,
    skillRollTotal: rollTotal,
    rollSuccessCount,
    costOffsetCount,
  };
}

function calculateStatistics(data) {
  const sum = data.reduce((a, b) => a + b, 0);
  const min = Math.min(...data);
  const max = Math.max(...data);
  return {
    avg: sum / data.length,
    min,
    max,
  };
}

function runSingleTrial(
  timeRunthrough,
  baseTime,
  skillBonus,
  toolBonus,
  dc,
  cost,
  costOffsetValue
) {
  let timeSuccessCount = 0;
  let timeForThisTrial = 0;
  let offsetCostForThisTrial = 0;

  let skillRollTotal = 0;
  let toolRollTotal = 0;
  let attemptedRollCount = 0;
  let successfulRollCount = 0;

  while (timeSuccessCount < baseTime && timeForThisTrial < 999) {
    let thisTimeData = timeRunthrough(skillBonus, toolBonus, dc);

    timeSuccessCount += thisTimeData.timeSuccessCount;
    skillRollTotal += thisTimeData.skillRollTotal;
    toolRollTotal += thisTimeData.toolRollTotal;
    attemptedRollCount += thisTimeData.rollCount;
    successfulRollCount += thisTimeData.rollSuccessCount;
    offsetCostForThisTrial += thisTimeData.costOffsetCount * costOffsetValue;

    timeForThisTrial++;
  }

  return {
    timeForThisTrial,
    trialCost: cost + offsetCostForThisTrial,
    skillRollTotal,
    toolRollTotal,
    attemptedRollCount,
    successfulRollCount,
  };
}

function accumulateTrialResults(trialStats, trialResults) {
  trialStats.trialCount++;
  trialStats.timeForThisTrial = trialResults.timeForThisTrial;
  trialStats.skillRollTotal += trialResults.skillRollTotal;
  trialStats.toolRollTotal += trialResults.toolRollTotal;
  trialStats.attemptedRollCount += trialResults.attemptedRollCount;
  trialStats.successfulRollCount += trialResults.successfulRollCount;

  trialStats.successfulTrialCount +=
    trialResults.timeForThisTrial < 999 ? 1 : 0;

  trialStats.timeData.push(trialResults.timeForThisTrial);
  trialStats.costData.push(trialResults.trialCost);

  return trialStats;
}
function calculateFinalResults(trialStats, returnItem) {
  return {
    time: {
      ...returnItem.time,
      ...calculateStatistics(trialStats.timeData),
      trialSuccessRate: trialStats.successfulTrialCount / trialStats.trialCount,
      timeData: trialStats.timeData,
    },
    cost: {
      ...returnItem.cost,
      ...calculateStatistics(trialStats.costData),
      costData: trialStats.costData,
    },
    rolls: {
      ...returnItem.rolls,
      skill: trialStats.skillRollTotal / trialStats.attemptedRollCount,
      tool:
        returnItem.rolls.toolBonus === null
          ? null
          : trialStats.toolRollTotal / trialStats.attemptedRollCount,
      successChance:
        trialStats.successfulRollCount / trialStats.attemptedRollCount,
    },
  };
}

const runTrials = function (
  timeRunthrough,
  baseTime,
  skillBonus,
  toolBonus,
  dc,
  cost,
  costOffsetValue,
  returnItem
) {
  let trialStats = {
    trialCount: 0,
    successfulTrialCount: 0,
    skillRollTotal: 0,
    toolRollTotal: 0,
    attemptedRollCount: 0,
    successfulRollCount: 0,
    timeData: [],
    costData: [],
  };

  for (let trialIndex = 0; trialIndex < trialCountLimit.value; trialIndex++) {
    const trialResult = runSingleTrial(
      timeRunthrough,
      baseTime,
      skillBonus,
      toolBonus,
      dc,
      cost,
      costOffsetValue
    );

    trialStats = accumulateTrialResults(trialStats, trialResult);
  }
  let newReturnItem = calculateFinalResults(trialStats, returnItem);
  console.log(newReturnItem.rolls);
  return newReturnItem;
};

function langToolTrial() {
  let weeks = currentActivity.value.time - intMod.value;
  return {
    time: {
      unit: currentActivity.value.timeUnit,
      originalRequired: weeks,
    },
    cost: {
      originalRequired: currentActivity.value.costPerTime * weeks,
    },
  };
}

function calculate() {
  if (calcMode.value === "craftMagicItem") {
    averages.value = initMagicItemTrials();
  } else if (calcMode.value === "learnLanguageTool") {
    averages.value = langToolTrial();
  } else if (calcMode.value === "learnSkill") {
    averages.value = initSkillTrials();
  }
  emit("updateAverages", averages.value);
}
</script>
