<template>
  <ExchangeWidget />
</template>

<script>
  import ExchangeWidget from './component/ExchangeWidget.vue'

  export default {
    name: 'App',
    components: {
      ExchangeWidget
    }
  }
</script>

<style>
  #app {
    font-family: Vollkorn, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>

<head>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Vollkorn">
</head>