<template>
  <q-page padding>
    <q-table
      v-if="tracksLoaded"
      grid
      :rows="rows"
      :columns="columns"
      row-key="name"
      hide-header
      :rows-per-page-options="[10]"
    >
      <template v-slot:item="props">
        <div class="q-pa-xs col-xs-12 col-sm-6 col-md-6">
          <iframe
            style="border-radius: 12px"
            :src="`https://open.spotify.com/embed/track/${props.row.id}`"
            width="100%"
            height="152"
            frameBorder="0"
            allowfullscreen=""
            allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
            loading="lazy"
          ></iframe>
        </div>
      </template>
    </q-table>

    <div v-else class="flex flex-center text text-center q-pa-lg">
      <div class="text-h1 gt-sm text-weight-light">
        No results to display 😔
      </div>
      <div class="text-h3 lt-md text-weight-light">
        No results to display 😔
      </div>
    </div>

    <div class="row justify-center q-gutter-md">
      <div class="col-md-2 col-11">
        <q-btn
          class="full-width"
          size="lg"
          rounded
          label="Return to search"
          style="background-color: #6b7b8f"
          @click="backToSearch()"
        />
      </div>
      <div class="col-md-2 col-11">
        <q-btn
          class="full-width"
          rounded
          label="Save this playlist"
          style="background-color: #1db954"
          size="lg"
          icon="fa-brands fa-spotify"
          @click="savePlaylist()"
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import { ref, toRaw } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import store from "src/store";

const columns = [
  {
    name: "desc",
    required: true,
    align: "left",
    field: (row) => row.name,
    sortable: true,
  },
];

export default {
  name: "ResultsPage",

  setup() {
    const router = useRouter();
    const rows = ref([]);
    const tracksLoaded = ref(true);

    rows.value = store.tracks.value;

    if (!rows.value) {
      tracksLoaded.value = false;
    }

    function backToSearch() {
      router.push("/");
    }

    async function savePlaylist() {
      const playlistId = await createNewPlaylist();
      addTracksToPlaylist(playlistId);
    }

    async function createNewPlaylist() {
      const userId = await getSpotifyUserId();

      let response = await axios.post(
        `https://api.spotify.com/v1/users/${userId}/playlists`,
        {
          name: store.playlistName.value,
          public: true,
        },
        {
          headers: {
            Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
          },
        }
      );

      return response.data.id;
    }

    async function getSpotifyUserId() {
      let response = await axios.get("https://api.spotify.com/v1/me", {
        headers: {
          Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
        },
      });

      return response.data.id;
    }

    async function addTracksToPlaylist(playlistId) {
      const trackUris = store.tracks.value.map((track) => {
        const rawTrackData = toRaw(track);
        return rawTrackData.uri;
      });
      console.log(trackUris);

      let response = await axios.post(
        `https://api.spotify.com/v1/playlists/${playlistId}/tracks`,
        {
          uris: trackUris,
        },
        {
          headers: {
            Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
          },
        }
      );

      console.log(response.data);
    }

    return {
      columns,
      rows,

      tracksLoaded,

      backToSearch,
      savePlaylist,
    };
  },
};
</script>
