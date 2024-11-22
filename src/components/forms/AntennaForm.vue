<template>
  <div class="antenna">
      <FormKit
        :name="formKey('lat')"
        label="Latitude of your antenna"
        :validation="[['matches', /^\-?\d\d?\.\d\d\d\d$/]]"
        :validation-messages="{
            matches: 'Latitude format is DD.DDDD',
        }"
        validation-behavior="live"
        value=""
        placeholder="format DD.DDDD"
        help="Specify your antenna latitude here. It will be shared with Flightradar24 and with MLAT servers if you choose to feed those. It will *NOT* be shared when doing Beast ADS-B data sharing."
      />

      <FormKit
        :name="formKey('long')"
        label="Longitude of your antenna"
        :validation="[['matches', /^\-?\d\d?\d?\.\d\d\d\d$/]]"
        :validation-messages="{
            matches: 'Longitude format is DDD.DDDD',
        }"
        validation-behavior="live"
        value=""
        placeholder="format DDD.DDDD"
        help="Specify your antenna longitude here. It will be shared with Flightradar24 and with MLAT servers if you choose to feed those. It will *NOT* be shared when doing Beast ADS-B data sharing."
      />

      <FormKit
        :name="formKey('alt')"
        label="Altitude of your antenna"
        :validation="[['matches', /^\d\d?\d?$/]]"
        :validation-messages="{
            matches: 'Altitude format is DDD',
        }"
        validation-behavior="live"
        value=""
        placeholder="altitude over sea level (in feet). format DDD"
        help="Specify your antenna altitude over sea level (in feet) here. It will be shared with Flightradar24 and with MLAT servers if you choose to feed those. It will *NOT* be shared when doing Beast ADS-B data sharing."
      />
  </div>
</template>

<script>
import Utils from "../../utils/utils.js";
import { ref } from "vue";
const formPrefix = "antenna";

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
    
      json.combustion += "\n# Antenna details\ncat > /etc/antenna.env <<EOF\n";

      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
          if (formData[key] != undefined) {
            json.combustion +=
              key.split("_").slice(0,-1).join("_").toUpperCase() + "=" + formData[key] + "\n"
          }
        });
        
      json.combustion += "EOF\n";
    },
    encodeToExport: function (json, formData) {
      json.antenna_details = {}
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
            json.antenna_details[key.split("_").slice(0,-1).join("_")] = formData[key]
        }); 
    },
    fillImport: function (json, formData) {
      if (json.antenna_details == undefined) return;
      Object.keys(formData)
          .filter((x) => x.includes(formPrefix) )
          .forEach((key) => {
             formData[key] = json.antenna_details[key.split("_").slice(0,-1).join("_")];
          });
    },
    countImport: function (json) {
      if (json.antenna_details != undefined) {
        return 1;
      } else {
        return 0;
      }
    },
  },
};
</script>
