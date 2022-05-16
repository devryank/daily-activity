
<script setup>
import { RouterLink, RouterView } from "vue-router";
import HelloWorld from "@/components/HelloWorld.vue";
</script>
<script>
export default {
  data() {
    return {
      path: "",
    };
  },
  created() {
    this.path = window.location.href.split("/")["3"];
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
