<template>
  <div id="app">
     <nav class="navbar navbar-light bg-light justify-content-between">
  <a class="navbar-brand">Camera</a>
  
  <div>
    <a class="settings" href=""><i class="settings fas fa-cog"></i></a>
    </div>
  <!-- </form> -->
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
a.settings {
  color: black;
}
.settings i { 
    font-size: 1rem;
    vertical-align: middle;
}
.brand {
  font-size: 3rem;
  font-weight: bold;
}
</style>
