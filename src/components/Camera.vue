<template>
  <div class="camera">
    <div class="actions">
      <b-button-group>
        <b-button :disabled="!isStartEnabled" v-on:click="start">Start</b-button>
        <b-button :disabled="isStartEnabled" v-on:click="stop">Stop</b-button>
        <b-button :disabled="isStartEnabled" v-on:click="snapshot">Snapsot</b-button>
        <b-button :disabled="!isPhoto" v-on:click="download">Download</b-button>
        <b-button
          :disabled="!isPhoto || settings.cloudname.length === 0 || settings.preset.length === 0"
          v-on:click="upload"
        >Upload</b-button>
      </b-button-group>
      <form class>
        <b-form-select
          v-model="selectedDevice"
          :options="options"
          v-on:change="deviceChange()"
          size="sm"
        ></b-form-select>
      </form>
    </div>

    <b-container>
      <b-row>
        <b-col sm="12" md="6">
          <video playsinline autoplay></video>
        </b-col>
        <b-col sm="12" md="6">
          <canvas></canvas>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
// https://cloudinary.com/documentation/image_upload_api_reference
// https://codepen.io/team/Cloudinary/pen/Edxjbj
// https://support.cloudinary.com/hc/en-us/articles/209754169-Is-it-possible-to-upload-directly-to-Cloudinary-using-pure-Javascript-code-
async function uploadToCloudinary(cloudName, preset, fileData) {
  try {
    let fd = new FormData();
    let url = `https://api.cloudinary.com/v1_1/${cloudName}/image/upload`;
    console.log(url);
    fd.append("upload_preset", preset);
    fd.append("tags", "browser_upload"); // Optional - add tag for image admin in Cloudinary
    fd.append("file", fileData);
    let res = await axios({
      method: "post",
      url: url,
      data: fd
    });
    return await res.data;
  } catch (err) {
    console.log("error upload:", err);
    throw err;
  }
}

// import cloudinary from "cloudinary-core"
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
      selectedDevice: null
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
      uploadToCloudinary(
        this.settings.cloudname,
        this.settings.preset,
        this.fileData
      )
        .then(result => {
          console.log(result);
          this.$bvToast.toast(`Upload to Cloudinary successful`, {
            title: "Cloudinary Upload",
            autoHideDelay: 5000,
            appendToast: false
          });
        })
        .catch(err => {
          this.$bvToast.toast(
            `Upload to Cloudinary unsuccessful. Check settings.`,
            {
              title: "Cloudinary Upload",
              autoHideDelay: 5000,
              appendToast: false
            }
          );
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
      //when starting up again use first option
      this.selectedDevice = this.options[0].value
      this.getMedia().then((this.isStartEnabled = false));
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
        window.stream = this.stream
        this.currentStream = stream
        this.video.srcObject = stream
        return true
      } catch (err) {
        throw err;
        console.log("getmedia", err);
      }
    },
    deviceChange: function() {
      this.stop();
      //don't change selected device
      this.setConstraints();
      this.getMedia().then((this.isStartEnabled = false));
    },
    setConstraints: function() {
      const videoContstraints = {};
      //set selected to highest option
      // this.selectedDevice = this.options[this.options.length - 1].value;

      if (this.selectedDevice === null) {
        videoContstraints.facingMode = "environment";
      } else {
        videoContstraints.deviceId = {
          exact: this.selectedDevice
        };
      }
      this.constraints = {
        video: videoContstraints,
        audio: false
      };
    },
    getDevices: async function() {
      if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
        console.log("enumerateDevices() not supported.");
        return false
      }
      try {
        let allDevices = await navigator.mediaDevices.enumerateDevices();
        for (let mediaDevice of allDevices) {
          // console.log("enumerate", mediaDevice)
          if (mediaDevice.kind === "videoinput") {
            let option = {};
            option.text = mediaDevice.label;
            option.value = mediaDevice.deviceId;
            this.options.push(option);
            this.devices.push(mediaDevice);
          }
        }
        return true;
      } catch (err) {
        console.log("getDevices", err);
        throw err;
      }
      console.log("devices", this.devices);
    }
  },
  mounted() {
    console.log("camera mounted cloudname", this.settings.cloudname);
    this.canvas = document.querySelector("canvas");
    this.video = document.querySelector("video");
    // this.options.push({ text: "Select Device", value: "", disable: true });
    this.getDevices()
      .then(res => {
        //when first loaded selected device can use 1st option
        this.selectedDevice = this.options[0].value
        this.setConstraints();
      })
      .then(res => {
        this.getMedia();
      })
      .then(res => {
        this.isStartEnabled = false;
      });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button:disabled {
  background: lightgray;
  color: black;
}
select option:disabled {
  color: lightgray;
  font-weight: bold;
}
form {
  margin: 1em;
}

@media only screen and (min-width: 600px) {
  .btn-group button {
    margin: 0 0.5em;
    border-radius: 1em;
    font-size: 0.5em;
  }
}
</style>
