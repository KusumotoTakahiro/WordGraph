<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="5" lg="5" xl="5">
        <!-- <my-speech-recognition ref="spRecog"></my-speech-recognition>
        <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
        <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn> -->
        <v-btn @click="start_from_excel()" color="green">エクセルを解析</v-btn>
        <v-btn @click="db_test()" color="yellow">db_test</v-btn>
        <input type="file" @change="onFileChange" />
        
      </v-col>
      <v-col cols="12" sm="12" md="7" lg="7" xl="7">
        
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import * as xlsx from 'xlsx';
import kuromoji from 'kuromoji'

export default {
  name: 'all_methods',
  data() {
    return {
      excel_data: null,
      talk_title: "test",

    }
  },
  methods: {
    start_from_excel() {
      let len = this.excel_data.length;
      for (let i = 1; i < len; i++) { //0番目は表の項目名のためデータは1番目から
        let sentence = this.excel_data[i][0];
        let speaker = this.excel_data[i][1];
        this.tokenize(sentence)
        .then(this.categolize)
        .then((res)=>{
          console.log(res);
        });
        // console.log('From MydataCenter');
        // await this.$refs.kuromoji.update_next(analysised_data); 
        // console.log('end update next');
        // await this.$refs.kuromoji.process_keyword(analysised_data, speaker);
        // console.log('end promise_keyword');
      }
    },
    db_test() {
      // this.$refs.kuromoji.db_test();
    },
    onFileChange(event) {
      this.file = event.target.files ? event.target.files[0] : null;
      if (this.file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const bstr = e.target.result;
          const wb = xlsx.read(bstr, {type: 'binary'});
          const wsname = wb.SheetNames[0];
          const ws = wb.Sheets[wsname];
          this.excel_data = xlsx.utils.sheet_to_json(ws, {header: 1}); 
        }
        reader.readAsBinaryString(this.file);
      }
    },
    tokenize(sentence) {
      return new Promise((resolve)=>{
        kuromoji.builder({ dicPath: '/dict' }).build((err, tokenizer) => {
          if (err) {
            console.log(err);
          }
          else {
            const tokens = tokenizer.tokenize(sentence);
            resolve(tokens);
          }
        })
      })
    },
    categolize(tokens) {
      return new Promise((resolve)=> {
        let analysised_data = [];
        for (let token of tokens) {
          switch (token.pos) {
            case '名詞':
              analysised_data.push({
                "keyword" : token.basic_form,
                "position": 'noun',
              });
              break;
            case '形容詞':
              analysised_data.push({
                "keyword": token.basic_form,
                "position": 'adjective',
              });
              break;
            case '副詞':
              analysised_data.push({
                "keyword": token.basic_form,
                "position": 'adverb',
              });
              break;
            case '動詞':
              analysised_data.push({
                "keyword": token.basic_form,
                "position": 'verb',
              });
              break;
            default:
              //その他の品詞として無視（格納しない)
          }
        }
        resolve(analysised_data);
      })
    },
    
    //databaseに新規単語をpushする．
    push_data(keyword, weight, before_next, after_next, isLatest, isInGraph, position, speaker) {
      return new Promise((resolve)=>{
        let vm = this;
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        talk_ref.child(keyword).set({
          "weight": weight,
          "before_next": before_next,
          "after_next": after_next,
          "isLatest": isLatest,
          "isInGraph": isInGraph,
          "position": position,
          "speaker": speaker,
        })
        console.log('保存したのは'+keyword);
      })
    },
    //databaseにある既出単語の更新を行う．
    update_data_weight(keyword, weight){
      return new Promise((resolve)=>{
        let vm = this;
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        talk_ref.child(keyword).update({
          "weight": weight,
        });
      })
    },
    update_data_bnext(keyword, b_next) {
      return new Promise((resolve)=>{
        let vm = this;
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        talk_ref.child(keyword).update({
          "before_next": b_next,
        });
      })
    },
    update_data_latest(keyword, flag){
      return new Promise((resolve)=>{
        let vm = this;
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        talk_ref.child(keyword).update({
          "isLatest": flag,
        });
      })
    },
    //databaseからdataを読み取る
    read_data_OOOO(){
    },
    //databaseから指定のtalk_titleの全データを削除する
    delete_db(talk_title) {
      return new Promise((resolve)=>{
        let vm = this;
        const talk_ref = this.$fire.database.ref('talks/'+talk_title);
        talk_ref.remove()
        console.log('全データ削除');
      })
    },
  }
}
</script>