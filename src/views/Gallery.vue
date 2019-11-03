<template>
  <div id="gallery">
    <h2>Gallery</h2>
    <section id="images"></section>
  </div>
</template>

<script>
export default {
  name: "gallery",

  mounted() {
    //get cloudname and preset from local storage
    // this.cloudname = this.$ls.get("cloudname")
    // this.preset = this.$ls.get("preset")
    //if these aren't set don't allow browse
    this.myGallery = cloudinary.galleryWidget({
      container: "#images",
      cloudName: "picturecloud7",
      mediaAssets: [{ tag: "browser_upload" }]
    });
    // this.myGallery = cloudinary.galleryWidget({
    //   container: "#images",
    //   cloudName: "demo",
    //   mediaAssets: [{ tag: "shoe", mediaType: "video" }]
    // });
    this.myGallery.render();
  },
  beforeDestroy: function() {
    this.myGallery.destroy;
  },
  data() {
    return {
      myGallery: null,
      cloudname: "",
      presest: ""
    };
  }
};
</script>

<styles scoped>
#images{
  width: 80%;
  height: 20%;
}
</style>
