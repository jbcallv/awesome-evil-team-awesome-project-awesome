<template>
  <q-page class="flex flex-center">
    <div class="row full-width justify-center items-center">
      <div class="col-3 text-h1 text-weight-light text">Mix me a...</div>

      <div class="col-3">
        <div class="custom-input">
          <input
            :type="text"
            :value="value"
            @input="$emit('input', $event.target.value)"
          />
          <div class="underline"></div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "IndexPage",
});
</script>
