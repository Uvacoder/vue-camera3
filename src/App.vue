<template>
  <div id="app">
      <nav class="row">
        <div class="two columns"><span class="brand">Camera</span></div>
        <div class="one  column">
          <button class="button"><i class="settings fas fa-cog"></i></button>
        </div>
        <div class="nine columns"></div>
      </nav>
    <Home />
  </div>
</template>

<script>
import Home from "./views/Home.vue";

export default {
  name: "app",
  components: {
    Home
  }
};
</script>

<style>
#app {
  /* font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px; */
}
.settings { 
    font-size: 3rem;
    vertical-align: middle;
}
.brand {
  font-size: 3rem;
  font-weight: bold;
}
</style>
