<template>
  <div class="q-pa-md">
    <VueApexCharts
      type="area"
      height="350"
      :options="timeChartOptions"
      :series="timeSeries"
    />

    <VueApexCharts
      type="area"
      height="350"
      :options="costChartOptions"
      :series="costSeries"
    />
  </div>
</template>

<script setup>
import { ref, watch, toRef, onMounted, onUnmounted } from "vue";
import VueApexCharts from "vue3-apexcharts";

const props = defineProps({
  trialResults: {
    type: Object,
    default: () => ({}),
  },
});

const trialResults = toRef(props, "trialResults");

function convertToGraphableData(data) {
  const frequencyCounts = data.reduce((acc, value) => {
    acc[value] = (acc[value] || 0) + 1;
    return acc;
  }, {});

  return Object.entries(frequencyCounts).map(([number, frequencyCounts]) => [
    Number(number),
    frequencyCounts,
  ]);
}

const timeSeries = ref([
  {
    name: "Frequency",
    data: [[0, 0]],
  },
]);
const timeChartOptions = ref({
  chart: {
    width: "100%",
    type: "area",
  },
  annotations: {
    xaxis: [
      {
        borderColor: "#00E396",
        label: {
          borderColor: "#00E396",
          style: {
            color: "#fff",
            background: "#00E396",
          },
        },
      },
    ],
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
      const days = w.globals.labels[dataPointIndex];
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
  },
});
const costSeries = ref([
  {
    name: "Frequency",
    data: [[0, 0]],
  },
]);
const costChartOptions = ref({
  annotations: {
    xaxis: [
      {
        borderColor: "#00E396",
        label: {
          borderColor: "#00E396",
          style: {
            color: "#fff",
            background: "#00E396",
          },
        },
      },
    ],
  },
  chart: {
    height: 350,
    width: "100%",
    type: "area",

    background: "#f7f2e0",
  },
  stroke: {
    colors: ["#d4af37"],
  },
  dataLabels: {
    enabled: false,
  },
  xaxis: {
    tickAmount: 10,
    title: {
      text: "Gold",
      style: {
        color: "#b8860b",
      },
    },
  },
  yaxis: {
    title: {
      text: "Frequency",
      style: {
        color: "#b8860b",
      },
    },
  },
  y: {
    formatter: function (value, { series, seriesIndex, dataPointIndex, w }) {
      const gold = w.globals.labels[dataPointIndex];
      return `It took ${gold} gold ${value}% of the time`;
    },
  },
  legend: {
    horizontalAlign: "left",
    labels: {
      colors: "#d4af37",
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

    style: {
      backgroundColor: "#fffbe6",
      colors: ["#b8860b"],
    },
  },
  fill: {
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
      const data = convertToGraphableData(timeData);

      timeChartOptions.value.annotations.xaxis[0].x =
        trialResults.value.time.average;
      timeChartOptions.value.annotations.xaxis[0].label.text = `Average: ${trialResults.value.time.average}`;
      timeSeries.value[0].data = data;
    }
    if (costData) {
      const data = convertToGraphableData(costData);

      costChartOptions.value.annotations.xaxis[0].x =
        trialResults.value.cost.average;
      costChartOptions.value.annotations.xaxis[0].label.text = `Average: ${trialResults.value.cost.average}`;
      costSeries.value[0].data = data;
    }
  },

  {
    deep: true,
    immediate: true,
  }
);
</script>
