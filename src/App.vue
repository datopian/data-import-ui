<template>
  <div id="app">
    <DataImport></DataImport>
    <DataPreview></DataPreview>
  </div>
</template>

<script>
import DataImport from './components/DataImport'
import DataPreview from './components/DataPreview'

export default {
  name: 'app',
  components: {
    DataImport,
    DataPreview
  }
}
</script>

<style>
#app {
  font-family: 'Lato', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
