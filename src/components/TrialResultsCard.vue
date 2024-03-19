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

      <div v-if="hasData" class="q-pa-md">
        <VueApexCharts
          type="area"
          height="350"
          :options="chartOptions"
          :series="series"
        ></VueApexCharts>
      </div>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, watch, computed, toRefs } from "vue";
import VueApexCharts from "vue3-apexcharts";

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
const hasOtherStats = computed(
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
  () => hasTime.value || hasCost.value || hasOtherStats.value
);

const formattedSuccessChance = computed(() =>
  trialResults.value.rolls &&
  typeof trialResults.value.rolls.successChance !== "undefined"
    ? `${formatNumber(trialResults.value.rolls.successChance * 100)}%`
    : undefined
);
const formattedTrialSuccessChance = computed(() =>
  trialResults.value.time &&
  typeof trialResults.value.time.trialSuccessRate !== "undefined"
    ? `${formatNumber(trialResults.value.time.trialSuccessRate * 100)}%`
    : undefined
);

const series = ref([
  {
    name: "Frequency",
    data: [[0, 0]],
  },
]);
const chartOptions = ref({
  chart: {
    height: 350,
    type: "area",
  },
  stroke: {
    //curve: "smooth",
  },
  dataLabels: {
    enabled: false,
  },
  xaxis: {
    tickAmount: 10,
    title: {
      text: "Days",
      style: {
        color: "#000",
      },
    },
  },
  yaxis: {
    title: {
      text: "Frequency",
      style: {
        color: "#000",
      },
    },
  },
  y: {
    formatter: function (value, { series, seriesIndex, dataPointIndex, w }) {
      // Assuming `value` is the frequency and `w.globals.labels[dataPointIndex]` gives you the days
      const days = w.globals.labels[dataPointIndex];
      // Format the message with days and frequency
      return `It took ${days} days ${value}% of the time`;
    },
  },
  legend: {
    horizontalAlign: "left",
  },
  tooltip: {
    x: {
      show: true,
      formatter: (val) => `${val} days`,
    },
    y: {
      formatter: (val) => {
        const totalTrials = trialResults.value.time.timeData.length;
        const percentage = ((val / totalTrials) * 100).toFixed(2); // Keeping two decimal places for precision
        return `${val} out of ${totalTrials} times (${percentage}%)`;
      },
      title: {
        formatter: () => "", // This removes the series name by returning an empty string
      },
    },
    marker: {
      show: false, // This disables the color marker
    },
  },
});

watch(
  () => trialResults.value.time?.timeData,
  (newTimeData, oldTimeData) => {
    if (newTimeData) {
      const counts = newTimeData.reduce((acc, value) => {
        acc[value] = (acc[value] || 0) + 1;
        return acc;
      }, {});

      const data = Object.entries(counts).map(([number, count]) => [
        Number(number),
        count,
      ]);

      const totalValues = trialResults.value.time.timeData.length;
      const halfwayPoint = Math.floor((totalValues - 1) / 2);
      let accumulated = 0;
      let median;

      for (let [day, frequency] of data) {
        accumulated += frequency;
        if (accumulated > halfwayPoint) {
          median = day;
          break;
        } else if (accumulated === halfwayPoint && totalValues % 2 === 0) {
          // For even total values, average this day with the next day
          median =
            (day + data.find(([nextDay]) => Number(nextDay) > day)[0]) / 2;
          break;
        }
      }

      chartOptions.value = {
        ...chartOptions.value,
        annotations: {
          xaxis: [
            // {
            //   x: median,
            //   borderColor: "#00E396",
            //   label: {
            //     borderColor: "#00E396",
            //     style: {
            //       color: "#fff",
            //       background: "#00E396",
            //     },
            //     text: `Median: ${median}`,
            //   },
            // },
            {
              x: trialResults.value.time.average,
              borderColor: "#00E396",
              label: {
                borderColor: "#00E396",
                style: {
                  color: "#fff",
                  background: "#00E396",
                },
                text: `Average: ${trialResults.value.time.average}`,
              },
            },
          ],
        },
      };
      series.value[0].data = data;
      console.log(data);
    }
  },
  {
    deep: true,
    immediate: true,
  }
);
</script>
