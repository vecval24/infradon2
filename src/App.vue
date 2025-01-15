<script setup>
import ReminderList from "./components/ReminderList.vue";
</script>

<template>
  <header>
    <img
      alt="Vue logo"
      class="logo"
      src="./assets/logo.svg"
      width="125"
      height="125"
    />
  </header>
  <main>
    <ReminderList />
  </main>
</template>

<style scoped>
header {
  text-align: center;
  margin-bottom: 2rem;
}

.logo {
  display: block;
  margin: 0 auto;
}
</style>
