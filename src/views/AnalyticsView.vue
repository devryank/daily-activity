<script>
import Vue3ChartJs from "@j-t-mcc/vue3-chartjs";
export default {
  components: {
    Vue3ChartJs,
  },
  setup() {},

  data() {
    return {
      DB_NAME: "acitivitydb",
      DB_VERSION: 1,
      db: null,
      ready: false,
      activities: [],
      period: "week",
      totalActivities: 0,
      totalPositiveActivities: 0,
      totalNegativeActivities: 0,
      totalNeutralActivities: 0,
      pieChart: {},
      chartKey: 0,

      groupDate: [],
      groupDay: [],
      chartPositiveActivities: [],
      chartNegativeActivities: [],
      chartNeutralActivities: [],
    };
  },
  async created() {
    this.db = await this.getDb();
    this.activities = await this.getActivitiesFromDb();
    this.ready = true;
    await this.getWeekActivities();
  },
  methods: {
    async makeChart() {
      if (this.period == "year") {
        // data chart specific for year period
        for (let i = 0; i < this.groupDay.length; i++) {
          // for example, positive activities in January is this.groupDay[0][01].positive
          this.chartPositiveActivities[i] =
            this.groupDay[i][("0" + (i + 1)).slice(-2)].positive;
          this.chartNegativeActivities[i] =
            this.groupDay[i][("0" + (i + 1)).slice(-2)].negative;
          this.chartNeutralActivities[i] =
            this.groupDay[i][("0" + (i + 1)).slice(-2)].neutral;
        }
      } else {
        // data chart for today and month
        for (let i = 0; i < this.groupDay.length; i++) {
          this.chartPositiveActivities[i] = this.groupDay[i].positive;
          this.chartNegativeActivities[i] = this.groupDay[i].negative;
          this.chartNeutralActivities[i] = this.groupDay[i].neutral;
        }
      }
      let pieChart = {
        type: "line",
        data: {
          labels: this.groupDate,
          datasets: [
            {
              label: "Positif",
              borderColor: "#059669",
              data: this.chartPositiveActivities,
            },
            {
              label: "Negatif",
              borderColor: "#E11D48",
              data: this.chartNegativeActivities,
            },
            {
              label: "Netral",
              borderColor: "#4F46E5",
              data: this.chartNeutralActivities,
            },
          ],
        },
        options: {
          responsive: true,
          interaction: {
            intersect: false,
            axis: "x",
          },
          x: {
            ticks: {
              maxTicksLimit: 15,
            },
          },
          scales: {
            y: {
              ticks: {
                stepSize: 1,
              },
            },
          },
        },
      };
      return pieChart;
    },
    async getDb() {
      return new Promise((resolve, reject) => {
        let request = window.indexedDB.open(this.DB_NAME, this.DB_VERSION);

        request.onerror = (e) => {
          console.log("Error opening db", e);
          reject("Error");
        };

        request.onsuccess = (e) => {
          resolve(e.target.result);
        };

        request.onupgradeneeded = (e) => {
          console.log("onupgradeneeded");
          let db = e.target.result;
          let objectStore = db.createObjectStore("activities", {
            autoIncrement: true,
            keyPath: "id",
          });
        };
      });
    },

    async getActivitiesFromDb() {
      return new Promise((resolve, reject) => {
        let trans = this.db.transaction(["activities"], "readonly");
        trans.oncomplete = (e) => {
          resolve(activities);
        };

        let store = trans.objectStore("activities");
        let activities = [];

        store.openCursor().onsuccess = (e) => {
          let cursor = e.target.result;
          if (cursor) {
            activities.push(cursor.value);
            cursor.continue();
          }
        };
      });
    },

    getDatesInRange(startDate, endDate) {
      const date = new Date(startDate.getTime());

      const dates = [];

      while (date <= endDate) {
        dates.push(new Date(date));
        date.setDate(date.getDate() + 1);
      }

      return dates;
    },

    async getWeekActivities() {
      this.period = "week";
      // If no date object supplied, use current date
      // Copy date so don't modify supplied date
      let date = new Date(Date.now());
      let now = date ? new Date(date) : new Date();

      // set time to some convenient value
      now.setHours(0, 0, 0, 0);

      this.activities = await this.getActivitiesFromDb();

      // Get the previous Sunday
      let sunday = new Date(now);
      sunday.setDate(sunday.getDate() - sunday.getDay());

      // Get next Saturday
      let saturday = new Date(now);
      saturday.setDate(saturday.getDate() - saturday.getDay() + 6);
      saturday = new Date(saturday.setHours(23, 59, 59, 999));

      // get this week activities
      this.activities = this.activities.filter((d) => {
        return new Date(d.time) >= sunday && new Date(d.time) < saturday;
      });

      // get list dates between two dates
      let dateInRange = this.getDatesInRange(sunday, saturday);

      let tempGroupDay = [];
      tempGroupDay = dateInRange;
      let groupDay = [];
      let groupDate = [];

      for (let j = 0; j < tempGroupDay.length; j++) {
        groupDay[j] = tempGroupDay[j];
        // declare default value positive, negative, and neutral for each day
        groupDay[j]["positive"] = 0;
        groupDay[j]["negative"] = 0;
        groupDay[j]["neutral"] = 0;

        // x axis label chart
        groupDate.push(
          ("0" + (groupDay[j].getMonth() + 1)).slice(-2) +
            "/" +
            ("0" + groupDay[j].getDate()).slice(-2) +
            "/" +
            groupDay[j].getFullYear()
        );
      }
      this.groupDate = groupDate;
      for (let i = 0; i < this.activities.length; i++) {
        for (let j = 0; j < groupDay.length; j++) {
          let nowDay = this.activities[i].time;
          let formatGroupDay =
            ("0" + (groupDay[j].getMonth() + 1)).slice(-2) +
            "/" +
            ("0" + groupDay[j].getDate()).slice(-2) +
            "/" +
            groupDay[j].getFullYear();
          if (formatGroupDay == nowDay) {
            if (this.activities[i].value == "+") {
              groupDay[j]["positive"] += 1;
            } else if (this.activities[i].value == "-") {
              groupDay[j]["negative"] += 1;
            } else if (this.activities[i].value == "=") {
              groupDay[j]["neutral"] += 1;
            }
            break;
          }
        }
      }
      this.groupDay = groupDay;
      this.pieChart = await this.makeChart();
      this.chartKey += 1; // update chart by changing the key

      this.getTotalActivities();
    },

    async getMonthActivities() {
      this.period = "month";
      // If no date object supplied, use current date
      // Copy date so don't modify supplied date
      let date = new Date(Date.now());
      let start = date ? new Date(date) : new Date();

      this.activities = await this.getActivitiesFromDb();
      // Get the first Month
      let firstDay = new Date(start);
      firstDay = new Date(firstDay.getFullYear(), firstDay.getMonth(), 1);

      // Get the last day of Month
      let lastDay = new Date(start);
      lastDay = new Date(lastDay.getFullYear(), lastDay.getMonth() + 1, 0);
      lastDay = new Date(lastDay.setHours(23, 59, 59, 999));

      // get this month activities
      this.activities = this.activities.filter((d) => {
        return new Date(d.time) >= firstDay && new Date(d.time) < lastDay;
      });

      // get list dates between two dates
      let dateInRange = this.getDatesInRange(firstDay, lastDay);

      let tempGroupDay = [];
      tempGroupDay = dateInRange;
      let groupDay = [];

      let groupDate = [];
      for (let j = 0; j < tempGroupDay.length; j++) {
        groupDay[j] = tempGroupDay[j];
        // declare default value positive, negative, and neutral for each day
        groupDay[j]["positive"] = 0;
        groupDay[j]["negative"] = 0;
        groupDay[j]["neutral"] = 0;

        // x axis label chart
        groupDate.push(
          ("0" + (groupDay[j].getMonth() + 1)).slice(-2) +
            "/" +
            ("0" + groupDay[j].getDate()).slice(-2) +
            "/" +
            groupDay[j].getFullYear()
        );
      }
      this.groupDate = groupDate;

      for (let i = 0; i < this.activities.length; i++) {
        for (let j = 0; j < groupDay.length; j++) {
          let nowDay = this.activities[i].time;
          let formatGroupDay =
            ("0" + (groupDay[j].getMonth() + 1)).slice(-2) +
            "/" +
            ("0" + groupDay[j].getDate()).slice(-2) +
            "/" +
            groupDay[j].getFullYear();
          if (formatGroupDay == nowDay) {
            if (this.activities[i].value == "+") {
              groupDay[j]["positive"] += 1;
            } else if (this.activities[i].value == "-") {
              groupDay[j]["negative"] += 1;
            } else if (this.activities[i].value == "=") {
              groupDay[j]["neutral"] += 1;
            }
            break;
          }
        }
      }
      this.groupDay = groupDay;
      this.pieChart = await this.makeChart();
      this.chartKey += 1; // update chart by changing the key

      this.getTotalActivities();
    },

    async getYearActivities() {
      this.period = "year";
      // If no date object supplied, use current date
      // Copy date so don't modify supplied date
      let date = new Date(Date.now());
      let start = date ? new Date(date) : new Date();

      this.activities = await this.getActivitiesFromDb();
      // Get the first day of Year
      let firstDay = new Date(start);
      firstDay = new Date(firstDay.getFullYear(), 0, 1);

      // Get the last day of Year
      let lastDay = new Date(start);
      lastDay = new Date(lastDay.getFullYear(), 11, 31);
      lastDay = new Date(lastDay.setHours(23, 59, 59, 999));

      // get this month activities
      this.activities = this.activities.filter((d) => {
        return new Date(d.time) >= firstDay && new Date(d.time) < lastDay;
      });

      let dateInRange = this.getDatesInRange(firstDay, lastDay);

      let nameMonth = [
        "Januari",
        "Februari",
        "Maret",
        "April",
        "Mei",
        "Juni",
        "Juli",
        "Agustus",
        "September",
        "Oktober",
        "November",
        "Desember",
      ];

      let tempGroupDay = [
        { "01": [] },
        { "02": [] },
        { "03": [] },
        { "04": [] },
        { "05": [] },
        { "06": [] },
        { "07": [] },
        { "08": [] },
        { "09": [] },
        { 10: [] },
        { 11: [] },
        { 12: [] },
      ];
      let groupDay = [];

      for (let i = 0; i < tempGroupDay.length; i++) {
        groupDay[i] = tempGroupDay[i];
        groupDay[i][Object.keys(tempGroupDay[i])]["positive"] = 0;
        groupDay[i][Object.keys(tempGroupDay[i])]["negative"] = 0;
        groupDay[i][Object.keys(tempGroupDay[i])]["neutral"] = 0;
      }

      for (let i = 0; i < this.activities.length; i++) {
        for (let j = 0; j < groupDay.length; j++) {
          let nowDay = this.activities[i].time.substring(0, 2);
          let formatGroupDay = Object.keys(groupDay[j])[0];
          if (formatGroupDay == nowDay) {
            if (this.activities[i].value == "+") {
              groupDay[j][Object.keys(tempGroupDay[j])]["positive"] += 1;
            } else if (this.activities[i].value == "-") {
              groupDay[j][Object.keys(tempGroupDay[j])]["negative"] += 1;
            } else if (this.activities[i].value == "=") {
              groupDay[j][Object.keys(tempGroupDay[j])]["neutral"] += 1;
            }
            break;
          }
        }
      }
      this.groupDay = groupDay;
      this.groupDate = nameMonth;
      this.pieChart = await this.makeChart();
      this.chartKey += 1; // update chart by changing the key
      this.getTotalActivities();
    },

    async getTotalActivities() {
      this.totalPositiveActivities = Object.keys(
        this.activities.filter((d) => {
          return d.value == "+";
        })
      ).length;

      this.totalNegativeActivities = Object.keys(
        this.activities.filter((d) => {
          return d.value == "-";
        })
      ).length;

      this.totalNeutralActivities = Object.keys(
        this.activities.filter((d) => {
          return d.value == "=";
        })
      ).length;

      this.totalActivities = Object.keys(this.activities).length;
    },
  },
};
</script>

