<template>
  <div id="app" class="flex flex-col h-full">
    <img v-if="showSplashscreen" class="z-10 absolute h-full object-cover" src="./assets/images/splashscreen.svg"/>
    <router-view v-show="!showSplashscreen" class="flex-1 regal-blue"/>
    <!-- Content -->
    <!-- Header -->
    <!-- Navigation -->
    <Navigation v-show="!showSplashscreen"/>
  </div>
</template>

<script>
import Navigation from './components/Navigation.vue'
import {restartWebsockets} from "vue-cli-plugin-apollo/graphql-client";

// import setRecipePreferencesForUser from "@/graphql/PostPreferences.gql"
// import gql from "graphql-tag"

export default {
  name: 'App',
  components: {
    Navigation
  },
  data() {
    return {
      splashscreenTimeRunning: true
    }
  },
  beforeCreate() {
    if (!this.isAuthenticated) {
      this.$store.dispatch(
          'userStorage/authRequest',
          this.$apolloProvider.defaultClient,
          {root: true}
      );
    }
  },
  // when app is created, do this
  created() {

    if (this.$route.name == 'Home'){
      setTimeout(function(){ 
      this.splashscreenTimeRunning = false }
      .bind(this), 5000)
    }
    else this.splashscreenTimeRunning = false
    this.$store.dispatch('settingsStorage/retrieveSettingsFromDB', this.$apolloProvider.defaultClient);
  },
  computed: {
    isAuthenticated() {
      return this.$store.getters['userStorage/authenticationStatus'];
    },
    firstApolloLoaded(){
      return this.$store.getters['splashscreenStorage/firstApolloLoaded'];
    },
    showSplashscreen(){
      return (this.splashscreenTimeRunning || !this.firstApolloLoaded) && (this.$route.name == 'Home')
    }
  },
  methods: {
    async resetAuthHeader(apolloClient) {
      if (apolloClient.wsClient) restartWebsockets(apolloClient.wsClient)
      try {
        await apolloClient.resetStore()
      } catch (e) {
        // eslint-disable-next-line no-console
        console.log('%cError on cache reset (login)', 'color: orange;', e.message)
      }
    }
  },

  // when app is destroyed, do this
  destroyed() {
    this.$store.dispatch('settingsStorage/updateSettingsInDB', this.$apolloProvider.defaultClient);
  }
}

</script>

<style>

#app {
  box-sizing: border-box;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #333333;
}
</style>


