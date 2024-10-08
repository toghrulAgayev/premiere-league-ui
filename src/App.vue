<template>
  <div id="app">
    <h1>Premier League</h1>
    <!-- Add League Table Component Here -->
    <LeagueTable />
  </div>
</template>

<script setup>
import LeagueTable from './components/LeagueTable.vue';
</script>

<style>
/* Add some global styles if necessary */
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  padding: 20px;
}
</style>
