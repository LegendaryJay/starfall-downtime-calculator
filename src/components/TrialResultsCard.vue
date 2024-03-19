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

      <div v-if="hasData" class="q-pa-md column">
        <div class="col">
          <VueApexCharts
            type="area"
            height="350"
            :options="timeChartOptions"
            :series="timeSeries"
          ></VueApexCharts>
        </div>
        <div class="col">
          <VueApexCharts
            type="area"
            height="350"
            :options="costChartOptions"
            :series="costSeries"
          ></VueApexCharts>
        </div>
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

const timeSeries = ref([
  {
    name: "Frequency",
    data: [[0, 0]],
  },
]);
const timeChartOptions = ref({
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
const costSeries = ref([
  {
    name: "Frequency",
    data: [[0, 0]],
  },
]);
const costChartOptions = ref({
  chart: {
    height: 350,
    type: "area",
    // Adding a background color that complements gold
    background: "#f7f2e0", // A light cream background for contrast
  },
  stroke: {
    curve: "smooth",
    // Setting the stroke color to a golden shade
    colors: ["#d4af37"], // A rich gold color
  },
  dataLabels: {
    enabled: false,
  },
  xaxis: {
    tickAmount: 10,
    title: {
      text: "Gold",
      style: {
        color: "#b8860b", // Darker gold for text to ensure readability
      },
    },
  },
  yaxis: {
    title: {
      text: "Frequency",
      style: {
        color: "#b8860b", // Darker gold for text to ensure readability
      },
    },
  },
  y: {
    formatter: function (value, { series, seriesIndex, dataPointIndex, w }) {
      const days = w.globals.labels[dataPointIndex];
      return `It took ${days} gold ${value}% of the time`;
    },
  },
  legend: {
    horizontalAlign: "left",
    labels: {
      colors: "#d4af37", // Gold color for legend text
    },
  },
  tooltip: {
    x: {
      show: true,
      formatter: (val) => `${val} gold`,
    },
    y: {
      formatter: (val) => {
        const totalTrials = trialResults.value.time.timeData.length;
        const percentage = ((val / totalTrials) * 100).toFixed(2);
        return `${val} out of ${totalTrials} times (${percentage}%)`;
      },
      title: {
        formatter: () => "",
      },
    },
    marker: {
      show: false,
    },
    // Adjusting the tooltip background and text colors for the gold theme
    style: {
      backgroundColor: "#fffbe6", // Light gold background for tooltip
      colors: ["#b8860b"], // Dark gold for tooltip text
    },
  },
  fill: {
    // Gradient fill for the area chart to give it a more dynamic gold appearance
    type: "gradient",
    gradient: {
      shadeIntensity: 1,
      inverseColors: false,
      opacityFrom: 0.5,
      opacityTo: 0.3,
      stops: [0, 90, 100],
      colorStops: [
        {
          offset: 0,
          color: "#d4af37",
          opacity: 0.5,
        },
        {
          offset: 100,
          color: "#f7f2e0",
          opacity: 0.3,
        },
      ],
    },
  },
});

watch(
  () => trialResults.value,
  (newData, oldTimeData) => {
    let timeData = newData?.time?.timeData;
    let costData = newData?.cost?.costData;
    if (timeData) {
      const counts = timeData.reduce((acc, value) => {
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

      timeChartOptions.value = {
        ...timeChartOptions.value,
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
      timeSeries.value[0].data = data;
    }
    if (costData) {
      const counts = costData.reduce((acc, value) => {
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

      costChartOptions.value = {
        ...costChartOptions.value,
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
              x: trialResults.value.cost.average,
              borderColor: "#00E396",
              label: {
                borderColor: "#00E396",
                style: {
                  color: "#fff",
                  background: "#00E396",
                },
                text: `Average: ${trialResults.value.cost.average}`,
              },
            },
          ],
        },
      };
      costSeries.value[0].data = data;
    }
  },

  {
    deep: true,
    immediate: true,
  }
);
</script>
