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
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "IndexPage",

  setup() {
    const songMixify = ref("");

    return {
      songMixify,
    };
  },
});
</script>
