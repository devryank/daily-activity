
<script>
import { onBeforeMount, onMounted } from "@vue/runtime-core";
export default {
  data() {
    return {
      DB_NAME: "acitivitydb",
      DB_VERSION: 1,
      db: null,
      ready: false,
      activities: [],
      totalActivities: 0,
      totalPositiveActivities: 0,
      totalNegativeActivities: 0,
      totalNeutralActivities: 0,

      data: {
        name: "",
        value: "",
      },
      errors: {
        name: "",
        value: "",
      },
      todayName: "",
      time: "",

      id: null,
      openEdit: false,
    };
  },
  async created() {
    this.db = await this.getDb();
    this.activities = await this.getActivitiesFromDb();
    this.ready = true;

    let days = [
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
    ];
    let timestamp = new Date(Date.now());
    this.todayName = days[timestamp.getDay()];
    this.time =
      ("0" + (timestamp.getMonth() + 1)).slice(-2) +
      "/" +
      ("0" + timestamp.getDate()).slice(-2) +
      "/" +
      timestamp.getFullYear();

    this.activities = this.activities.filter((d) => {
      return d.time == this.time;
    });
    console.log(timestamp.getDay());
    this.getTotalActivities();
  },
  methods: {
    async getId(id) {
      this.id = id;
      this.openEdit = true;

      const request = this.db
        .transaction("activities")
        .objectStore("activities")
        .get(id);

      request.onsuccess = () => {
        const activity = request.result;

        this.data.name = activity.name;
        this.data.value = activity.value;
      };

      request.onerror = (err) => {
        console.error(`Error to get activity information: ${err}`);
      };
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
        this.filterToday();
      });
    },

    async filterToday() {
      let timestamp = new Date(Date.now());
      this.time =
        ("0" + (timestamp.getMonth() + 1)).slice(-2) +
        "/" +
        ("0" + timestamp.getDate()).slice(-2) +
        "/" +
        timestamp.getFullYear();

      this.activities = this.activities.filter((d) => {
        return d.time == this.time;
      });
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

    async addDummyActivity() {
      this.errors.name = "";
      this.errors.value = "";

      let timestamp = new Date(Date.now());

      let dummy = {
        name: (Math.random() + 1).toString(36).substring(7),
        value: ["+", "-", "="][
          Math.floor(Math.random() * ["+", "-", "="].length)
        ],
        time:
          "04" +
          "/" +
          ("0" + timestamp.getDate()).slice(-2) +
          "/" +
          timestamp.getFullYear(),
      };

      console.log("about to add " + JSON.stringify(dummy));
      await this.addActivityToDb(dummy);

      this.activities = await this.getActivitiesFromDb();

      this.data.name = "";
      this.data.value = "";

      this.getTotalActivities();
    },

    async addActivity() {
      if (this.data.name == "") {
        this.errors.name = "Nama masing kosong";
      } else {
        this.errors.name = "";
      }

      if (
        this.data.value != "+" &&
        this.data.value != "-" &&
        this.data.value != "="
      ) {
        this.errors.value = `Value harus diisi dengan "+", "-", atau "="`;
      } else {
        this.errors.value = "";
      }

      if (this.errors.name == "" && this.errors.value == "") {
        this.errors.name = "";
        this.errors.value = "";

        let timestamp = new Date(Date.now());

        let activity = {
          name: this.data.name,
          value: this.data.value,
          time:
            ("0" + (timestamp.getMonth() + 1)).slice(-2) +
            "/" +
            ("0" + timestamp.getDate()).slice(-2) +
            "/" +
            timestamp.getFullYear(),
        };
        console.log("about to add " + JSON.stringify(activity));
        await this.addActivityToDb(activity);

        this.activities = await this.getActivitiesFromDb();
        this.filterToday();

        this.data.name = "";
        this.data.value = "";

        this.getTotalActivities();
      }
    },

    async addActivityToDb(activity) {
      return new Promise((resolve, reject) => {
        let trans = this.db.transaction(["activities"], "readwrite");
        trans.oncomplete = (e) => {
          resolve();
        };

        let store = trans.objectStore("activities");
        store.add(activity);
      });
    },

    async updateActivity(id) {
      if (this.data.name == "") {
        this.errors.name = "Nama masing kosong";
      } else {
        this.errors.name = "";
      }

      if (
        this.data.value != "+" &&
        this.data.value != "-" &&
        this.data.value != "="
      ) {
        this.errors.value = `Value harus diisi dengan "+", "-", atau "="`;
      } else {
        this.errors.value = "";
      }

      if (this.errors.name == "" && this.errors.value == "") {
        this.errors.name = "";
        this.errors.value = "";
        this.id = null;
        this.openEdit = false;

        await this.updateActivityFromDb(id);
        this.activities = await this.getActivitiesFromDb();
        this.filterToday();

        this.data.name = "";
        this.data.value = "";

        this.getTotalActivities();
      }
    },

    async updateActivityFromDb(id) {
      const objectStore = this.db
        .transaction("activities", "readwrite")
        .objectStore("activities");

      const request = objectStore.get(id);

      request.onsuccess = (e) => {
        const activity = e.target.result;

        activity.name = this.data.name;
        activity.value = this.data.value;
        activity.time = this.time;

        // Create a request to update
        const updateRequest = objectStore.put(activity);

        updateRequest.onsuccess = () => {
          console.log(`Acitivty updated`);
        };
      };
    },
    async deleteActivity(id) {
      await this.deleteActivityFromDb(id);
      this.activities = await this.getActivitiesFromDb();
      this.filterToday();

      this.getTotalActivities();
    },

    async deleteActivityFromDb(id) {
      const request = this.db
        .transaction("activities", "readwrite")
        .objectStore("activities")
        .delete(id);

      request.onsuccess = () => {
        console.log(`Activity deleted`);
      };

      request.onerror = (err) => {
        console.error(`Error to delete activity: ${err}`);
      };
    },
  },
};
</script>
<template>
  <div>
    <h2 class="text-xl mb-5">{{ todayName }}, {{ time }}</h2>
    <label for="Name">Name</label>
    <input type="text" v-model="data.name" class="mx-2 text-black" />
    <small class="text-red-600 text-sm">{{ errors.name }}</small>
    <br />
    <br />

    <label for="Value">Value</label>
    <input type="text" v-model="data.value" class="mx-2 text-black" />
    <small class="text-red-600 text-sm">{{ errors.value }}</small>
    <br />
    <button
      v-if="ready && !openEdit"
      @click="addActivity"
      class="
        dark:bg-indigo-800 dark:text-white dark:hover:bg-indigo-600
        py-2
        px-4
        my-3
      "
    >
      Tambah
    </button>

    <div v-if="ready && openEdit">
      <button
        @click="updateActivity(id)"
        class="
          dark:bg-indigo-800 dark:text-white dark:hover:bg-indigo-600
          py-2
          px-4
          my-3
          mr-2
        "
      >
        Edit
      </button>
      <button
        @click="openEdit = false"
        class="
          dark:bg-gray-800 dark:text-white dark:hover:bg-gray-600
          py-2
          px-4
          my-3
        "
      >
        Batal
      </button>
    </div>

    <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
      <thead
        class="
          text-xs text-gray-700
          uppercase
          bg-gray-50
          dark:bg-gray-700 dark:text-gray-400
        "
      >
        <tr>
          <th scope="col" class="px-6 py-3" width="50%">Daily Habits</th>
          <th scope="col" class="px-6 py-3">
            POSITIVE (+), NEGATIVE (-), OR NEUTRAL (=)
          </th>
          <th scope="col" class="px-6 py-3">
            <span class="sr-only">Edit</span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          class="
            border-b
            dark:bg-gray-800 dark:border-gray-700
            odd:bg-white
            even:bg-gray-50
            odd:dark:bg-gray-800
            even:dark:bg-gray-700
          "
          v-for="(activity, index) in activities"
          :key="index"
        >
          <th
            scope="row"
            class="
              px-6
              py-4
              font-medium
              text-gray-900
              dark:text-white
              whitespace-nowrap
            "
          >
            {{ activity.name }}
          </th>
          <td class="px-6 py-4">{{ activity.value }}</td>
          <td class="px-6 py-4 text-right">
            <a
              href="#"
              class="
                mr-2
                font-medium
                text-blue-600
                dark:text-blue-500
                hover:underline
              "
              @click="getId(activity.id)"
              >Edit</a
            >
            <a
              href="#"
              class="
                font-medium
                text-blue-600
                dark:text-blue-500
                hover:underline
              "
              @click="deleteActivity(activity.id)"
              >Delete</a
            >
          </td>
        </tr>
      </tbody>
    </table>

    <p class="mt-4">
      Positive: {{ totalPositiveActivities }}
      <br />
      Negative: {{ totalNegativeActivities }}
      <br />
      Neutral: {{ totalNeutralActivities }}
      <br />
      Total: {{ totalActivities }}
    </p>
  </div>
</template>