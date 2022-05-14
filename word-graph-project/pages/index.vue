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
          <flag iso="it" />
          <flag iso="gb" />
          <flag iso="us" />
          ここにWordGraphを表示する．
          <p>{{finalTranscript}}</p>
          <p>{{keywords}}</p>
          <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
          <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
          <v-btn @click="analysis()" color="green">test analysis</v-btn>
        </v-card-text>
        <v-card-actions>
          <!-- ここには遷移するためのボタンとかを置くと思われる -->
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import kuromoji from "kuromoji";
import axios from "axios";

export default {
  name: 'IndexPage',
  data() {
    return {
      recognition: null,
      finalTranscript: "",
      keywords : [],
    }
  },
  methods: {
    //音声認識APIの初期化
    async init_recog() {
    },
    //音声認識APIの実行
    recog() {
    },
    //kuromoji.jsを使って形態素解析を行う．
    async analysis() {
      kuromoji.builder({ dicPath: "/dict" }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        } else {
          const tokens = tokenizer.tokenize(this.finalTranscript);
          console.log("finalTranscript = "+this.finalTranscript);
          for (let token of tokens) {
            if (token.pos == "名詞") {
              console.log("token: ", token.basic_form);
              this.keywords.push(token.basic_form);
            }
          }
        }
      });
    },
    //現時点で上記の音声認識関数が分割できないのでひとまとめにしている．
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
            this.finalTranscript = transcript;
            this.analysis();
          }
        }
      }
      this.recognition.start();
    },
    //音声認識開始を取りまとめる関数．ボタンと連携．
    start_recog() {
      this.talk_recog();
    },
    //音声認識終了を取りまとめる関数．ボタンと連携.
    stop_recog() {
      this.recognition.stop();
    },
  }
}
</script>
<style scoped>

</style>