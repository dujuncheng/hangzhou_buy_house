<template>
  <div id="app">
      <hangzhouMap></hangzhouMap>
  </div>
</template>

<script>
    import hangzhouMap from './components/map.vue'
    export default {
      name: 'app',
      components: {
          hangzhouMap
      }
    }
</script>

<style>

    html, body {
        margin: 0;
        padding: 0;
    }
    body {
        width: 100%;
        height: 100%;
    }
    #app {
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
    }
    .amap-icon img {
        width: 22px !important;
        height: 22px !important;
        border-radius: 50%;
        border: 1px solid green;
    }
</style>
