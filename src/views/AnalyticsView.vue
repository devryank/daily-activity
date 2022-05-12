<script>
export default {};
</script>

<template>
  <div class="grid grid-cols-12 gap-4 text-white">
    <div class="col-span-12 my-5 mx-auto">
      <a
        href=""
        class="
          mx-1
          py-3
          px-6
          dark:bg-transparent
          border-2 border-indigo-800
          dark:text-white dark:hover:bg-indigo-600 dark:hover:border-indigo-600
        "
      >
        Minggu Ini
      </a>
      <a
        href=""
        class="
          mx-1
          py-3
          px-6
          dark:bg-indigo-800 dark:text-white dark:hover:bg-indigo-600
        "
      >
        Bulan Ini
      </a>
      <a
        href=""
        class="
          mx-1
          py-3
          px-6
          dark:bg-indigo-800 dark:text-white dark:hover:bg-indigo-600
        "
      >
        Tahun Ini
      </a>
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
          <h2 class="text-4xl">230</h2>
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
          <h2 class="text-4xl">230</h2>
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
          <h2 class="text-4xl">230</h2>
        </div>
      </div>
    </div>
  </div>
</template>