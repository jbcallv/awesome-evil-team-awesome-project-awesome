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
              :loading="isLoading"
              @click="searchSpotify()"
            />
          </div>
        </div>
      </div>

      <div v-else class="column q-gutter-md">
        <div class="row">
          <div class="col-12 text text-h2 text-center q-pb-md gt-sm">
            Connect to Spotify to start
            <span class="important-text">mixing</span>
          </div>

          <div
            class="col-12 text text-h4 text-weight-light text-center q-pb-md lt-md"
          >
            Connect to Spotify to start
            <span class="important-text">mixing</span>
          </div>
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
const stopWords = [
  "i",
  "me",
  "my",
  "myself",
  "we",
  "our",
  "ours",
  "ourselves",
  "you",
  "you're",
  "you've",
  "you'll",
  "you'd",
  "your",
  "yours",
  "yourself",
  "yourselves",
  "he",
  "him",
  "his",
  "himself",
  "she",
  "she's",
  "her",
  "hers",
  "herself",
  "it",
  "it's",
  "its",
  "itself",
  "they",
  "them",
  "their",
  "theirs",
  "themselves",
  "what",
  "which",
  "who",
  "whom",
  "this",
  "that",
  "that'll",
  "these",
  "those",
  "am",
  "is",
  "are",
  "was",
  "were",
  "be",
  "been",
  "being",
  "have",
  "has",
  "had",
  "having",
  "do",
  "does",
  "did",
  "doing",
  "a",
  "an",
  "the",
  "and",
  "but",
  "if",
  "or",
  "because",
  "as",
  "until",
  "while",
  "of",
  "at",
  "by",
  "for",
  "with",
  "about",
  "between",
  "into",
  "through",
  "during",
  "before",
  "after",
  "above",
  "below",
  "to",
  "from",
  "up",
  "down",
  "in",
  "out",
  "on",
  "off",
  "over",
  "under",
  "again",
  "further",
  "then",
  "once",
  "here",
  "there",
  "when",
  "where",
  "why",
  "how",
  "all",
  "any",
  "both",
  "each",
  "few",
  "more",
  "most",
  "other",
  "some",
  "such",
  "no",
  "nor",
  "not",
  "only",
  "own",
  "same",
  "so",
  "than",
  "too",
  "very",
  "s",
  "t",
  "can",
  "will",
  "just",
  "don",
  "don't",
  "should",
  "should've",
  "now",
  "d",
  "ll",
  "m",
  "o",
  "re",
  "ve",
  "y",
  "ain",
  "aren",
  "aren't",
  "couldn",
  "couldn't",
  "didn",
  "didn't",
  "doesn",
  "doesn't",
  "hadn",
  "hadn't",
  "hasn",
  "hasn't",
  "haven",
  "haven't",
  "isn",
  "isn't",
  "ma",
  "mightn",
  "mightn't",
  "mustn",
  "mustn't",
  "needn",
  "needn't",
  "shan",
  "shan't",
  "shouldn",
  "shouldn't",
  "wasn",
  "wasn't",
  "weren",
  "weren't",
  "won",
  "won't",
  "wouldn",
  "wouldn't",
];

export default defineComponent({
  name: "IndexPage",

  setup() {
    const $q = useQuasar();
    const router = useRouter();
    const route = useRoute();
    const isAuthenticated = ref(false);
    const isLoading = ref(false);

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
      isLoading.value = true;
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
        })
        .finally(() => {
          isLoading.value = false;
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
      //randomize
      const candidates = response.data.playlists.items.slice(0, 20);
      let items = [];
      for (let i = 0; i < 5; i++) {
        const randomIndex = Math.floor(
          Math.random() * (candidates.length - 1 - 0 + 1) + 0
        );
        items.push(candidates[randomIndex]);
      }
      return items;
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
      //split da words
      const value = songMixify.value;
      const words = value.split(" ");
      const trimmedString = words.filter((word) => !stopWords.includes(word));
      //join da words
      const finalString = trimmedString.join(" ");
      if (finalString === "") finalString = "100 gecs";
      return finalString;
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
      isLoading,

      getAuthorizationToken,
      searchSpotify,
    };
  },
});
</script>
