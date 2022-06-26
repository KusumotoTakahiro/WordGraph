<template>
  <v-card>
    <v-card-title>
      音声認識の結果
    </v-card-title>
    <v-card-text style="margin:10px">{{finalTranscript}}</v-card-text>
  </v-card>
</template>
<script>
export default{
  data() {
    return {
      recognition: null,
      finalTranscript: "",
    }
  },
  methods: {
    talk_recog() {
      let vm = this;
      let SpeechRecog = window.webkitSpeechRecognition || window.webkitSpeechRecognition;
      vm.recognition = new SpeechRecog();
      //recognitionの設定
      vm.recognition.lang = "ja-JP";
      vm.recognition.interimResults = true;
      vm.recognition.continuous = true;
      vm.recognition.onresult = (event) => {
        for (let i = event.resultIndex; i < event.results.length; i++) {
          let transcript = event.results[i][0].transcript;
          if (event.results[i].isFinal) {
            vm.finalTranscript = transcript;
            //vm.analysis(); //ここで形態素解析すると依存関係が複雑化するから取り除いておく．
          }
        }
      }
      vm.recognition.start();
    },
    //音声認識終了を取りまとめる関数．ボタンと連携.
    stop_recog() {
      this.recognition.stop();
    },
    //音声認識開始を取りまとめる関数．ボタンと連携．
    // start_recog() {
    //   this.talk_recog();
    // },
  }
}
</script>
