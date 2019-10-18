<template>
  <div id="gallery">
    <h2>Gallery</h2>
  </div>
</template>

<script>
export default {
  name: "gallery",

  mounted() {
    //get cloudname and preset from local storage
    this.cloudname = this.$ls.get("cloudname")
    this.preset = this.$ls.get("preset")
    //if these aren't set don't allow browse
  },
  data() {
    return {
        cloudname:"",
        presest:""
    };
  }
};
</script>

<style>

</style>
