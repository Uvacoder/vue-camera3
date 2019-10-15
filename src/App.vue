<template>
  <div id="app">
    <nav class="navbar navbar-light bg-light justify-content-between">
      <a class="navbar-brand">Camera</a>

      <div>
        <b-button v-b-modal.modal-prevent-closing>
          <i class="settings fas fa-cog"></i>
        </b-button>

        <b-modal
          id="modal-prevent-closing"
          ref="modal"
          title="Cloudinary Upload Info"
          @show="resetModal"
          @hidden="resetModal"
          @ok="handleOk"
        >
          <form ref="form" @submit.stop.prevent="updateSettings">
            <b-form-group
              :state="cloudnameState"
              label="Cloudname"
              label-for="cloud-name-input"
              invalid-feedback="Cloudname is required"
            >
              <b-form-input
                id="cloud-name-input"
                v-model="cloudname"
                :state="cloudnameState"
                required
              ></b-form-input>
            </b-form-group>
            <b-form-group
              :state="presetState"
              label="Preset"
              label-for="preset-input"
              invalid-feedback="Preset is required"
            >
              <b-form-input id="preset-input" v-model="preset" :state="presetState" required></b-form-input>
            </b-form-group>
            <b-form-group>
              <b-form-checkbox
                id="clearsettings"
                v-model="clearsettings"
                name="clearsettings"
                value="true"
                unchecked-value="false"
              >Clear Settings</b-form-checkbox>
            </b-form-group>
          </form>
        </b-modal>

        <!-- <b-button v-b-modal.modal-prevent-closing>Open Modal</b-button> -->

        <!-- <div class="mt-3"> -->
        <!-- Submitted Names: -->
        <!-- <div v-if="submittedNames.length === 0">--</div> -->
        <!-- <ul v-else class="mb-0 pl-3">
        <li v-for="name in submittedNames">{{ name }}</li>
        </ul>-->
        <!-- </div> -->
      </div>
    </nav>
    <Home :cloudname="cloudname" :preset="preset" />
  </div>
</template>

<script>
import Home from "./views/Home.vue";

export default {
  name: "app",
  components: {
    Home
  },
  created: function() {
    if (this.$ls.get("cloudname")) {
      this.cloudname = this.$ls.get("cloudname");
      console.log("App.vue cloudname", this.cloudname);
    }
    if (this.$ls.get("preset")) {
      this.preset = this.$ls.get("preset");
    }
  },
  data() {
    return {
      cloudname: "",
      cloudnameState: null,
      preset: "",
      presetState: null,
      clearsettings: "false",
      expire: 3600000
    };
  },
  methods: {
    checkFormValidity() {
      // const valid = this.$refs.form.checkValidity();
      // this.cloudnameState = valid ? "valid" : "invalid";
      // return valid;
      return true;
    },
    resetModal() {
      this.cloudname = this.$ls.get("cloudname", "");
      this.cloudnameState = null;
      this.preset = this.$ls.get("preset", "");
      this.presetState = null;
    },
    handleOk(bvModalEvt) {
      // Prevent modal from closing
      bvModalEvt.preventDefault();
      // Trigger submit handler
      this.handleSubmit();
    },
    handleSubmit() {
      //clear local storage and return
      if (this.clearsettings === "true") {
        this.$ls.clear();
        this.cloudname = "";
        this.preset = "";
        this.clearsettings = "false"
        this.$nextTick(() => {
          this.$refs.modal.hide();
        });
        return;
      } else {
        // Exit when the form isn't valid
        if (!this.checkFormValidity()) {
          console.log("settings data not valid");
          return;
        }
      }

      // update local storage
      this.$ls.set("cloudname", this.cloudname, this.expire);
      this.$ls.set("preset", this.preset, this.expire);
      // Hide the modal manually
      this.$nextTick(() => {
        this.$refs.modal.hide();
      });
    }
  }
};
</script>

<style>
#app {
  font-family: "Roboto", "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px;  */
}
a.settings {
  color: black;
}
.settings i {
  font-size: 1rem;
}
.brand {
  font-size: 3rem;
  font-weight: bold;
}
</style>
