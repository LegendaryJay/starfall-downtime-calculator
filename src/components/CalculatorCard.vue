<template>
  <q-card class="q-ma-md">
    <q-card-section>
      <q-btn-toggle
        v-model="calcMode"
        push
        glossy
        toggle-color="primary"
        :options="[
          { label: 'Craft Magic Item', value: 'craftMagicItem' },
          { label: 'Learn Language/Tool', value: 'learnLanguageTool' },
          { label: 'Learn Skill', value: 'learnSkill' },
        ]"
      />
    </q-card-section>
    <q-card-section>
      <q-select
        v-model="itemRarity"
        label="Magic Item Rarity"
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
    </q-card-section>
    <q-card-section class="row">
      <div class="col-auto q-pr-md">
        <q-input
          dense
          filled
          v-model.number="trialCount"
          label="Trials"
          type="number"
          class="q-mt-md q-pa-auto"
        />
      </div>
      <q-btn class="col" @click="calculate">Calculate</q-btn>
    </q-card-section>
  </q-card>
  <TrialResultsCard :trialResults="averages" />
</template>

<script>
import { ref } from "vue";
import TrialResultsCard from "./TrialResultsCard.vue";
export default {
  components: {
    TrialResultsCard,
  },
  // name: 'ComponentName',

  setup() {
    const trialCount = ref(1000);
    const averages = ref({});

    const rarityOptions = [
      {
        label: "Common",
        value: 0,
        dc: 10,
        days: 5,
        gold: 50,
        daily: 10,
      },
      {
        label: "Uncommon",
        value: 1,
        dc: 15,
        days: 15,
        gold: 300,
        daily: 20,
      },
      {
        label: "Rare",
        value: 2,
        dc: 20,
        days: 30,
        gold: 7500,
        daily: 250,
      },
      {
        label: "Very Rare",
        value: 3,
        dc: 25,
        days: 50,
        gold: 35000,
        daily: 700,
      },
    ];

    const itemRarity = ref(rarityOptions[0]); // Default value can be adjusted
    const isExpertise = ref(false);
    const isConsumable = ref(false);
    const intMod = ref(0);
    const skillMod = ref(0);
    const toolMod = ref(0);
    const calcMode = ref("craftMagicItem");

    const output = {};
    function roll() {
      return Math.floor(Math.random() * 20) + 1;
    }

    let magicTrial = function () {
      let trials = trialCount.value;

      let requiredSuccesses = itemRarity.value.days - intMod.value;
      requiredSuccesses = requiredSuccesses <= 0 ? 1 : requiredSuccesses;

      let baseCost = itemRarity.value.gold;
      let cost = isConsumable.value ? baseCost / 2 : baseCost;

      let dc = itemRarity.value.dc;

      let totalTime = 0;
      let totalCost = 0;
      let totalSkillRolls = 0;
      let totalToolRolls = 0;
      let successCount = 0;

      let minTime = Infinity;
      let maxTime = 0;
      let minCost = Infinity;
      let maxCost = 0;

      for (let trial = 0; trial < trials; trial++) {
        let successes = 0;
        let trialTime = 0;
        let trialCost = cost;

        while (successes < requiredSuccesses) {
          if (trialTime > 999) break;

          let toolRoll = roll();
          let skillRoll = roll();

          let nat1 = toolRoll == 1 || skillRoll == 1;
          let nat20 = toolRoll == 20 || skillRoll == 20;

          if (nat1) {
            successes--;
            trialCost += itemRarity.value.daily;
          }

          if (nat20) {
            successes++;
          }

          if (
            toolRoll + toolMod.value >= dc &&
            skillRoll + skillMod.value >= dc
          ) {
            successes++;
          }

          totalSkillRolls += skillRoll;
          totalToolRolls += toolRoll;

          trialTime++;
        }

        successCount += successes;
        totalTime += trialTime;
        totalCost += trialCost;
        minTime = Math.min(minTime, trialTime);
        maxTime = Math.max(maxTime, trialTime);
        minCost = Math.min(minCost, trialCost);
        maxCost = Math.max(maxCost, trialCost);
      }

      let averageTime = totalTime / trials;
      let averageCost = totalCost / trials;
      let averageSkillRoll = totalSkillRolls / totalTime;
      let averageToolRoll = totalToolRolls / totalTime;
      let successChance = successCount / totalTime;

      return {
        time: {
          unit: "Days",
          originalRequired: requiredSuccesses,
          average: averageTime,
          min: minTime,
          max: maxTime,
        },
        cost: {
          originalRequired: baseCost,
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
          successChance: successChance,
        },
      };
    };

    function langToolTrial() {
      let weeks = 15 - intMod.value;
      return {
        time: {
          unit: "Weeks",
          originalRequired: weeks,
        },
        cost: {
          originalRequired: 50 * weeks,
        },
      };
    }

    function skillTrial() {
      let trials = trialCount.value;
      let weeklyCost = isExpertise.value ? 1500 : 250;
      let requiredWeeks = (isExpertise.value ? 20 : 15) - intMod.value;
      let dc = isExpertise.value ? 22 : 16;

      let totalWeeks = 0;
      let totalCost = 0;
      let totalSkillRolls = 0;
      let totalSkillModRolls = 0;
      let totalSuccesses = 0;

      let minWeeks = Infinity;
      let maxWeeks = 0;
      let minCost = Infinity;
      let maxCost = 0;

      let totalDays = 0;

      for (let trial = 0; trial < trials; trial++) {
        let weeks = 0;
        let weeksuccess = 0;
        let skillRollsThisTrial = 0;
        let skillModRollThisTrial = 0;
        let successesThisTrial = 0;

        while (weeksuccess < requiredWeeks) {
          let weekDaysSuccess = 0;
          let day;

          for (day = 0; day < 5; day++) {
            totalDays++;
            let skillRoll = roll();
            skillRollsThisTrial += skillRoll;

            if (skillRoll + skillMod.value >= dc) {
              weekDaysSuccess++;
            }
            if (weekDaysSuccess >= 3) {
              successesThisTrial++;
              break; // Success for the week
            }
          }
          if (weekDaysSuccess >= 3) {
            weeksuccess++;
          }
          weeks++;
        }

        let trialCost = weeklyCost * weeks;
        totalWeeks += weeks;
        totalCost += trialCost;
        totalSkillRolls += skillRollsThisTrial;
        totalSkillModRolls += skillModRollThisTrial;

        minWeeks = Math.min(minWeeks, weeks);
        maxWeeks = Math.max(maxWeeks, weeks);
        minCost = Math.min(minCost, trialCost);
        maxCost = Math.max(maxCost, trialCost);
      }

      let averageWeeks = totalWeeks / trials;
      let averageCost = totalCost / trials;
      let averageSkillRoll = totalSkillRolls / totalDays;
      let successChance = (requiredWeeks * trials) / totalWeeks;
      console.log("success: ", successChance);

      return {
        time: {
          unit: "Weeks",
          originalRequired: requiredWeeks,
          average: averageWeeks,
          min: minWeeks,
          max: maxWeeks,
        },
        cost: {
          originalRequired: weeklyCost * requiredWeeks,
          average: averageCost,
          min: minCost,
          max: maxCost,
        },
        rolls: {
          dcRequired: dc,
          skill: averageSkillRoll,
          skillBonus: skillMod.value,
          successChance: successChance,
        },
      };
    }

    function calculate() {
      if (calcMode.value === "craftMagicItem") {
        averages.value = magicTrial();
      } else if (calcMode.value === "learnLanguageTool") {
        averages.value = langToolTrial();
      } else if (calcMode.value === "learnSkill") {
        averages.value = skillTrial();
      }
      console.log(averages.value);
    }
    return {
      itemRarity,
      isExpertise,
      intMod,
      skillMod,
      toolMod,
      rarityOptions,
      calcMode,
      isConsumable,
      roll,
      calculate,
      trialCount,
      averages,
    };
  },
};
</script>
