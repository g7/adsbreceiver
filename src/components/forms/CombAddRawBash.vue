<template>
  <div class="combustionraw">
    <FormKit
      :name="formKey('script_content')"
      label="Lines to add"
      type="textarea"
      validation="required"
      validation-behavior="live"
      :value="`echo &quot;hello combustion&quot;`"
      help="You can find examples of a combustion script here: https://en.opensuse.org/Portal:MicroOS/Combustion#Simple_example"
    />
  </div>
</template>

<script>
import Utils from "@/utils/utils.js";
const formPrefix = "comb_raw";

export default {
  setup: () => {
    const uid = Utils.uid();

    return {
      uid,
      formKey: (key) => Utils.getFormKey(formPrefix, key, uid),
    };
  },

  methods: {
    encodeToInstallation: function (json, formData) {
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
	  if (formData[key] != undefined) {
            json.combustion += "\n" + formData[key];
	  }
        });
    },
    encodeToExport: function (json, formData) {
      Object.keys(formData)
        .filter((x) => x.includes(formPrefix))
        .forEach((key) => {
          if (json.scripts === undefined) {
            json.scripts = [];
          }
	  if (formData[key] != undefined) {
	    json.scripts.push(formData[key])
	  }
	});
    },
    fillImport: function (json, formData) {
      if (json.scripts == undefined) return;
      Object.keys(formData)
          .filter((x) => x.includes(formPrefix) )
          .forEach((key) => {
	    formData[key] = json.scripts.shift();
          });
    },
    countImport: function (json) {
      if (json.scripts != undefined) {
        return json.scripts.length;
      } else {
        return 0;
      }
    },
  },
};
</script>
