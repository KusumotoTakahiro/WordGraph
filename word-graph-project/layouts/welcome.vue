<template>
  <v-app dark>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer
      :absolute="!fixed"
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'for testing',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'refactoring',
          to: '/inspire'
        },
        {
          icon: 'mdi-graph-outline',
          title: 'word graph',
          to: '/sample'
        },
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Word Graph - ver1.0'
    }
  }
}
</script>
