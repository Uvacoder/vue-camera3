<template>
  <div class="camera">
    <div>
      <b-button-group>
        <b-button :disabled="!isStartEnabled" v-on:click="start">Start</b-button>
        <b-button :disabled="isStartEnabled" v-on:click="stop">Stop</b-button>
        <b-button :disabled="isStartEnabled" v-on:click="snapshot">Snapsot</b-button>
        <b-button :disabled="!isPhoto" v-on:click="download">Downlolad</b-button>
        <b-button
          :disabled="!isPhoto || settings.cloudname.length === 0 || settings.preset.length === 0"
          v-on:click="upload"
        >Upload</b-button>
      </b-button-group>
    </div>
    <form class>
      <b-form-select
        v-model="selectedDevice"
        :options="options"
        v-on:change="deviceChange()"
        size="sm"
      ></b-form-select>
    </form>

    <div id="container">
      <video playsinline autoplay></video>
      <canvas></canvas>
    </div>
    <!-- <form id="upload-form" class="upload-form">
      <input type="text" name="cloud-name" value="picturecloud7" placeholder="Enter cloud name" />
      <input type="text" name="preset" value="bp_test_1" placeholder="Enter upload preset" />
      <button type="submit">Upload</button>
    </form>
    <div id="gallery" />-->
  </div>
</template>

<script>
async function uploadToCloudinary(cloudName, preset, fileData) {
  try {
    let fd = new FormData();
    fd.append("upload_preset", preset);
    fd.append("tags", "browser_upload"); // Optional - add tag for image admin in Cloudinary
    fd.append("file", fileData);
    let res = await axios({
      method: "post",
      url: `https://api.cloudinary.com/v1_1/${cloudName}/upload`,
      data: fd
    });
    // console.log(res);
    return await res.data;
  } catch (err) {
    console.log("error upload:", err);
  }
}

// import cloudinary from "cloudinary-core";
import axios from "axios";
import { mapState } from "vuex";

export default {
  name: "Camera",
  data() {
    return {
      video: null,
      canvas: null,
      fileData: null,
      isStartEnabled: true,
      isPhoto: false,
      stream: null,
      currentStream: null,
      devices: [],
      options: [],
      constraints: {},
      selectedDevice: 0
      // cloudname: '',
      // preset:''
    };
  },
  computed: mapState(["settings"]),
  methods: {
    upload: function() {
      console.log("upload");

      if (
        this.settings.cloudname.length === 0 ||
        this.settings.preset.length === 0
      ) {
        console.log("error: upload missing cloudname or preset");
        this.$bvToast.toast(
          `Upload to Cloudinary unsuccessful: use settings to provide cloudname and preset`,
          {
            title: "Cloudinary Upload",
            autoHideDelay: 5000,
            appendToast: false
          }
        );
        return;
      }
      let data = uploadToCloudinary(
        this.settings.cloudname,
        this.settings.preset,
        this.fileData
      );

      this.$bvToast.toast(`Upload to Cloudinary successful`, {
        title: "Cloudinary Upload",
        autoHideDelay: 5000,
        appendToast: false
      });
    },
    snapshot: function() {
      console.log("snapshot");
      this.canvas.width = this.video.videoWidth;
      this.canvas.height = this.video.videoHeight;
      this.canvas
        .getContext("2d")
        .drawImage(this.video, 0, 0, this.canvas.width, this.canvas.height);
      this.fileData = this.canvas.toDataURL("image/jpeg");
      this.isPhoto = true;
    },
    stop: function() {
      console.log("stop");
      this.video.pause();
      if (this.currentStream) {
        this.currentStream.getTracks().forEach(track => {
          track.stop();
        });
        this.video.srcObject = null;
      }

      this.video.removeAttribute("src");
      this.video.load();
      this.canvas
        .getContext("2d")
        .clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.isStartEnabled = true;
      this.isPhoto = false;
    },
    start: function() {
      console.log("start");
      this.stop();
      this.getMedia();
      this.isStartEnabled = false;
    },
    download: function() {
      console.log("downlaod");
      if (this.fileData) {
        console.log("download");
        this.canvas.width = this.video.videoWidth;
        this.canvas.height = this.video.videoHeight;
        this.canvas
          .getContext("2d")
          .drawImage(this.video, 0, 0, this.canvas.width, this.canvas.height);
        let a = document.createElement("a");
        a.href = this.fileData;
        a.textContent = "photo";
        a.target = "_blank";
        a.download = "photo.jpeg";
        document.querySelector("body").append(a);
        a.click();
      }
    },
    getMedia: async function() {
      try {
        this.stream = await navigator.mediaDevices.getUserMedia(
          this.constraints
        );
        window.stream = this.stream;
        // this.video.srcObject = stream
        this.currentStream = stream;
        this.video.srcObject = stream;
      } catch (err) {
        console.log("getmedia", err);
      }
    },
    deviceChange: function(){
      this.getMedia()
      this.isStartEnabled = false
    },
    setConstraints: function() {
      const videoContstraints = {};
      if (this.selectedDevice === 0) {
        videoContstraints.facingMode = "environment";
      } else {
        videoContstraints.deviceId = {
          exact: this.selectedDevice.value
        };
      }
      this.constraints = {
        video: videoContstraints,
        audio: false
      }
    },
    getDevices: async function() {
      if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
        console.log("enumerateDevices() not supported.");
        return;
      }
      try {
        let allDevices = await navigator.mediaDevices.enumerateDevices();
        for (let mediaDevice of allDevices) {
          // console.log("enumerate", mediaDevice);
          if (mediaDevice.kind === "videoinput") {
            let option = {};
            option.text = mediaDevice.label;
            option.value = mediaDevice.deviceId;
            this.options.push(option);
            this.devices.push(mediaDevice);
          }
        }
        //default if only one

        if (this.options.length < 3)
          this.selectedDevice = this.options[1].value;
      } catch (err) {
        console.log("getDevices", err);
      }
      console.log("devices", this.devices);
    }
  },
  // props:['cloudname','preset'],
  // created: function() {
  //   this.$parent.$on("update-ls", function(payload) {
  //     this.cloudname = payload.cloudname
  //     this.preset = payload.preset
  //   });
  //},
  mounted() {
    //get cloudname and preset from local storage
    // this.cloudname = Vue.ls.get("cloudname")
    // this.preset = Vue.ls.get("preset")
    //if these aren't set don't allow upload

    console.log("camera mounted cloudname", this.settings.cloudname);
    this.canvas = document.querySelector("canvas");
    this.video = document.querySelector("video");
    this.options.push({ text: "Select Device", value: 0 });
    this.getDevices();
    this.setConstraints();
    this.getMedia();
    this.isStartEnabled = false;
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button:disabled {
  background: gray;
}
select option:disabled {
  color: gray;
  font-weight: bold;
}
select {
  margin: 0.2rem 0;
}

@media only screen and (max-width: 400px) {
  button {
    font-size: 0.6rem;
  }
}
</style>
