
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

      name: "",
      value: "",
      todayName: "",
      time: "",
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
      ("0" + timestamp.getDate()).slice(-2) +
      "-" +
      ("0" + (timestamp.getMonth() + 1)).slice(-2) +
      "-" +
      timestamp.getFullYear();
  },
  methods: {
    async getId(id) {
      console.log(id);
      const request = this.db
        .transaction("activities")
        .objectStore("activities")
        .get(id);

      request.onsuccess = () => {
        const activity = request.result;

        this.name = activity.name;
        this.value = activity.value;
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
      });
    },

    async addActivity() {
      let timestamp = new Date(Date.now());
      let activity = {
        name: this.name,
        value: this.value,
        time:
          ("0" + timestamp.getDate()).slice(-2) +
          "-" +
          ("0" + (timestamp.getMonth() + 1)).slice(-2) +
          "-" +
          timestamp.getFullYear(),
      };
      console.log("about to add " + JSON.stringify(activity));
      await this.addActivityToDb(activity);
      this.activities = await this.getActivitiesFromDb();
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

    async deleteActivity(id) {
      await this.deleteActivityFromDb(id);
      this.activities = await this.getActivitiesFromDb();
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
    <div class="w-3/4 mx-auto relative overflow-x-auto shadow-md mt-10">
      <h2 class="text-xl mb-5">{{ todayName }}, {{ time }}</h2>
      <label for="Name">Name</label>
      <input type="text" v-model="name" class="mx-2 text-black" />
      <br />
      <br />

      <label for="Value">Value</label>
      <input type="text" v-model="value" class="mx-2 text-black" />
      <br />
      <button
        v-if="ready"
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
    </div>
  </div>
</template>