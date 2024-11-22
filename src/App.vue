<script setup>
import { ref, watch } from "vue";
import Utils from "./utils/utils.js";
import Navigation from "./components/sections/NavSection.vue";
import About from "./components/sections/AboutSection.vue";
import EditView from "./pages/EditView.vue";
</script>

<template>
  <Navigation> </Navigation>

  <EditView></EditView>

  <!-- Footer-->
  <footer class="bg-light py-5">
    <div class="container px-4 px-lg-5">
      <div class="small text-center text-muted">
        Copyright &copy; {{ new Date().getFullYear() }} – built with 💚, Vue,
        and openSUSE<br /><br />
        
        This is a SUSE Hack Week 24 project by <a href="https://github.com/g7">Eugenio Paolantonio</a>. It is in no way related or sponsored
        to SUSE LLC or the openSUSE project.<br /><br />
        
        Heavily based on the awesome <a href="https://opensuse.github.io/fuel-ignition/">Fuel Ignition</a> web app
      </div>
    </div>
  </footer>
</template>
