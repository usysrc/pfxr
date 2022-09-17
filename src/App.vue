<template>
  <v-app>
    <v-main>
      <SingleSprite />
    </v-main>
  </v-app>
</template>

<script>
import SingleSprite from "./components/SingleSprite";

export default {
  name: "App",

  components: {
    SingleSprite,
  },

  data: () => ({
    //
  }),
};
</script>
