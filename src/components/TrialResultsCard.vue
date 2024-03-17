<template>
  <q-card v-show="hasResults">
    <q-card-section class="bg-primary text-white">
      <div class="text-h6">Trial Results</div>
    </q-card-section>
    <q-card-section>
      <div class="q-pr-md row q-gutter-md row">
        <div class="col-12 col-sm row" v-if="hasTime">
          <q-card class="col">
            <q-card-section class="bg-secondary text-white">
              <div>Time</div>
            </q-card-section>
            <q-separator />
            <q-card-section>
              <div>
                <div v-if="trialResults.time.originalRequired">
                  <strong>Base:</strong>
                  {{ trialResults.time.originalRequired }}
                  {{ trialResults.time.unit }}s
                </div>
                <div v-if="trialResults.time.average">
                  <strong>Avg:</strong>
                  {{ formatNumber(trialResults.time.average) }}
                  {{ trialResults.time.unit }}s
                </div>
                <div v-if="trialResults.time.min">
                  <strong>Min:</strong> {{ trialResults.time.min }}
                  {{ trialResults.time.unit }}s
                </div>
                <div v-if="trialResults.time.max">
                  <strong>Max:</strong> {{ trialResults.time.max }}
                  {{ trialResults.time.unit }}s
                </div>
                <div
                  v-if="
                    typeof trialResults.time.trialSuccessRate !== 'undefined' &&
                    trialResults.time.trialSuccessRate < 1
                  "
                >
                  <strong>Successful Trials:</strong>
                  {{ formattedTrialSuccessChance }}
                </div>
              </div>
            </q-card-section>
          </q-card>
        </div>
        <div class="col-12 col-sm row" v-if="hasCost">
          <q-card class="col">
            <q-card-section class="bg-secondary text-white">
              <div>Cost</div>
            </q-card-section>
            <q-separator />
            <q-card-section>
              <div>
                <div v-if="trialResults.cost.originalRequired">
                  <strong>Base:</strong>
                  {{ trialResults.cost.originalRequired }}
                  Gold
                </div>
                <div v-if="trialResults.cost.average">
                  <strong>Avg:</strong>
                  {{ formatNumber(trialResults.cost.average) }}
                  Gold
                </div>
                <div v-if="trialResults.cost.min">
                  <strong>Min:</strong> {{ trialResults.cost.min }}
                  Gold
                </div>
                <div v-if="trialResults.cost.max">
                  <strong>Max:</strong> {{ trialResults.cost.max }}
                  Gold
                </div>
              </div>
            </q-card-section>
          </q-card>
        </div>
        <div class="col-12 col-sm row" v-if="hasOtherStats">
          <q-card class="col">
            <q-card-section class="bg-secondary text-white">
              <div>Rolls</div>
            </q-card-section>
            <q-separator />
            <q-card-section>
              <div>
                <div v-if="trialResults.rolls.dcRequired">
                  <strong>DC:</strong> {{ trialResults.rolls.dcRequired }}
                </div>
                <div
                  v-if="typeof trialResults.rolls.successChance !== 'undefined'"
                >
                  <strong>Success Chance:</strong>
                  {{ formattedSuccessChance }}
                </div>
                <div v-if="trialResults.rolls.skill">
                  <strong>Avg Skill Roll:</strong>
                  {{
                    formatNumber(
                      trialResults.rolls.skill +
                        (trialResults.rolls?.skillBonus ?? 0)
                    )
                  }}
                </div>
                <div v-if="trialResults.rolls.tool">
                  <strong>Avg Tool Roll:</strong>
                  {{
                    formatNumber(
                      trialResults.rolls.tool +
                        (trialResults.rolls?.toolBonus ?? 0)
                    )
                  }}
                </div>
              </div>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </q-card-section>
  </q-card>
</template>

<script>
export default {
  name: "TrialResultsCard",
  props: {
    trialResults: {
      type: Object,
      default: () => ({}),
    },
  },
  methods: {
    formatNumber(value) {
      // This will convert to a fixed number of decimal places, then parse to float to remove trailing zeros
      return parseFloat(value.toFixed(2));
    },
  },
  computed: {
    hasTime() {
      return (
        this.trialResults.time &&
        (typeof this.trialResults.time.average !== "undefined" ||
          typeof this.trialResults.time.min !== "undefined" ||
          typeof this.trialResults.time.max !== "undefined" ||
          typeof this.trialResults.time.originalRequired !== "undefined")
      );
    },
    hasCost() {
      return (
        this.trialResults.cost &&
        (typeof this.trialResults.cost.average !== "undefined" ||
          typeof this.trialResults.cost.min !== "undefined" ||
          typeof this.trialResults.cost.max !== "undefined" ||
          typeof this.trialResults.cost.originalRequired !== "undefined")
      );
    },
    hasOtherStats() {
      return (
        this.trialResults.rolls &&
        (typeof this.trialResults.rolls.dcRequired !== "undefined" ||
          typeof this.trialResults.rolls.successChance !== "undefined" ||
          typeof this.trialResults.rolls.skill !== "undefined" ||
          typeof this.trialResults.rolls.tool !== "undefined" ||
          typeof this.trialResults.rolls.skillBonus !== "undefined" ||
          typeof this.trialResults.rolls.toolBonus !== "undefined")
      );
    },
    formattedSuccessChance() {
      if (
        this.trialResults.rolls &&
        typeof this.trialResults.rolls.successChance !== "undefined"
      ) {
        // Convert to percentage and format
        return (
          this.formatNumber(this.trialResults.rolls.successChance * 100) + "%"
        );
      }
      return undefined;
    },
    formattedTrialSuccessChance() {
      if (
        this.trialResults.rolls &&
        typeof this.trialResults.time.trialSuccessRate !== "undefined"
      ) {
        // Convert to percentage and format
        return (
          this.formatNumber(this.trialResults.time.trialSuccessRate * 100) + "%"
        );
      }
      return undefined;
    },
    hasResults() {
      return this.hasTime || this.hasCost || this.hasOtherStats;
    },
  },
};
</script>

<style scoped></style>
