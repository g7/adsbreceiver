<template>
  <div class="flighradar24">
    <FormKit
      :name="formKey('key')"
      label="Flightradar24 sharing key (if you have one)"
      :validation="[['matches', /^[a-fA-F0-9]{16}$/]]"
      :validation-messages="{
          matches: 'Invalid key format',
      }"
      validation-behavior="live"
      value=""
      placeholder="e.g. 4bd8df99c89e12e2"
      help="If you already configured Flightradar24 before, you can use the sharing key here. Otherwise you would need to do the initial registration (see below)"
    />

    <FormKit
      :name="formKey('do_initial_registration')"
      label="I don't have a key"
      type="checkbox"
      validation-behavior="live"
      v-model="do_initial_registration"
    />

  <div v-if="do_initial_registration === true">
    <div class="container-fluid">
    <div class="row">
    <div class="col-1"></div>
    <div class="col-12">
      <FormKit
        :name="formKey('mail')"
        label="Flightradar24 registration - Mail address"
        validation="[['required']]"
        validation-behavior="live"
        value=""
        placeholder="e.g. me@geeko.tld"
        help="If you HAVEN'T configured Flightradar24 before, specify your e-mail address here.
        It will be used for corrispondence with FR24. If you have a flightradar24.com account already, it's a good idea to use the same e-mail address, as that will allow your account to be upgraded automatically to a Business subscription. Flightradar24 will get the latitude, longitude and altitude details of your antenna."
      />
  </div>
  </div>
  </div>
  </div>
  </div>
</template>

<script>
import Utils from "../../utils/utils.js";
import { ref } from "vue";
const formPrefix = "fr24";

export default {
  setup: () => {
    const uid = Utils.uid();
    const do_initial_registration = ref(false);

    return {
      do_initial_registration,
      formKey: (key) => Utils.getFormKey(formPrefix, key, uid),
    };
  },

  methods: {
    encodeToInstallation: function (json, formData) {
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
          if (formData[key] != undefined) {
            json.combustion += "echo '" +
              key.split("_").slice(0,-1).join("_").toUpperCase() + "=" + formData[key] +
              "' >> /etc/fr24feeder.env\n"
            }
        });        
    },
    encodeToExport: function (json, formData) {
      json.fr24_data = {}
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
            json.fr24_data[key.split("_").slice(0,-1).join("_")] = formData[key]
        }); 
    },
    fillImport: function (json, formData) {
      if (json.fr24_data == undefined) return;
      Object.keys(formData)
          .filter((x) => x.includes(formPrefix) )
          .forEach((key) => {
             formData[key] = json.fr24_data[key.split("_").slice(0,-1).join("_")];
          });
    },
    countImport: function (json) {
      if (json.fr24_data != undefined) {
        return 1;
      } else {
        return 0;
      }
    },

  },
};
</script>
