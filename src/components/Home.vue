<template>
  <div class="max-w-screen-xl mx-auto px-5 font-lato">
    <div class="text-xl md:text-5xl font-bold">Critical Illness Insurance Calculator</div>
    <div
      class="text-base md:text-xl p-5"
    >Adjust the sliders to estimate the coverage amount you may need</div>

    <!--  -->
    <div class="flex flex-col md:flex-row md:mt-10 w-full">
      <div class="w-full md:w-1/4">
        <input-with-slider
          label="Desired replaced income (after-tax)"
          layoverText="/month"
          v-model="DesiredRepIncome"
        />
        <input-with-slider label="Out-of-pocket healthcare expenses" v-model="OutOfPockExp" />
        <input-with-slider label="Home modification expenses" v-model="HomeModExp" />
        <input-with-slider
          label="Medical homecare expenses"
          v-model="MedHomeExp"
          layoverText="/month"
        />
        <input-with-slider
          label="Other expenses (transport, childcare, etc)"
          v-model="OtherExp"
          layoverText="/month"
        />
        <div class="flex justify-between">
          <div class="font-bold">Additional Info</div>
          <!-- plus minus toggle -->
          <div class="theme-color" style="cursor-pointer" @click="additionalInfo = !additionalInfo">
            <svg
              v-if="!additionalInfo"
              width="1em"
              height="1em"
              viewBox="0 0 20 20"
              class="bi bi-plus-square cursor-pointer"
              fill="currentColor"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                fill-rule="evenodd"
                d="M8 3.5a.5.5 0 0 1 .5.5v4a.5.5 0 0 1-.5.5H4a.5.5 0 0 1 0-1h3.5V4a.5.5 0 0 1 .5-.5z"
              />
              <path
                fill-rule="evenodd"
                d="M7.5 8a.5.5 0 0 1 .5-.5h4a.5.5 0 0 1 0 1H8.5V12a.5.5 0 0 1-1 0V8z"
              />
              <path
                fill-rule="evenodd"
                d="M14 1H2a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V2a1 1 0 0 0-1-1zM2 0a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V2a2 2 0 0 0-2-2H2z"
              />
            </svg>
            <svg
              v-else
              width="1em"
              height="1em"
              viewBox="0 0 20 20"
              class="bi bi-dash-square cursor-pointer"
              fill="currentColor"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                fill-rule="evenodd"
                d="M14 1H2a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1V2a1 1 0 0 0-1-1zM2 0a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V2a2 2 0 0 0-2-2H2z"
              />
              <path
                fill-rule="evenodd"
                d="M3.5 8a.5.5 0 0 1 .5-.5h8a.5.5 0 0 1 0 1H4a.5.5 0 0 1-.5-.5z"
              />
            </svg>
          </div>
        </div>
        <div
          class="w-full bg-gray-100 text-gray-600 p-2 my-2 text-left"
          v-if="additionalInfo"
        >More information here</div>
      </div>
      <div class="w-full md:w-3/4 flex flex-col">
        <div class="md:mx-20">
          <div
            class="w-full p-5 md:p-8 border border-gray-400"
            style="box-shadow: -3px 3px 3px #00000029;"
          >
            <input-with-slider
              label="Recovery period"
              layoverText="months"
              mask="000"
              v-model="recovPeriod"
              :min="6"
              :max="36"
              :step="1"
              type
              labelOnLeft
            />
          </div>
        </div>
        <div class="mt-10 md:m-20 md:mx-20" style="min-height:400px">
          <span v-if="loading">loading...</span>
          <apexchart
            type="bar"
            :options="options"
            :series="series"
            height="100%"
            class="text-xs md:text-base"
            v-show="!loading"
          ></apexchart>
        </div>
      </div>
    </div>

    <div class="flex flex-col md:flex-row mb-10 w-full justify-end">
      <div class="w-full md:w-1/4 flex flex-col"></div>
      <div class="w-full md:w-3/4 flex flex-col">
        <div class="theme-color my-5">Assumptions</div>
        <div class="px-2 md:px-32 mb-5">
          <p class="text-xl">
            A serious illness with recovery lasting
            <span
              class="theme-color font-bold"
            >{{recovPeriod}} months</span> could put your finances down by
            <span class="theme-color font-bold">{{costTodayF}}</span> today and by
            <span class="theme-color font-bold">{{costIn10YF}}</span> in 10 years.
          </p>
        </div>
        <div class="mx-auto">
          <button
            class="font-lato text-white theme-bg-color px-5 py-2 md:py-3 md:px-8 font-bold"
          >START COMPARING QUOTES</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import InputWithSlider from "./InputWithSlider";
