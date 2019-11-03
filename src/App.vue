<template>
  <div id="app">
    <b-nav>
      <b-nav-item>
        <router-link to="/">Camera</router-link>
      </b-nav-item>
      <b-nav-item>
        <router-link to="/gallery">Gallery</router-link>
      </b-nav-item>

      <b-nav-item class="ml-auto">
        <div v-b-tooltip.hover title="Settings are required to upload photos.">
          <b-button class="settings" v-b-modal.modal-prevent-closing>
            <i class="fas fa-cog"></i>
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
        </div>
      </b-nav-item>
    </b-nav>
    <router-view></router-view>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { mapActions } from "vuex";

export default {
  name: "app",
  created: function() {
    console.log("App created", this.cloudname, this.preset);
    // if (this.$ls.get("cloudname")) {
    //   this.cloudname = this.$ls.get("cloudname");
    //   console.log("App.vue cloudname", this.cloudname);
    // }
    // if (this.$ls.get("preset")) {
    //   this.preset = this.$ls.get("preset");
    // }
  },
  data() {
    return {
      // cloudname: "",
      cloudnameState: null,
      // preset: "",
      presetState: null,
      clearsettings: "false",
      expire: 3600000
    };
  },
  // computed: mapState(['settings']),
  computed: {
    cloudname: {
      get() {
        return this.$store.state.settings.cloudname;
      },
      set(value) {
        this.$store.commit("updateCloudname", value);
      }
    },
    preset: {
      get() {
        return this.$store.state.settings.preset;
      },
      set(value) {
        this.$store.commit("updatePreset", value);
      }
    }
  },
  methods: {
    checkFormValidity() {
      // const valid = this.$refs.form.checkValidity();
      // this.cloudnameState = valid ? "valid" : "invalid";
      // return valid;
      return true;
    },
    resetModal() {
      this.cloudnameState = null;
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
        this.cloudname = "";
        this.preset = "";
        this.clearsettings = "false";
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
      // this.$ls.set("cloudname", this.cloudname, this.expire);
      // this.$ls.set("preset", this.preset, this.expire);

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
}
a.settings {
  color: black;
}


.settings i {
  font-size: 1rem;
}

.nav-item a a {
  color: black;
  font-size: 1.5rem;
}
input {
  text-transform: lowercase;
}
</style>
