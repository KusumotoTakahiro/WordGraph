<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="5" lg="5" xl="5">
        <my-data-center></my-data-center>
      </v-col>
      <v-col cols="12" sm="12" md="7" lg="7" xl="7">
        <my-cytoscape ></my-cytoscape>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import MyCytoscape from '../components/MyCytoscape.vue'
import MyDataCenter from '../components/MyDataCenter.vue'

export default {
  components: { MyCytoscape, MyDataCenter },
  name: 'InspirePage',
  data() {
    return {
    }
  },
}
</script>
