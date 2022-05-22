<template>
  <v-container>
  <v-row class="grey lighten-3" style="height: 600px;">
    <v-col cols="12" sm="8" md="6" lg="6" xl="6" style="background-color: #FFCDD2">
      column 1
    </v-col>
    <v-col cols="12" sm="8" md="6" lg="6" xl="6" style="background-color: #F8BBD0">
      column 2
    </v-col>
  </v-row>
</v-container>
</template>

<script>
import NuxtLogo from '../components/NuxtLogo.vue'
import Tutorial from '../components/Tutorial.vue'
export default {
  components: { NuxtLogo, Tutorial },
  name: 'InspirePage'
}
</script>
