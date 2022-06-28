<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="5" lg="5" xl="5">
        <my-speech-recognition ref="spRecog"></my-speech-recognition>
        <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
        <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
        
      </v-col>
      <v-col cols="12" sm="12" md="7" lg="7" xl="7">
        <my-cytoscape ></my-cytoscape>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import MyCytoscape from '../components/MyCytoscape.vue'
import MySpeechRecognition from '../components/MySpeechRecognition.vue'
import MyKuromoji from '../components/MyKuromoji.vue'
import Mydatabase from '../components/Mydatabase.vue'
import DatabaseCheck from '../components/DatabaseCheck.vue'

export default {
  components: { MyCytoscape, MySpeechRecognition, MyKuromoji, Mydatabase, DatabaseCheck },
  name: 'InspirePage',
  data() {
    return {
    }
  },
  methods: {
    start_recog() {
      this.$refs.spRecog.talk_recog();
    },
    stop_recog() {
      this.$refs.spRecog.stop_recog();
    }
  },
}
</script>
