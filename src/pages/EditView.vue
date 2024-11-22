<script setup>

import { ref, watch, onUpdated } from "vue";
import Utils from "@/utils/utils.js";
import ExpandableComponent from "@/components/ExpandableComponent.vue";
import BlobEditorComponent from "@/components/TemplateBlobEditorComponent.vue";

import AddUsersForm from "@/components/forms/AddUsersForm.vue";
import CreateFileForm from "@/components/forms/CreateFileForm.vue";
import AddHostnameForm from "@/components/forms/AddHostnameForm.vue";
import AddLanguageForm from "@/components/forms/AddLanguageForm.vue";
import AddNetworkForm from "@/components/forms/AddNetworkForm.vue";
import ChangeStorageForm from "@/components/forms/ChangeStorageForm.vue";
import StartServiceForm from "@/components/forms/StartServiceForm.vue";
import ModifyServiceForm from "@/components/forms/ModifyServiceForm.vue";

import DebugAddBytesForm from "@/components/forms/DebugAddBytesForm.vue";
import DebugAnalyzeImgForm from "@/components/forms/DebugAnalyzeImgForm.vue";
import AddKeyboardForm from "@/components/forms/AddKeyboardForm.vue";
import AddTimezoneForm from "@/components/forms/AddTimezoneForm.vue";
import InstallPackageForm from "@/components/forms/InstallPackageForm.vue";
import CombAddRawBash from "../components/forms/CombAddRawBash.vue";

import Flightradar24Form from "../components/forms/Flightradar24Form.vue";
import AntennaForm from "../components/forms/AntennaForm.vue";
import BeastForm from "../components/forms/BeastForm.vue";

const formComponents = [
  AddUsersForm,
  CreateFileForm,
  AddHostnameForm,
  AddLanguageForm,
  ChangeStorageForm,
  AddNetworkForm,
  StartServiceForm,
  ModifyServiceForm,
  AddKeyboardForm,
  AddTimezoneForm,
  InstallPackageForm,
  CombAddRawBash,

  Flightradar24Form,
  AntennaForm,
  BeastForm,

  DebugAddBytesForm,
  DebugAnalyzeImgForm,
];

const formData = ref({ debug: false, save_to: "adsbreceiver.json" });
const importedData = ref({})

// setup optional Watchers if a FormComponent needs it
formComponents.forEach((comp) =>
  Utils.setupFormComponentWatcher(comp, watch, formData)
);

let componentKey = ref(0);
const forceRerender = () =>{
   componentKey.value += 1;
}

const elementNumber = (section) => {
  if (section.methods.hasOwnProperty("countImport")) {
    return section.methods.countImport(importedData.value);
  } else {
    return 0
  }
}

onUpdated(() => {
  if (Object.keys(importedData.value).length > 0) {
    formComponents
      .filter((comp) => comp.methods.hasOwnProperty("fillImport"))
      .forEach((comp) => comp.methods.fillImport(importedData.value, formData.value));
    importedData.value = {};
  }
});

const downloadConfigIgn = (formData) => {
  console.log("downloading..");
  Utils.saveTemplateAsFile("config.ign", toIgnitionConfig(formData));
};

const copyConfigToClipboard = (formData) => {
  Utils.copy(JSON.stringify(toIgnitionConfig(formData), null, 2));
};

function copyToClipboard(text) {
  window.prompt("Copy to clipboard: Ctrl+C, Enter", text);
}

const toIgnitionConfig = (formData) => {
  let json = {
    ignition: { version: "3.2.0" },
  };

  formComponents
    .filter((comp) => comp.methods.hasOwnProperty("encodeToInstallation"))
    .forEach((comp) => comp.methods.encodeToInstallation(json, formData));

  if (formData.debug) {
    json["debug:form"] = formData;
  }

  json.combustion = undefined;
  json.combustion_initrd_and_running_system = undefined;
  json.combustion_initrd = undefined;

  return json;
};

