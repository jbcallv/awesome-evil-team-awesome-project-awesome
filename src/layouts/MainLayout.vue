<template>
  <q-layout class="dark-container" view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <div class="text-h3 text-weight-light q-pa-lg">
          <span class="important-text">Mixify</span>
          <span class="text">Me</span>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "MainLayout",
});
</script>
