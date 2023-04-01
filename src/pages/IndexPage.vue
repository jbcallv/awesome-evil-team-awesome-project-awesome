<template>
  <q-page class="flex flex-center">
    <div class="column q-gutter-md">
      <div class="row">
        <div class="col-12 text text-h1 text-weight-light gt-sm">
          Mixify me a...
        </div>

        <div class="col-12 text text-h3 text-weight-light lt-md">
          Mixify me a...
        </div>
      </div>

      <div class="row">
        <div class="col-12">
          <q-input label="Song" v-model="songMixify" />
        </div>
      </div>

      <div class="row">
        <div class="col-6">
          <q-btn
            rounded
            size="lg"
            label="Mixify"
            style="background-color: #a94d5a"
            @click="searchSpotify()"
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import axios from "axios";

export default defineComponent({
  name: "IndexPage",

  setup() {
    const router = useRouter();
    const route = useRoute();
    const songMixify = ref("");

    function searchSpotify() {
      axios
        .get(`${import.meta.env.VITE_BACKEND_URL}/login`, {
          params: {
            redirect_uri: import.meta.env.VITE_SPOTIFY_REDIRECT_URI,
          },
        })
        .then((response) => {
          window.location = response.data;
        });
    }

    return {
      songMixify,

      searchSpotify,
    };
  },
});
</script>
