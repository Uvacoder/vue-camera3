<template>
  <div class="home">
   
       
       
    <camera :cloudname="cloudname" :preset="preset"></camera>
  </div>
</template>

<script>
import Camera from '@/components/Camera.vue'
export default {
  name: 'Home',
  props: ['cloudname','preset'],
  components:{
    camera: Camera
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
