<template>
  <v-container>
    <my-speech-recognition ref="spRecog"></my-speech-recognition>
    <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
    <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
    <v-btn @click="start_from_excel()" color="green">エクセルを解析</v-btn>
    <my-excel-reader @submit="get_excel_data"></my-excel-reader>
    <my-kuromoji ref="kuromoji"></my-kuromoji>
  </v-container>
</template>
<script>
import MyCytoscape from '../components/MyCytoscape.vue'
import MySpeechRecognition from '../components/MySpeechRecognition.vue'
import MyKuromoji from '../components/MyKuromoji.vue'
import MyExcelReader from './MyExcelReader.vue'

export default {
  components: { MyCytoscape, MySpeechRecognition, MyKuromoji, MyExcelReader },
  data() {
    return {
      transcript: '',
      excel_data: [],
    }
  },
  methods: {
    start_recog() {
      this.$refs.spRecog.talk_recog();
    },
    stop_recog() {
      this.$refs.spRecog.stop_recog();
    },
    get_excel_data(value) {
      this.excel_data = value;
      console.log(this.excel_data[1]);
      //this.start();
    },
    //excelからのdataをkuromojiで解析してkeyword群に加工し，dbに格納し，graphを更新するメソッド.
    //要は全部を統括して実行するメソッド
    start_from_excel() {
      let len = this.excel_data.length;
      for (let i = 1; i < len; i++) {
        let sentence = this.excel_data[i][0];
        let speaker = this.excel_data[i][1];
        //console.log(sentence + " : " + speaker);
        let analysised_data = this.$refs.kuromoji.analysis(sentence);
        console.log('From MydataCenter');
        console.log(analysised_data);
        this.$refs.kuromoji.update_next(analysised_data);
        this.$refs.kuromoji.process_keyword(analysised_data, speaker);
      }
    }
  },
}
</script>
