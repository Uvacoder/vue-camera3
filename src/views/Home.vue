<template>
  <div class="home">
    <camera></camera>
  </div>
</template>

<script>
import Camera from "@/components/Camera.vue";
export default {
  name: "Home",
  data() {
    return {
      // cloudname: "",
      // preset: ""
    };
  },
  components: {
    camera: Camera
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