const toCombustionScript = (formData) => {
  let json = { combustion: "", combustion_initrd: "", combustion_initrd_and_running_system: "", output: "" };

  formComponents
    .filter((comp) => comp.methods.hasOwnProperty("encodeToInstallation"))
    .forEach((comp) => comp.methods.encodeToInstallation(json, formData));

  if (json.combustion_initrd_and_running_system != "" || json.combustion_initrd != "") {
    if (json.output == "") {
      json.output =
        "#!/bin/bash\n# combustion: network prepare\n# script generated with https://opensuse.github.io/fuel-ignition/\n\n"
    }

    if (json.combustion_initrd_and_running_system != "") {
      console.log("initrd and running system: " + json.combustion_initrd_and_running_system);
      json.output += json.combustion_initrd_and_running_system + "\n"
    }

    json.output += "if [ \"${1-}\" = \"--prepare\" ]; then\n"
    if (json.combustion_initrd != "") {
      console.log("initrd: " + json.combustion_initrd);
      json.output += json.combustion_initrd + "\n"
    }

    json.output += "  exit 0\n" +
      "fi\n"
  } else {
    json.output = "#!/bin/bash\n# combustion: network\n# script generated with https://opensuse.github.io/fuel-ignition/\n"
  }

  json.output +=
    "\n# Redirect output to the console\n" +
    "exec > >(exec tee -a /dev/tty0) 2>&1\n"

  json.combustion = ""
  json.combustion_initrd_and_running_system = ""
  json.combustion_initrd = ""
  formComponents
    .filter((comp) => comp.methods.hasOwnProperty("encodeToInstallation"))
    .forEach((comp) => comp.methods.encodeToInstallation(json, formData));

  if (json.combustion !== "") {
    console.log(json.combustion);
    json.output += json.combustion
  }

  json.output += 'zypper --non-interactive --gpg-auto-import-keys ar --refresh --check https://download.opensuse.org/repositories/home:/epaolantonio:/adsbreceiver/openSUSE_Tumbleweed/home:epaolantonio:adsbreceiver.repo\n';
  json.output += 'zypper --non-interactive --gpg-auto-import-keys refresh\nzypper --non-interactive install adsbreceiver\n';
  json.output += 'systemctl enable cockpit.socket\n';
  json.output += 'systemctl enable chrony-wait.service\n';
  json.output += 'setsebool -P container_use_devices=true\n';
  json.output += 'echo "MACHINE_UUID=$(uuidgen -r)" > /etc/adsbreceiver-uuid\n';

  json.output += '\n# Leave a marker\necho "Configured with combustion" > /etc/issue.d/combustion';

  json.output += '\n\n# Close outputs and wait for tee to finish.\nexec 1>&- 2>&-; wait;';

  return json.output;
};

async function importData(event) {
  let file = event.target.files[0];

  // Load settings form file and
  // parse json from file object.
  importedData.value = await Utils.parseJsonFile(file)
    .catch((jsonErrorMsg) => {
      alert(jsonErrorMsg);
      Utils.clearFile();
    });
  formData.value = {debug: formData.value.debug, save_to: formData.value.save_to}
  forceRerender();
}

const exportSettings= (formData) => {
  let json = {};

  formComponents
    .filter((comp) => comp.methods.hasOwnProperty("encodeToExport"))
    .forEach((comp) => comp.methods.encodeToExport(json, formData));

    Utils.saveTemplateAsFile(
      formData.save_to,
      json,
      false // isNotJson parameter
    )
};

</script>

