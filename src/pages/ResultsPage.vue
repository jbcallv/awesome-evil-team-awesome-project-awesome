<template>
  <q-page>
    <div v-if="playlistLoaded" class="q-pa-xl">
      <iframe
        style="border-radius: 12px"
        :src="`https://open.spotify.com/embed/playlist/${playlistId}`"
        width="100%"
        height="352"
        frameBorder="0"
        allowfullscreen=""
        allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture"
        loading="lazy"
      ></iframe>
    </div>

    <div v-else class="flex flex-center text text-center q-pa-lg">
      <div class="text-h1 gt-sm text-weight-light">
        No results to display 😔
      </div>
      <div class="text-h3 lt-md text-weight-light">
        No results to display 😔
      </div>
    </div>

    <div class="flex flex-center">
      <q-btn
        rounded
        label="Return to search page"
        style="background-color: #a94d5a"
        size="lg"
        @click="backToSearch()"
      />
    </div>
  </q-page>
</template>

<script>
import { ref } from "vue";
import { useRouter } from "vue-router";
export default {
  name: "ResultsPage",

  setup() {
    const router = useRouter();
    const playlistId = ref("");
    const playlistLoaded = ref(true);

    playlistId.value = sessionStorage.getItem("spotifyPlaylistId");

    if (!playlistId.value) {
      playlistLoaded.value = false;
    }

    function backToSearch() {
      router.push("/");
    }

    return {
      playlistId,
      playlistLoaded,

      backToSearch,
    };
  },
};
</script>