import VueApexCharts from "vue-apexcharts";

let sleep = (t) => {
  return new Promise((r) => setTimeout(r, t));
};

let usCurFormatter = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
  maximumSignificantDigits: 6,
});
let usCurFormatterP = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

export default {
  name: "home",
  components: {
    InputWithSlider,
    apexchart: VueApexCharts,
  },
  data() {
    return {
      loading: false,
      additionalInfo: false,
      DesiredRepIncome: 2000,
      OutOfPockExp: 150,
      HomeModExp: 200,
      MedHomeExp: 150,
      OtherExp: 200,
      recovPeriod: 6,
      options: {
        chart: {
          id: "vuechart-example",
          fontFamily: "Barlow",
          animations: {
            enabled: false,
          },
        },
        yaxis: {
          labels: {
            style: {
              fontSize: "1.1em",
              fontWeight: 600,
            },
            formatter: (val) => {
              if (val > 1000) {
                return usCurFormatter.format(val / 1000) + "k";
              } else {
                return usCurFormatter.format(val);
              }
            },
          },
        },
        plotOptions: {
          bar: {
            dataLabels: {
              position: "top",
            },
            columnWidth: "40%",
            barHeight: "60%",
            backgroundBarColors: ["#17b3e4"],
          },
        },
        colors: ["#17b3e4"],
        dataLabels: {
          enabled: true,
          style: {
            fontSize: "1.2em",
            fontFamily: "Barlow",
            fontWeight: "regular",
            colors: ["#333"],
          },
          offsetY: -20,
          formatter: function (val) {
            return usCurFormatterP.format(val);
          },
        },
        xaxis: {
          categories: ["Estimated Cost today", "Estimated Cost in 10 years"],
          labels: {
            style: {
              fontSize: "1em",
              fontWeight: 500,
            },
          },
        },
      },
    };
  },
  computed: {
    // Formula needs review
    costToday() {
      return (
        this.DesiredRepIncome +
        this.OutOfPockExp +
        this.HomeModExp +
        this.MedHomeExp -
        this.OtherExp
      );
    },
    costTodayF() {
      return usCurFormatterP.format(this.costToday);
    },
    costIn10Y() {
      return (
        ((this.DesiredRepIncome +
          this.OutOfPockExp +
          this.HomeModExp +
          this.MedHomeExp) *
          1.2) /
        this.recovPeriod
      );
    },
    costIn10YF() {
      return usCurFormatterP.format(this.costIn10Y);
    },
    series() {
      return [
        {
          name: "Cost",
          data: [this.costToday, this.costIn10Y],
        },
      ];
    },
  },
  mounted() {
    this.$watch(
      (vm) => [
        vm.DesiredRepIncome,
        vm.OutOfPockExp,
        vm.HomeModExp,
        vm.MedHomeExp,
        vm.OtherExp,
        vm.recovPeriod,
      ],
      () => {
        this.loadChart();
      }
    );
  },
  methods: {
    async loadChart() {
      this.loading = true;
      await sleep(3000);
      this.loading = false;
    },
  },
};
</script>

<style scoped>

</style>

