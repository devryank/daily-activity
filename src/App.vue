
<script setup>
import { RouterLink, RouterView } from "vue-router";
import HelloWorld from "@/components/HelloWorld.vue";
</script>
<script>
export default {
  data() {
    return {
      path: "",
      DB_NAME: "acitivitydb",
      DB_VERSION: 1,
      db: null,
      ready: false,
      activities: [],
      time: "",
    };
  },
  created() {
    this.path = window.location.href.split("/")["3"];
    this.exportToJSON();
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

    async exportToJSON() {
      this.db = await this.getDb();
      this.activities = await this.getActivitiesFromDb();
      var data =
        "text/json;charset=utf-8," +
        encodeURIComponent(JSON.stringify(this.activities));

      var a = document.createElement("a");
      a.href = "data:" + data;
      a.download = "data.json";
      a.innerHTML = "download JSON";

      var container = document.getElementById("download");
      container.appendChild(a);
    },
  },
};
</script>

<template>
  <div class="grid grid-cols-1">
    <div class="col-span-1 mx-auto mt-10 mb-5">
      <a
        href="daily"
        :class="
          'mr-2 py-3 px-6' +
          (path == 'daily'
            ? ' dark:bg-transparent border-2 border-indigo-800 dark:text-white dark:hover:bg-indigo-600 dark:hover:border-indigo-600'
            : ' bg-indigo-800 text-white dark:hover:bg-indigo-600')
        "
      >
        Home
      </a>
      <a
        href="analytics"
        :class="
          'mr-2 py-3 px-6' +
          (path == 'analytics'
            ? ' dark:bg-transparent border-2 border-indigo-800 dark:text-white dark:hover:bg-indigo-600 dark:hover:border-indigo-600'
            : ' bg-indigo-800 text-white dark:hover:bg-indigo-600')
        "
      >
        Analytics
      </a>
      <span
        class="mr-2 py-3 px-6 bg-indigo-800 text-white dark:hover:bg-indigo-600"
        id="download"
      ></span>
    </div>
  </div>
  <div class="w-3/4 mx-auto relative mt-5">
    <RouterView />
  </div>
</template>

<style>
@import "@/assets/base.css";
@import url("https://fonts.googleapis.com/css2?family=Work+Sans:wght@100;300;500&display=swap");
body {
  font-family: "Work Sans", sans-serif;
}
</style>
