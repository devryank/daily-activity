<script>
export default {
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
    };
  },
  async created() {
    this.db = await this.getDb();
    this.activities = await this.getActivitiesFromDb();
    this.ready = true;
    await this.getWeekActivities(new Date(Date.now()));
    // var dateString = "05/13/2022";

    // console.log(new Date(dateString));
  },
  methods: {
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
    async getWeekActivities(date) {
      // If no date object supplied, use current date
      // Copy date so don't modify supplied date
      var now = date ? new Date(date) : new Date();

      // set time to some convenient value
      now.setHours(0, 0, 0, 0);

      // Get the previous Sunday
      var sunday = new Date(now);
      sunday.setDate(sunday.getDate() - sunday.getDay());

      // Get next Saturday
      var saturday = new Date(now);
      saturday.setDate(saturday.getDate() - saturday.getDay() + 6);

      // get this week activities
      this.activities = this.activities.filter((d) => {
        return new Date(d.time) >= sunday && new Date(d.time) < saturday;
      });

      this.getTotalAnctivities();
    },

    async getTotalAnctivities() {
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
        @click="getWeekActivities"
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
          <h2 class="text-lg">Positive</h2>
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
          <h2 class="text-lg">Negative</h2>
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
          <h2 class="text-lg">Neutral</h2>
          <h2 class="text-4xl">{{ totalNeutralActivities }}</h2>
        </div>
      </div>
    </div>
  </div>
</template>