<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card class="logo py-4 d-flex justify-center">
      </v-card>
      <v-card>
        <v-card-title class="headline">
          word graph
        </v-card-title>
        <v-card-text>
          <div>
            <flag iso="it" />
            <flag iso="gb" />
            <flag iso="us" />
          </div>
          
          <div>
            <p>{{finalTranscript}}</p>
            <p>{{keywords.keyword}}</p>
          </div>
          <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
          <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
          <v-btn @click="make_graph()" color="green">update graph</v-btn>
        </v-card-text>
        
        <v-card-actions>
          <!-- ここには遷移するためのボタンとかを置くと思われる -->
        </v-card-actions>
      </v-card>
    </v-col>
    <v-col>
    <div id="cy"></div>
    </v-col>
  </v-row>
</template>

<script>
import kuromoji from "kuromoji";
import cytoscape from 'cytoscape';
import axios from "axios";

export default {
  name: 'IndexPage',
  data() {
    return {
      recognition: null,
      finalTranscript: "",
      keywords : [{"keyword":'start', "weight":0, "isInGraph":true}], //ここがword graphのノードにもなる．
      theta : 0, //nodeの位置をΘで管理する．
    }
  },
  methods: {
    // //音声認識APIの初期化
    // async init_recog() {
    // },
    // //音声認識APIの実行
    // recog() {
    // },
    //keywordがすでに出たものかを確認する関数
    check_duplicate(keyword) {
      let index = -1;
      this.keywords.forEach(function(value, index){
        if (keyword == value.keyword) {
          index = index;
        }
      });
      return index;
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
              keyword = token.basic_form;
              let index = check_duplicate(keyword);
              if (index != -1) { //keywordが重複していた場合は重複した値のプロパティを更新する．
                this.keywords[index].weight += 1;
                this.keywords[index].isInGraph = false;
              }
              else {  //keywordが重複していなかった場合は新規でkeywordを追加する
                keywords.push({
                  "keyword":keyword, 
                  "weight":0, 
                  "isInGraph":false
                });
              }
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
    //グラフ初期化の関数
    init_graph() {
      this.cy = cytoscape({
        container : document.getElementById('cy'),
        elements : [
          {
            data : { id : 'start' }
          },
        ],
        style: [
          {
            selector: 'node',
            style: {
              'background-color': '#3cb371',
              'color': 'white',
              'label' : 'data(id)'
            }
          },
          {
            selector: 'edge',
            style: {
              'width':3,
              'line-color':'#ccc',
              'target-arrow-color':'#ccc',
              'target-arrow-shape':'triangle',
              'curve-style': 'bezier',
            }
          }
        ],
        layout: {
          name: 'grid',
          rows: 1
        }
      });
    }
  },
  mounted: function() {
    this.init_graph()
  }
}
</script>
<style scoped>
#cy {
    width: 100%;
    height: 80%;
    position: absolute;
    top: 50px;
    left: 0px;
    text-align: left;
}
</style>