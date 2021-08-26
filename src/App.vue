<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        Proagro Fácil
      </div>

      <v-spacer></v-spacer>

    </v-app-bar>

    <v-main>
      <Ocorrencias/>
    </v-main>
  </v-app>
</template>

<script>
import Ocorrencias from './components/Ocorrencias';

export default {
  name: 'App',

  components: {
    Ocorrencias,
  },

  data: () => ({
    //
  }),
};
</script>
