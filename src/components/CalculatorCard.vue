<template>
  <div class="q-pa-md">
    <div class="q-gutter-md">
      <q-card>
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Calculator</div>
        </q-card-section>
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
              v-model.number="trialCount"
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
      </q-card>
      <TrialResultsCard :trialResults="averages" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import TrialResultsCard from "./TrialResultsCard.vue";
import EditActivitiesItem from "./EditActivitiesItem.vue";
import { activities } from "app/data/data";

const trialCount = ref(100000);

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

const gold = computed(
  () => currentActivity.value.daily * currentActivity.value.time
);

let magicDay = function (skillBonus, toolBonus, dc) {
  let successes = 0;
  let rollSuccesses = 0;

  let toolRoll = roll();
  let skillRoll = roll();

  let nat1 = toolRoll == 1 || skillRoll == 1;
  let nat20 = toolRoll == 20 || skillRoll == 20;

  if (nat1) {
    successes--;
  }

  if (nat20) {
    successes++;
  }

  if (toolRoll + toolBonus >= dc && skillRoll + skillBonus >= dc) {
    successes++;
    rollSuccesses++;
  }

  return {
    rollSuccesses,
    successes,
    toolRoll,
    skillRoll,
    costOffset: nat1 ? 1 : 0,
  };
};

// let magicTrial = function (skillBonus, toolBonus, dc, dailyCost) {
//   let rollCount = 0;
//   let skillRolls = 0;
//   let toolRolls = 0;
//   let rollSuccesses = 0;
//   let successes = 0;

//   let trailSuccess = false;

//   let trialTime = 0;

//   let costOffset = 0;

//   while (effectiveSuccesses < requiredSuccesses &&trialTime > 999 ) {
//     let dayData = magicDay(skillBonus, toolBonus, dc);
//     effectiveSuccesses += dayData.effectiveSuccesses;
//     costOffset += (dayData.nat1 ? 1 : 0) * dailyCost;

//     totalSkillRolls += dayData.skillRoll;
//     totalToolRolls += dayData.toolRoll;

//     trialTime++;
//   }
//   if (effectiveSuccesses >= requiredSuccesses) {
//     trailSuccess = true;
//   }
//   return {};
// };

let magicTrials = function () {
  let trials = trialCount.value;
  let requiredSuccesses = Math.max(
    currentActivity.value.time - intMod.value,
    1
  );
  let cost = isConsumable.value ? gold.value / 2 : gold.value;
  let dailyCost = cost / requiredSuccesses;
  let dc = currentActivity.value.dc;

  let totalTime = 0;
  let totalCost = 0;
  let totalSkillRolls = 0;
  let totalToolRolls = 0;

  let trialFalures = 0;
  let rollSuccesses = 0;

  let minTime = Infinity;
  let maxTime = 0;
  let minCost = Infinity;
  let maxCost = 0;

  let timeData = [];

  for (let trial = 0; trial < trials; trial++) {
    let successes = 0;
    let trialTime = 0;

    let trialCost = cost;

    while (successes < requiredSuccesses) {
      if (trialTime > 999) {
        trialFalures++;
        break;
      }

      let dayData = magicDay(skillMod.value, toolMod.value, dc);
      successes += dayData.successes;
      trialCost += dayData.costOffset * dailyCost;

      totalSkillRolls += dayData.skillRoll;
      totalToolRolls += dayData.toolRoll;
      rollSuccesses += dayData.rollSuccesses;

      totalTime++;
      trialTime++;
    }
    timeData.push(trialTime);
    minTime = Math.min(minTime, trialTime);
    minCost = Math.min(minCost, trialCost);
    maxTime = Math.max(maxTime, trialTime);
    maxCost = Math.max(maxCost, trialCost);

    totalCost += trialCost;
  }

  let rollSuccessChance = rollSuccesses / totalTime;
  let averageSkillRoll = totalSkillRolls / totalTime;
  let averageToolRoll = totalToolRolls / totalTime;

  let averageTime = totalTime / trials;
  let averageCost = totalCost / trials;

  return {
    time: {
      unit: currentActivity.value.timeUnit,
      originalRequired: requiredSuccesses,
      average: averageTime,
      min: minTime,
      max: maxTime,
      trialSuccessRate: 1 - trialFalures / trialCount.value,
      timeData,
    },
    cost: {
      originalRequired: cost,
      average: averageCost,
      min: minCost,
      max: maxCost,
    },
    rolls: {
      dcRequired: dc,
      tool: averageToolRoll,
      toolBonus: toolMod.value,
      skill: averageSkillRoll,
      skillBonus: skillMod.value,
      successChance: rollSuccessChance,
    },
  };
};

