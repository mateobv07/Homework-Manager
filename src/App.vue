<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        

        <h1 class="text-h4">Homework Tec</h1>
      </div>

      <v-spacer></v-spacer>

      <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="../src/assets/tec.png"
          transition="scale-transition"
          width="140"
        />
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    //
  }),
};
</script>
