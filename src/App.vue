<script setup>
import ReminderList from "./components/ReminderList.vue";
</script>

<template>
  <header>
    <h1>Ma liste de rappels</h1>
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
