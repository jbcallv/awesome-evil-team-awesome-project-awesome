<template>
  <q-page class="flex flex-center">
    <div v-if="isAuthenticated" class="column q-gutter-md">
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

    <div v-else class="column q-gutter-md">
      <div class="row">
        <div class="col-12">
          <q-btn
            rounded
            size="lg"
            icon="fa-brands fa-spotify"
            label="LOG INTO
          SPOTIFY"
            style="background-color: #1db954"
            @click="getAuthorizationToken()"
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useQuasar } from "quasar";
import axios from "axios";

export default defineComponent({
  name: "IndexPage",

  setup() {
    const $q = useQuasar();
    const router = useRouter();
    const route = useRoute();
    const isAuthenticated = ref(false);

    const songMixify = ref("");

    if (checkQueryParameters()) {
      getAccessToken();
    }

    function checkQueryParameters() {
      if (route.query.code) {
        return true;
      }
      return false;
    }

    function getAuthorizationToken() {
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

    function getAccessToken() {
      axios
        .get(`${import.meta.env.VITE_BACKEND_URL}/access-token`, {
          params: {
            code: route.query.code,
            redirect_uri: `${import.meta.env.VITE_SPOTIFY_REDIRECT_URI}`,
          },
        })
        .then((response) => {
          if (response.data.success) {
            $q.notify({
              icon: "fa-solid fa-check",
              message: "Authenticated with Spotify",
            });
            isAuthenticated.value = true;
            sessionStorage.setItem("accessToken", response.data.access_token);
          }
        });
    }

    function searchSpotify() {
      console.log(songMixify.value);
      axios
        .get(
          `https://api.spotify.com/v1/search?q=${songMixify.value}&type=playlist`,
          {
            headers: {
              Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
            },
          }
        )
        .then((response) => {
          const randomIndex = Math.floor(Math.random() * (15 - 0 + 1) + 0);
          console.log(response.data.playlists.items[randomIndex]);
          sessionStorage.setItem(
            "spotifyPlaylistId",
            response.data.playlists.items[randomIndex].id
          );
          router.push("/results");
        });
    }

    return {
      isAuthenticated,
      songMixify,

      getAuthorizationToken,
      searchSpotify,
    };
  },
});
</script>
