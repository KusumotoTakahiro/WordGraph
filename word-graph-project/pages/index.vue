<template>
  <v-container>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
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
            <p>{{keywords}}</p>
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
  </v-row>
  <v-row>
    <div id="cy"></div>
  </v-row>
  </v-container>
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
      keywords : [
          {
            "keyword":'start', 
            "weight":0,
            "next":[],  //次の発言で出てくる単語
            "isLatest":true, //最新の発言であるか
            "isInGraph":true, //graphにnodeが含まれているか．
          },
        ], 
      theta : 0, //nodeの位置をΘで管理する．
      k : 100,
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
      let idx = -1;
      this.keywords.forEach(function(value, index){
        if (keyword == value.keyword) {
          idx = index;
          console.log("idx="+idx);
        }
      });
      console.log("idx="+idx);
      return idx;
    },
    //kuromoji.jsを使って形態素解析を行う．
    async analysis() {
      kuromoji.builder({ dicPath: "/dict" }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        } else {
          const tokens = tokenizer.tokenize(this.finalTranscript);
          console.log("finalTranscript = "+this.finalTranscript);
          //latestをもとにnextを更新
          for (let token of tokens) {
            if (tokens.pos=="名詞"){
              let keyword = token.basic_form;
              this.keywords.forEach(function(value){
                if (value.isLatest==true){
                  value.next.push(keyword);
                }
              })
            }
          }
          this.keywords.forEach(function(value){
            value.isLatest = false;
          })
          //keywordsの更新
          for (let token of tokens) {
            if (token.pos == "名詞") {
              let keyword = token.basic_form;
              let index = this.check_duplicate(keyword);
              //keywordが重複していた場合は重複した値のプロパティを更新する．
              if (index != -1) { 
                this.keywords[index].weight += 1;
                this.keywords[index].isLatest = true;
              }
              //keywordが重複していなかった場合は新規でkeywordを追加する
              else { 
                this.keywords.push({
                  "keyword":keyword, 
                  "weight":0, 
                  "next":[],
                  "isLatest":true,
                  "isInGraph":false,
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
    },
    update_nodes(keyword) {
      this.cy.add([
        {
          group : 'nodes',
          data : { id : keyword },
          position : {  x : 150 + Math.cos(this.theta)*this.k, y: 150 + Math.sin(this.theta)*this.k }
        }
      ]);
      this.k = 200 + Math.random()*100;
      this.theta += Math.PI/10;
    },
    update_edges(edge_id, source_node, target_node) {
      this.cy.add([
        {
          group : 'edges',
          data : {
            id : edge_id, 
            source : source_node,
            target : target_node,
          }
        }
      ]);
    },
    update_graph() {
      //先にキーワードをノードとして追加する
      this.keywords.forEach(function(value){
        if (value.isInGraph==false){
          update_nodes(value.keyword);
          value.isInGraph = true;
        }
      })
      //keywordsからedgeを生成する
      this.keywords.forEach(function(value){
        let next = value.next;
        let keyword = value.keyword;
        next.forEach(function(next_word){
          let id = keyword+next_word;
          if (this.cy.edges(id)==null){ //すでに存在していないか確認する
            this.update_edges(id, keyword, next_word);
          }
        })
      })
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
    top: 200px;
    left: 0px;
    text-align: left;
}
</style>