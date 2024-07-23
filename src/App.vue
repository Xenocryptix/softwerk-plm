<template>
  <div id="app">
    <PartsTable/>
  </div>
</template>

<script>
import PartsTable from './components/PartsTable.vue'

export default {
  name: 'App',
  components: {
    PartsTable
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
