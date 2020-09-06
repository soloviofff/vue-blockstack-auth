<template>
  <v-app id="app">
    <v-navigation-drawer
      class="deep-purple accent-4"
      v-model="primaryDrawer.model"
      :clipped="false"
      :floating="true"
      :mini-variant="primaryDrawer.mini"
      :permanent="false === 'permanent'"
      :temporary="true === 'temporary'"
      app
      overflow
      dark
    >
      <template v-slot:prepend v-if="user">
        <v-list-item two-line>
          <v-list-item-avatar v-if="avatar">
            <img :src="avatar" />
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title>{{username}}</v-list-item-title>
            <v-list-item-subtitle>Logged In</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </template>

      <v-divider></v-divider>

      <v-list v-if="userSession.isUserSignedIn()">
        <v-list-item
          v-for="item in navigation_items"
          :key="item.title"
          link
          :to="item.to"
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>

      <template v-slot:append>
        <div class="pa-2" v-if="userSession.isUserSignedIn()">
          <v-btn block @click.prevent="signOut">Logout</v-btn>
        </div>
        <div class="pa-2" v-else>
          <v-btn block @click.prevent="signIn">Log In</v-btn>
        </div>
      </template>

    </v-navigation-drawer>

    <v-app-bar
      :clipped-left="primaryDrawer.clipped"
      app
    >
      <v-app-bar-nav-icon
        v-if="primaryDrawer.type !== 'permanent'"
        @click.stop="primaryDrawer.model = !primaryDrawer.model"
      ></v-app-bar-nav-icon>
      <v-toolbar-title>Vuetify</v-toolbar-title>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>

    <v-footer :inset="true" app>
      <span class="px-4">&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import { Person } from 'blockstack';
import { userSession } from '@/plugins/userSession';
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    blockstack: window.blockstack,
    isChartActive: false,
    isJournalActive: false,
    primaryDrawer: {
      model: null,
      clipped: false,
    },
    navigation_items: [
      { title: 'Home', icon: 'mdi-view-dashboard', to: "/" },
      { title: 'About', icon: 'mdi-account-outline', to: "/about" },
      { title: 'Icons', icon: 'mdi-gavel', to: "/icons" }
    ],
    userSession: null,
    user: null,
    username: null,
    avatar: null
  }),
  created () {
    this.userSession = userSession
  },
  mounted () {
    if (userSession.isUserSignedIn()) {
      this.userData = userSession.loadUserData()
      this.user = new Person(this.userData.profile)
      this.username = this.userData.profile.name
      this.avatar = this.userData?.profile?.image[0]?.contentUrl || null

    } else if (userSession.isSignInPending()) {
      userSession.handlePendingSignIn()
        .then((userData) => {
          window.location = window.location.origin
          this.username = userData.profile.name
          this.avatar = userData?.profile?.image[0]?.contentUrl || null
        })
    }
  },
  methods: {
    signIn () {
      userSession.redirectToSignIn()
    },
    signOut () {
      userSession.signUserOut(window.location.href)
    }
  },
};
</script>
