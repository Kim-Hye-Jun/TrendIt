<template>
  <q-card flat bordered class="chart-container text-center bg-grey-1">
    <q-card-section>
      <q-btn-toggle
        v-model="type"
        toggle-color="deep-orange-3"
        toggle-text-color="black"
        square
        :options="[
          { label: '일별', value: 'day' },
          { label: '주별', value: 'week' },
          { label: '월별', value: 'month' },
          { label: '연도별', value: 'year' },
        ]"
      />
      <!-- CHART -->
      <bar-chart-comp
        v-if="loaded"
        :chart-data="chartData"
        :chart-options="chartOptions"
        :width="550"
        :height="350"
        class="chart"
      />
      <div v-else class="chart" style="border: 1px red solid">
        그래프를 표시할 수 없습니다.
      </div>
      <!-- SLIDE -->
      <div class="text-center">
        <q-slider
          class="slider"
          reverse
          v-model="slide"
          :min="s_min"
          :max="s_max"
          markers
          :marker-labels="markers"
          color="blue-10"
          inner-track-color="blue-2"
          selection-color="blue-2"
        />
      </div>
    </q-card-section>
  </q-card>
</template>

<script>
import { ref } from "vue";
import { mainChartApi } from "boot/stats.js";
import BarChartComp from "./BarChartComp.vue";

export default {
  setup() {
    const response = ref(null);
    const data_list = ref(null);

    const type = ref("day");
    const slide = ref(0);
    const s_min = ref(0);
    const s_max = ref(7);
    const markers = ref([]);

    const loaded = ref(false);
    const chartData = ref(null);
    const chartOptions = ref({
      indexAxis: "y",
      plugins: {
        legend: {
          display: false,
        },
        datalabels: {
          display: false,
        },
        tooltip: {
          boxWidth: 10,
        },
      },
      scales: {
        x: {
          grid: {
            drawBorder: false,
            color: "black",
            lineWidth: 0.2,
          },
          ticks: {
            callback: (val, index) => {
              return val !== 0 ? val : "";
            },
            padding: 3,
            stepSize: 100, // 보조눈금
          },

          title: {
            display: true,
            align: "end",
            color: "#808080",
            font: {
              size: 12,
              family: "NanumBarunGothic",
              weight: 300,
            },
            text: "(단위: 건)",
          },
        },
        y: {
          grid: {
            display: false,
          },
        },
      },
      responsive: true,

      animation: {
        x: {
          duration: 500,
        },
        y: {
          duration: 200,
        },
      },
    });

    return {
      response,
      data_list,

      type,
      slide,
      s_min,
      s_max,
      markers,

      loaded,
      chartData,
      chartOptions,
    };
  },
  created() {
    this.loaded = false;
    this.loadChartData();
    this.resetSlide();
  },
  methods: {
    async loadChartData() {
      await mainChartApi(
        { type: this.type, val: this.slide + 1 },
        (response) => {
          this.response = response.data;
          this.data_list = response.data.data;

          this.chartData = {
            labels: [],
            datasets: [
              {
                label: "기사 수", // 툴팁 표시 이름
                backgroundColor: "#EE6C4D", // 막대 색상
                barThickness: "15", // 막대 굵기
                data: [],
              },
            ],
          };

          this.data_list.forEach((item) => {
            this.chartData.labels.push(item.keyword);
            this.chartData.datasets[0].data.push(item.frequency);
          });
          this.loaded = true;
        },
        () => console.warn("failed")
      );
    },
    resetSlide() {
      switch (this.type) {
        case "day":
          this.s_max = 6;
          break;
        case "week":
          this.s_max = 4;
          break;
        case "month":
          this.s_max = 11;
          break;
        case "year":
          this.s_max = 7;
          break;
        default:
          break;
      }
      this.markers = [];
      for (var i = 0; i <= this.s_max; i++) {
        var marker = this.getMarker(i);
        this.markers.push({ value: i, label: marker });
      }
    },
    getMarker(offset) {
      var date_val = new Date();
      var date_label = "";
      switch (this.type) {
        case "day":
          date_val.setDate(date_val.getDate() - offset - 1);
          date_label = date_val.getMonth() + 1 + "/" + date_val.getDate();
          break;
        case "week":
          if (date_val.getDay() == 1) date_val.setDate(date_val.getDate() - 1);
          if (date_val.getDay() == 0) date_val.setDate(date_val.getDate() - 6);
          else date_val.setDate(date_val.getDate() - (date_val.getDay() - 1));
          date_val.setDate(date_val.getDate() - offset * 7);
          date_label = date_val.getMonth() + 1 + "/" + date_val.getDate();
          break;
        case "month":
          if (date_val.getDate() == 1) date_val.setDate(date_val.getDate() - 1);
          date_val.setMonth(date_val.getMonth() - offset);
          date_label = date_val.getMonth() + 1 + "월";
          break;
        case "year":
          date_val.setFullYear(date_val.getFullYear() - offset);
          date_label = date_val.getFullYear() + "년";
          break;
        default:
          break;
      }
      return date_label;
    },
  },
  watch: {
    type: function () {
      this.slide = 0;
      this.resetSlide();
      this.loadChartData();
    },
    slide: function () {
      this.loadChartData();
    },
  },
  components: { BarChartComp },
};
</script>

<style scoped>
.slider {
  width: 90%;
}
.chart {
  width: 550px;
  height: 350px;
  margin: 10px auto;
}
.chart-container {
  font-family: "NanumBarunGothic";
  padding: 5px;
  width: 700px;
  margin: 20px auto;
}
</style>