<template>
  <div class="grid grid-cols-12 gap-4 text-white">
    <div class="col-span-12 my-5 mx-auto">
      <button
        @click="getWeekActivities"
        :class="
          'mr-2 py-3 px-6' +
          (period == 'week'
            ? ' dark:bg-transparent border-2 border-indigo-800 text-indigo-800 dark:text-white dark:hover:bg-indigo-600 hover:border-indigo-600'
            : ' bg-indigo-800 dark:text-white hover:bg-indigo-600')
        "
      >
        Minggu Ini
      </button>
      <button
        @click="getMonthActivities"
        :class="
          'mr-2 py-3 px-6' +
          (period == 'month'
            ? ' dark:bg-transparent border-2 border-indigo-800 text-indigo-800 dark:text-white dark:hover:bg-indigo-600 hover:border-indigo-600'
            : ' bg-indigo-800 dark:text-white hover:bg-indigo-600')
        "
      >
        Bulan Ini
      </button>
      <button
        @click="getYearActivities"
        :class="
          'mr-2 py-3 px-6' +
          (period == 'year'
            ? ' dark:bg-transparent border-2 border-indigo-800 dark:text-white hover:bg-indigo-600 hover:border-indigo-600'
            : ' bg-indigo-800 dark:text-white hover:bg-indigo-600')
        "
      >
        Tahun Ini
      </button>
    </div>

    <div class="col-span-4 bg-emerald-600 rounded-lg">
      <div class="flex my-5">
        <div
          class="grid grid-rows-1 mx-5 p-7 my-auto bg-emerald-700 rounded-lg"
        >
          <font-awesome-icon icon="plus" class="text-2xl" />
        </div>
        <div class="grid grid-rows-2">
          <h2 class="text-lg">Positif</h2>
          <h2 class="text-4xl">{{ totalPositiveActivities }}</h2>
        </div>
      </div>
    </div>
    <div class="col-span-4 bg-rose-600 rounded-lg">
      <div class="flex my-5">
        <div class="grid grid-rows-1 mx-5 p-7 my-auto bg-rose-700 rounded-lg">
          <font-awesome-icon icon="minus" class="text-2xl" />
        </div>
        <div class="grid grid-rows-2">
          <h2 class="text-lg">Negatif</h2>
          <h2 class="text-4xl">{{ totalNegativeActivities }}</h2>
        </div>
      </div>
    </div>
    <div class="col-span-4 bg-indigo-600 rounded-lg">
      <div class="flex my-5">
        <div class="grid grid-rows-1 mx-5 p-7 my-auto bg-indigo-700 rounded-lg">
          <font-awesome-icon icon="equals" class="text-2xl" />
        </div>
        <div class="grid grid-rows-2">
          <h2 class="text-lg">Netral</h2>
          <h2 class="text-4xl">{{ totalNeutralActivities }}</h2>
        </div>
      </div>
    </div>

    <div
      style="
        height: 600px;
        width: 1000px;
        display: flex;
        flex-direction: column;
      "
      v-if="
        pieChart != null &&
        pieChart.type != undefined &&
        pieChart.data != undefined
      "
    >
      <vue3-chart-js
        v-bind="pieChart"
        :type="pieChart.type"
        :data="pieChart.data"
        :key="chartKey"
      />
    </div>
  </div>
</template>