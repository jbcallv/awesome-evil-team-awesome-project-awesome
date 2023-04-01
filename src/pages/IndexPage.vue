<template>
  <transition
    appear
    enter-active-class="animated fadeIn"
    leave-active-class="animated fadeOut"
  >
    <q-page class="flex flex-center">
      <div v-if="isAuthenticated" class="column q-gutter-md">
        <div class="row">
          <div
            class="col-12 q-pa-md text text-h1 text-weight-light gt-sm q-pl-xl"
          >
            Mixify me a...
          </div>

          <div class="col-12 text-center text text-h3 text-weight-light lt-md">
            Mixify me a...
          </div>
        </div>

        <div class="row">
          <div class="col-1 q-pr-md gt-sm"></div>
          <!--for md+ screens-->
          <div class="col-7 q-pr-xl q-pa-xl q-pl-xl gt-sm">
            <q-input
              v-model="songMixify"
              :input-style="{
                fontSize: '50px',
                textAlign: 'center',
                color: '#a94d5a',
              }"
            >
              <!-- <template v-slot:append>
              <div class="col-12 text text-h4">Playlist</div>
            </template> -->
            </q-input>
          </div>
          <div class="col-4 text text-h1 q-pr-xl gt-sm">playlist</div>

          <!--for small screens-->
          <div class="col-12 q-pa-md lt-md">
            <q-input
              v-model="songMixify"
              :input-style="{ fontSize: '30px', textAlign: 'center' }"
            >
              <!-- <template v-slot:append>
              <div class="col-12 text text-h4">Playlist</div>
            </template> -->
            </q-input>
          </div>
          <div class="col-12 text text-h3 text-center lt-md">playlist</div>
        </div>

        <div class="row">
          <div class="col-12 text-center q-pa-sm">
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
        <!-- <div class="row">
        <div class="col-9 text text-h1 text-left">
          Welcome To
        </div>
        <div class="col-3 text text-h1 text-left">
          Welcome To
        </div>
      </div> -->
        <div class="row">
          <div class="col-10 text text-h2 text-center q-pb-md">
            Connect to Spotify to start
          </div>
          <div class="col-2 important-text text-h2 text-center">mixing</div>
        </div>
        <div class="row">
          <div class="col-12 flex flex-center">
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
  </transition>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useQuasar } from "quasar";
import store from "src/store";
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
      store.playlistName.value = songMixify.value;

      axios
        .get(
          `https://api.spotify.com/v1/search?q=${songMixify.value}&type=playlist`,
          {
            headers: {
              Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
            },
          }
        )
        .then(async (response) => {
          const randomIndex = Math.floor(
            Math.random() * (response.data.playlists.items.length - 1 - 0 + 1) +
              0
          );

          const playlist = response.data.playlists.items[randomIndex];

          let playlistTracks = await searchPlaylistTracks(playlist.id);
          let spotifyTracks = await searchSpotifyTracks();

          let tracks = mergeTracks(playlistTracks, spotifyTracks);

          store.tracks.value = tracks;

          router.push("/results");
        });
    }

    async function searchPlaylistTracks(playlistId) {
      let response = await axios.get(
        `https://api.spotify.com/v1/playlists/${playlistId}/tracks`,
        {
          headers: {
            Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
          },
        }
      );

      return response.data.items.slice(0, 5);
    }

    async function searchSpotifyTracks() {
      const trackQuery = filterUserQuery();
      let response = await axios.get(
        `https://api.spotify.com/v1/search?q=${trackQuery}&type=track`,
        {
          headers: {
            Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
          },
        }
      );

      return response.data.tracks.items.slice(0, 5);
    }

    function filterUserQuery() {
      return "test";
    }

    function mergeTracks(playlistTracks, spotifyTracks) {
      const filteredPlaylistTracks = playlistTracks.map((track) => {
        return { id: track.track.id, uri: track.track.uri };
      });
      const filteredSpotifyTracks = spotifyTracks.map((track) => {
        return { id: track.id, uri: track.uri };
      });

      const tracks = [...filteredPlaylistTracks, ...filteredSpotifyTracks];
      return tracks;
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
