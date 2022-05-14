<template>
  <v-row>
    <v-col class="text-center">
      <img
        src="/v.png"
        alt="Vuetify.js"
        class="mb-5"
      >
      <nuxt-logo></nuxt-logo>
      <tutorial></tutorial>
      <blockquote class="blockquote">
        &#8220;First, solve the problem. Then, write the code.&#8221;
        <footer>
          <small>
            <em>&mdash;John Johnson</em>
          </small>
        </footer>
      </blockquote>
    </v-col>
  </v-row>
</template>

<script>
import NuxtLogo from '../components/NuxtLogo.vue'
import Tutorial from '../components/Tutorial.vue'
export default {
  components: { NuxtLogo, Tutorial },
  name: 'InspirePage'
}
</script>
