<template>
  <div id="app">
    <Lottery/>
  </div>
</template>

<script>
import Lottery from './components/Lottery'

export default {
  name: 'App',
  components: {
    Lottery
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