<template>
  <div :key="componentKey">
    <section class="page-section p-2 bg-dark" id="edit">
      <div class="container mt-5 px-0">
        <div class="row gx-4 gx-lg-5 justify-content-center">
          <div class="col-lg-8 col-xl-6 text-white text-center">
            <h1 class="mt-5">ADS-B MicroOS Config Generator</h1>
            <div class="d-grid mb-5">
              <p style="font-size: 100px">🛫</p>
            </div>
          </div>
        </div>
        <div class="row gx-4 gx-lg-5 justify-content-center">
          <div class="col-lg-8 col-xl-6 text-white text-right">
              This web app allows you to configure a simple ADS-B receiver based
              on openSUSE MicroOS and openSUSE Tumbleweed-based containers.<br /><br />

              <a href="https://github.com/wiedehopf/tar1090">tar1090</a> is the built-in web interface.<br /><br />

              If you wish so, you can also set-up feeding to some non-profit and for-profit aggregators, including Flightradar24.<br /><br />

              This has been only tested with a Raspberry Pi 3 Model B so far.<br /><br />

              What is needed:<br /><br />

              <ul> 
                <li>One RTL-SDR capable ADS-B receiver (RTL2832U DVB-T dongles might work)</li>
                <li>One Raspberry Pi or other SBC supported by openSUSE MicroOS</li>
                <li>One microSD card where to install openSUSE MicroOS</li>
                <li>One USB drive to use for the initial ignition+combustion bootstrap</li>
                <li>Working, wired internet connection</li>
              </ul>

              To start, generate a configuration from this page, convert the configuration to an image by pressing the 'Convert and Download' button and
              flash it in an USB drive.<br /><br />

              Then, flash the <a href="https://download.opensuse.org/ports/aarch64/tumbleweed/appliances/openSUSE-MicroOS.aarch64-ContainerHost-RaspberryPi.raw.xz">RaspberryPi MicroOS Base System + Container Runtime Environment image</a> in a microSD.<br /><br />

              Plug the USB drive, the RTL-SDR capable ADS-B receiver (if possible) and the MicroOS microSD in the Raspberry Pi. Ensure to connect the Raspbery Pi to the internet using a wired cable.<br /><br />

              If you weren't able to plug the ADS-B receiver during the initial bootstrap (due to the small space available), once Cockpit is up, unplug the USB drive and plug the receiver instead. Then reboot, and everything should now be properly set-up.<br /><br />

              Start the Pi, and wait ~15 minutes. If everything went right, you will be able to access the tar1090 interface at port :8080 and the Cockpit administration interface at port :9090.<br /><br />

              This interface is based on <a href="https://opensuse.github.io/fuel-ignition/">Fuel Ignition</a> web app.
            <hr class="divider-long" />

          </div>

        </div>
  
        <div class="row gx-4 gx-lg-5 justify-content-center mb-5">
          <div class="col-lg-6">
            <div class="form-floating mb-3">
              <FormKit type="group" v-model="formData">
                <ExpandableComponent
	        title="Add User"
  	        :displaysAtBegin="elementNumber(AddUsersForm)"
  	      >
                  <template #default="{index}">
                    <AddUsersForm :index="index"></AddUsersForm>
                  </template>
                </ExpandableComponent>

                <ExpandableComponent
                  title="Set Hostname"
                  :displaysAtBegin="elementNumber(AddHostnameForm)"
		  :maxComponents="1"
                >
                  <AddHostnameForm></AddHostnameForm>
                </ExpandableComponent>

		<hr class="divider-long" />

                <ExpandableComponent
                  title="Set Language"
                  :displaysAtBegin="elementNumber(AddLanguageForm)"
  		:maxComponents="1"
                >
                  <AddLanguageForm></AddLanguageForm>
                </ExpandableComponent>

		<ExpandableComponent
                  title="Set Keyboard"
                  :displaysAtBegin="elementNumber(AddKeyboardForm)"
		  :maxComponents="1"
                >
                  <AddKeyboardForm></AddKeyboardForm>
                </ExpandableComponent>

		<ExpandableComponent
                  title="Set Timezone"
                  :displaysAtBegin="elementNumber(AddTimezoneForm)"
		  :maxComponents="1"
                >
                  <AddTimezoneForm></AddTimezoneForm>
                </ExpandableComponent>

		<hr class="divider-long" />

                <ExpandableComponent
                  title="Add Network Interface"
                  :displaysAtBegin="elementNumber(AddNetworkForm)"
                >
                  <AddNetworkForm></AddNetworkForm>
                </ExpandableComponent>

		<hr class="divider-long" />

              <div style="width: 100% !important" class="text-white">
                <h4>Antenna details</h4>

                <div>
                  <AntennaForm></AntennaForm>
                </div>
              </div>

		<hr class="divider-long" />

                <ExpandableComponent
                  title="Feed ADS-B data to Beast-compatible aggregators"
                  :displaysAtBegin="elementNumber(BeastForm)"
                  :maxComponents="1"
                >
                  <BeastForm></BeastForm>
                </ExpandableComponent>

		<hr class="divider-long" />

                <ExpandableComponent
                  title="Enable Flightradar24 data sharing (for-profit service)"
                  :displaysAtBegin="elementNumber(Flightradar24Form)"
                  :maxComponents="1"
                >
                  <Flightradar24Form></Flightradar24Form>
                </ExpandableComponent>

		<hr class="divider-long" />
  
                <ExpandableComponent
                  title="Enable or Create Service"
                  :displaysAtBegin="elementNumber(StartServiceForm)"
                >
                  <StartServiceForm></StartServiceForm>
                </ExpandableComponent>
  
                <ExpandableComponent
                  title="Modify Existing Service"
                  :displaysAtBegin="elementNumber(ModifyServiceForm)"
                >
                  <ModifyServiceForm></ModifyServiceForm>
                </ExpandableComponent>
  
		<hr class="divider-long" />

                <ExpandableComponent
                  title="Install Additional Packages"
                  :displaysAtBegin="elementNumber(InstallPackageForm)"
                >
                  <InstallPackageForm></InstallPackageForm>
                </ExpandableComponent>

                <ExpandableComponent
                  title="Add Files To System"
                  :displaysAtBegin="elementNumber(CreateFileForm)"
                >
                  <CreateFileForm></CreateFileForm>
                </ExpandableComponent>

                <ExpandableComponent
                  title="Add Custom Lines To Combustion Script"
                  :displaysAtBegin="elementNumber(CombAddRawBash)"
                >
                  <CombAddRawBash></CombAddRawBash>
                </ExpandableComponent>

                <div v-if="formData.debug">
                  <!-- rename to "Modify Services (Drop-Ins)"? -->
                  <ExpandableComponent
                    title="DEBUG: Add Bytes"
                    :displaysAtBegin="elementNumber(DebugAddBytesForm)"
                  >
                    <DebugAddBytesForm></DebugAddBytesForm>
                  </ExpandableComponent>
  
                  <ExpandableComponent
                    title="DEBUG: Analyze File"
                    :displaysAtBegin="1"
                    :maxComponents="1"
                  >
                    <DebugAnalyzeImgForm></DebugAnalyzeImgForm>
                  </ExpandableComponent>
                </div>
              </FormKit>
            </div>
          </div>
        </div>
      </div>
    </section>
  
    <section class="page-section p-2" id="export">
      <div class="container mt-5 px-0">
        <div class="row gx-4 gx-lg-5 justify-content-center">
          <div class="col-lg-8 col-xl-6">
            <h1 class="mt-5 text-center">config.ign</h1>
            <div class="d-grid mb-5">
              <pre class="form-data">{{ toIgnitionConfig(formData) }}</pre>
  
              <div class="double">
                <FormKit
                  v-model="formData.debug"
                  type="checkbox"
                  label="Debug"
                  name="debug"
                />
  
                <button
                  class="btn btn-outline-secondary mb-2"
                  @click="copyConfigToClipboard(formData)"
                >
                  Copy
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-clipboard"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="M4 1.5H3a2 2 0 0 0-2 2V14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V3.5a2 2 0 0 0-2-2h-1v1h1a1 1 0 0 1 1 1V14a1 1 0 0 1-1 1H3a1 1 0 0 1-1-1V3.5a1 1 0 0 1 1-1h1v-1z"
                    />
                    <path
                      d="M9.5 1a.5.5 0 0 1 .5.5v1a.5.5 0 0 1-.5.5h-3a.5.5 0 0 1-.5-.5v-1a.5.5 0 0 1 .5-.5h3zm-3-1A1.5 1.5 0 0 0 5 1.5v1A1.5 1.5 0 0 0 6.5 4h3A1.5 1.5 0 0 0 11 2.5v-1A1.5 1.5 0 0 0 9.5 0h-3z"
                    />
                  </svg>
                </button>
              </div>
  
              <button
                class="btn btn-primary mb-4"
                @click="downloadConfigIgn(formData)"
		data-testid=download_ignition
              >
                Download
              </button>
  
              <div v-if="toCombustionScript(formData) !== undefined">
                <h1 class="mt-5 text-center">combustion script</h1>
                <hr class="divider" />
                <div class="d-grid mb-5">
                  <pre class="form-data">{{ toCombustionScript(formData) }}</pre>
  
                  <button
                    class="btn btn-primary mb-4"
		    data-testid=download_combustion
                    @click="
                      Utils.saveTemplateAsFile(
                        'script',
                        toCombustionScript(formData),
                        true // isNotJson parameter
                      )
                    "
                  >
                    Download
                  </button>
                </div>
              </div>
  
              <div>
                <hr class="divider" />
                <h2 class="mt-5 text-center">
                  Convert to Ignition-<span
                    v-if="toCombustionScript(formData) !== undefined"
                    >Combustion-</span
                  >Ready Filesystem IMG in the Browser
                </h2>
  
                <BlobEditorComponent
                  :ignJson="toIgnitionConfig(formData)"
                  :combustionScript="toCombustionScript(formData)"
                ></BlobEditorComponent>
              </div>
  
              <div>
                <h2 class="mt-5 text-center">
                  Convert to ISO Filesystem with mkisofs
                </h2>
  
                <pre class="form-data">
  Using ignition file only:
  # mkisofs -full-iso9660-filenames -o ignition.iso -V ignition -root ignition config.ign

  Using ignition and combustion configuration files:
  The files has to be stored under
   - fuel-ignition/combustion/script
   - fuel-ignition/ignition/config.ign
  # mkisofs -full-iso9660-filenames -o ignition.iso -V ignition fuel-ignition </pre>
                <p>
                  How to use the generated data with <a href="https://documentation.suse.com/sle-micro/6.0/html/Micro-deployment-raw-images/index.html#deployment-configuring-with-ignition" target="_blank">ignition</a> and <a href="https://documentation.suse.com/sle-micro/6.0/html/Micro-deployment-raw-images/index.html#deployment-configuring-with-combustion" target="_blank">combustion</a> .
                </p>
                <hr class="divider" />
	      </div>
  
              <h2 class="mt-5 text-center">Load Settings from</h2>
              <FormKit
                name="load_from"
                type="file"
  	      @change="importData"
              />
  
              <h2 class="mt-5 text-center">Save Settings to</h2>
              <FormKit
                v-model="formData.save_to"
                type="text"
                name="save_to"
              />
  
              <button
                class="btn btn-primary mt-3"
                @click="exportSettings(formData)"
              >
                Save
              </button>
  
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
