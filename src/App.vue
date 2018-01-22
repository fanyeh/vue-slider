<template lang="pug">
  <div id="app">
    <Slider/>
  </div>
</template>

<script>
import Slider from './components/Slider';

export default {
  name: 'App',
  components: {
    Slider,
  },
  methods: {
    enlarge(event) {
      event.currentTarget.classList.add('large');
    },
  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10vh;
}

body {
  margin:0;
  background-color:black;
}
</style>
