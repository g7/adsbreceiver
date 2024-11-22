<template>
  <div class="beast">
    <FormKit
      :name="formKey('feed_airplanes_live')"
      label="Feed ADS-B data to airplanes.live (Non-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
    <FormKit
      :name="formKey('feed_adsb_fi')"
      label="Feed ADS-B data to adsb.fi (Non-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
    <FormKit
      :name="formKey('feed_adsb_lol')"
      label="Feed ADS-B data to adsb.lol (Non-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
    <FormKit
      :name="formKey('feed_planespotters')"
      label="Feed ADS-B data to planespotters.net (Non-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
    <FormKit
      :name="formKey('feed_flyitalyadsb')"
      label="Feed ADS-B data to FlyItalyADSB (Non-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
    
		<hr class="divider-long" />

    <FormKit
      :name="formKey('feed_adsbexchange')"
      label="Feed ADS-B data to ADSBExchange (For-profit service)"
      type="checkbox"
      validation-behavior="live"
    />
  </div>
</template>

<script>
import Utils from "../../utils/utils.js";
import { ref } from "vue";
const formPrefix = "beast";

export default {
  setup: () => {
    const uid = Utils.uid();
    const feed_airplanes_live = ref(false);
    const feed_adsbexchange = ref(false);
    const feed_adsb_fi = ref(false);
    const feed_adsb_lol = ref(false);
    const feed_planespotters = ref(false);
    const feed_flyitalyadsb = ref(false);

    return {
      feed_airplanes_live,
      feed_adsbexchange,
      feed_adsb_fi,
      feed_adsb_lol,
      feed_planespotters,
      feed_flyitalyadsb,
      formKey: (key) => Utils.getFormKey(formPrefix, key, uid),
    };
  },

  methods: {
    encodeToInstallation: function (json, formData) {
    
      json.combustion += "\n# Beast sharing details\ncat > /etc/dumper.env <<EOF\n";

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
      json.beast_enabled = {}
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
            json.beast_enabled[key.split("_").slice(0,-1).join("_")] = formData[key]
        }); 
    },
    fillImport: function (json, formData) {
      if (json.beast_enabled == undefined) return;
      Object.keys(formData)
          .filter((x) => x.includes(formPrefix) )
          .forEach((key) => {
             formData[key] = json.beast_enabled[key.split("_").slice(0,-1).join("_")];
          });
    },
    countImport: function (json) {
      if (json.beast_enabled != undefined) {
        return 1;
      } else {
        return 0;
      }
    },
  },
};
</script>
