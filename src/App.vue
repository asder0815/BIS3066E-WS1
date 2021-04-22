<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h3>Big Data Management - Workshop #1</h3>
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/asder0815/BIS3066E-WS1"
        target="_blank"
        text
      >
        <span class="mr-2">Github Repository</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <HelloWorld :movieData="movie_data" />
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';
import movieData from './assets/movie_data.json'; 

export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    movie_data: movieData
  }),
};
</script>
