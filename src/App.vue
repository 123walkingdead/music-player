<template>
  <div id="app">
    <MusicDownload />
  </div>
</template>

<script setup>
import MusicDownload from './components/MusicDownload.vue'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

#app {
  font-family: 'Microsoft YaHei', 'PingFang SC', -apple-system, BlinkMacSystemFont, sans-serif;
  width: 100%;
  height: 100%;
}
</style>