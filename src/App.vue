<template>
  <RecordAudio/>
</template>

<script>
import RecordAudio from './components/RecordAudio.vue'

export default {
  name: 'App',
  components: {
    RecordAudio
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