function langToolTrial() {
  console.log(currentActivity.value.time);
  let weeks = currentActivity.value.time - intMod.value;
  return {
    time: {
      unit: currentActivity.value.timeUnit,
      originalRequired: weeks,
    },
    cost: {
      originalRequired: currentActivity.value.daily * weeks,
    },
  };
}

function skillWeek(bonus, dc) {
  let rollTotal = 0;
  let successTotal = 0;
  let day;
  for (day = 0; day < 5; day++) {
    let rawRoll = roll();
    let success = rawRoll + bonus >= dc;

    rollTotal += rawRoll;
    successTotal += success;
    if (successTotal >= 3) break;
  }
  return {
    timeSuccess: successTotal >= 3,
    rollTotal,
    rollSucesses: successTotal,
    rollCount: day,
  };
}

function skillTrial(time, bonus, dc) {
  let timeSuccesses = 0;
  let skillRollTotal = 0;
  let rollSuccesses = 0;
  let rollCount = 0;
  let timeCount = 0;

  while (timeSuccesses < time && timeCount < 999) {
    let weekData = skillWeek(bonus, dc);

    timeSuccesses += weekData.timeSuccess ? 1 : 0;
    skillRollTotal += weekData.rollTotal;
    rollSuccesses += weekData.rollSucesses;
    rollCount += weekData.rollCount;
    timeCount++;
  }
  return {
    skillRollTotal,
    rollSuccesses,
    rollCount,
    timeCount,
    trialSuccess: timeCount < 999,
  };
}

function skillExperiment() {
  let requiredTime = currentActivity.value.time - intMod.value;
  let dc = currentActivity.value.dc;
  let bonus = skillMod.value;

  let totalDays = 0;
  let totalWeeks = 0;
  let totalRollSuccesses = 0;
  let totalSkillRolls = 0;

  let minWeeks = Infinity;
  let maxWeeks = 0;
  let minCost = Infinity;
  let maxCost = 0;

  let fails = 0;

  let timeData = [];

  for (let trial = 0; trial < trialCount.value; trial++) {
    let trialData = skillTrial(requiredTime, bonus, dc);
    fails += trialData.trialSuccess ? 0 : 1;

    totalSkillRolls += trialData.skillRollTotal;
    totalRollSuccesses += trialData.rollSuccesses;
    totalDays += trialData.rollCount;
    totalWeeks += trialData.timeCount;

    let trialCost = trialData.timeCount * currentActivity.value.daily;

    minWeeks = Math.min(minWeeks, trialData.timeCount);
    maxWeeks = Math.max(maxWeeks, trialData.timeCount);
    minCost = Math.min(minCost, trialCost);
    maxCost = Math.max(maxCost, trialCost);
    timeData.push(trialData.timeCount);
  }

  return {
    time: {
      unit: currentActivity.value.timeUnit,
      originalRequired: requiredTime,
      average: totalWeeks / trialCount.value,
      min: minWeeks,
      max: maxWeeks,
      trialSuccessRate: 1 - fails / trialCount.value,
      timeData,
    },
    cost: {
      originalRequired: currentActivity.value.daily * requiredTime,
      average: (totalWeeks / trialCount.value) * currentActivity.value.daily,
      min: minCost,
      max: maxCost,
    },
    rolls: {
      dcRequired: dc,
      skill: totalSkillRolls / totalDays,
      skillBonus: skillMod.value,
      successChance: totalRollSuccesses / totalDays,
    },
  };
}

function calculate() {
  if (calcMode.value === "craftMagicItem") {
    averages.value = magicTrials();
  } else if (calcMode.value === "learnLanguageTool") {
    averages.value = langToolTrial();
  } else if (calcMode.value === "learnSkill") {
    averages.value = skillExperiment();
  }
}
</script>
