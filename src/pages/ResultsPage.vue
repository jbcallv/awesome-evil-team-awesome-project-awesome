<template>
  <q-page flex flex-center>
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
  </q-page>
</template>

<script>
import { ref } from "vue";
export default {
  name: "ResultsPage",

  setup() {
    const playlistId = ref("");

    playlistId.value = sessionStorage.getItem("spotifyPlaylistId");
    console.log(playlistId.value);

    return {
      playlistId,
    };
  },
};
</script>
