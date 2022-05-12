<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card class="logo py-4 d-flex justify-center">
        <!-- <NuxtLogo />
        <VuetifyLogo /> -->
      </v-card>
      <v-card>
        <v-card-title class="headline">
          word graph
        </v-card-title>
        <v-card-text>
          ここにWordGraphを表示する．
          <p>{{finalTranscript}}</p>
          <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
          <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
        </v-card-text>
        <v-card-actions>
          <!-- ここには遷移するためのボタンとかを置くと思われる -->
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
//import kuromoji from "kuromoji";
import axios from "axios";

export default {
  name: 'IndexPage',
  data() {
    return {
      recognition: null,
      interimTranscrip: "",
      finalTranscript: "",
    }
  },
  methods: {
    async init_recog() {
    },
    recog() {
    },
    analysis() {
    },
    talk_recog() {
      let SpeechRecog = window.webkitSpeechRecognition || window.webkitSpeechRecognition;
      this.recognition = new SpeechRecog();

      //recognitionの設定
      this.recognition.lang = "ja-JP";
      this.recognition.interimResults = true;
      this.recognition.continuous = true;

      this.recognition.onresult = (event) => {
        for (let i = event.resultIndex; i < event.results.length; i++) {
          let transcript = event.results[i][0].transcript;
          if (event.results[i].isFinal) {
            this.finalTranscript += transcript;
          }
          else {
            this.interimTranscript = transcript;
          }
        }
      }
      this.recognition.start();
    },
    start_recog() {
      this.talk_recog();
    },
    stop_recog() {
      this.recognition.stop();
    },
  }
}
</script>
<style scoped>

</style>