<template>
  <v-container>
    <my-speech-recognition ref="spRecog"></my-speech-recognition>
    <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
    <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
    <v-btn @click="start_from_excel()" color="green">エクセルを解析</v-btn>
    <v-btn @click="db_test()" color="yellow">db_test</v-btn>
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
    update_next_resolve() {
      return new Promise(resolve => {
        console.log('start update next');
        resolve();
      })
    },
    process_keyword_resolve() {
      return new Promise(resolve=>{
        console.log('start process_keyword');
        resolve();
      })
    },
    //MyExcelReaderからexcel_dataを受け取り，一時的にexcel_dataの中に格納するメソッド
    get_excel_data(value) {
      this.excel_data = value;
    },
    //excelからのdataをkuromojiで解析してkeyword群に加工し，dbに格納し，graphを更新するメソッド.
    //要は全部を統括して実行するメソッド
    async start_from_excel() {
      let len = this.excel_data.length;
      for (let i = 1; i < len; i++) { //0番目は表の項目名のためデータは1番目から
        let sentence = this.excel_data[i][0];
        let speaker = this.excel_data[i][1];
        let analysised_data = this.$refs.kuromoji.analysis(sentence);
        console.log('From MydataCenter');
        await this.$refs.kuromoji.update_next(analysised_data); 
        console.log('end update next');
        await this.$refs.kuromoji.process_keyword(analysised_data, speaker);
        console.log('end promise_keyword');
      }
    },

    db_test() {
      this.$refs.kuromoji.db_test();
    }
  },
}
</script>
