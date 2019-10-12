<template>
  <div class="camera">
    <div class="action-group">
      <button id="start-stream" :disabled="!isStartEnabled" v-on:click="start">Start</button>
      <button id="stop-stream" :disabled="isStartEnabled" v-on:click="stop">Stop</button>
      <button id="snapshot" :disabled="isStartEnabled" v-on:click="snapshot">Snapshot</button>
      <button id="download" :disabled="!isPhoto" v-on:click="download">Download</button>
      <select :disabled="!isPhoto" class="u-full-width" id="video-select"></select>
    </div>
    <div id="container">
      <video playsinline autoplay></video>
      <canvas></canvas>
    </div>
    <form id="upload-form" class="upload-form">
      <input type="text" name="cloud-name" value="picturecloud7" placeholder="Enter cloud name" />
      <input type="text" name="preset" value="bp_test_1" placeholder="Enter upload preset" />
      <button type="submit">Upload</button>
    </form>
    <div id="gallery" />
    <a
      href="https://github.com/rebeccapeltz/vue-camera"
      title="View source for this page on GitHub"
      id="viewSource"
    >
      View
      source on GitHub
    </a>
  </div>
</template>

<script>
import CameraService from "@/components/CameraService";
const cameraService = new CameraService();
export default {
  name: "Camera",
  data() {
    return {
      camera: null,
      isStartEnabled: true,
      isPhoto: false
    };
  },
  methods: {
    snapshot: function() {
      console.log("snapshot");
      this.camera.snapshot();
      this.isPhoto = true;
    },
    stop: function() {
      console.log("stop");
      this.camera.turnAllDevicesOff();
      this.isStartEnabled = true
      this.isPhoto = false
    },
    start: function() {
      console.log("start");
      this.camera.turnOnDevice()
      this.isStartEnabled = false
    },
    download: function() {
      console.log("downlaod")
      this.camera.download()
    }
  },
  mounted() {
    const videoEl = document.querySelector("video")
    const canvasEl = document.querySelector("canvas")
    const deviceSelectEl = document.querySelector("#video-select")
    this.camera = cameraService.getSingleInstance(
      videoEl,
      canvasEl,
      deviceSelectEl
    );
    this.isStartEnabled = false
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button:disabled {
  background:gray;
}
select option:disabled {
    color:gray;
    font-weight: bold;
}
</style>
