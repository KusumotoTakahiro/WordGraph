<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="5" lg="5" xl="5">
        <v-card>
          <v-card-title>
            音声認識の結果
          </v-card-title>
          <v-card-text style="margin:10px">{{finalTranscript}}</v-card-text>
        </v-card>
        <v-card>
          <v-card-title class="headline">
            Word Graph Table
          </v-card-title>
          <v-card-text>              
            <v-data-table
              :headers="headers"
              :items="keywords"
              class="elevation-1"
            >
            </v-data-table>
          </v-card-text>
        </v-card>
        <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
        <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn>
        <v-btn @click="update_graph()" color="green">グラフの更新</v-btn>
      </v-col>
      <v-col cols="12" sm="12" md="7" lg="7" xl="7" id="cy">
      </v-col>
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
      headers: [//テーブルのheaderの設定
        {
          text: 'Keywords', //列の名前
          align: 'start',
          sortable: false,
          value: 'keyword', //紐づける際のデータ名
        },
        {
          text : '重さ',
          value : 'weight',
        },
        {
          text : '次ノード(before)',
          value : 'before_next',
        },
        {
          text : '次ノード(after)',
          value : 'after_next',
        },
        {
          text : '最新の単語',
          value : 'isLatest'
        },
        {
          text : '描画済み',
          value : 'isInGraph'
        }
      ],
      recognition: null,
      finalTranscript: "",
      keywords : [
          {
            "keyword":'start', 
            "weight":0,
            "before_next":[], //次の単語(描画まえ)
            "after_next":[],  //次の単語(描画済み)
            "isLatest":true, //最新の発言であるか
            "isInGraph":true, //graphにnodeが含まれているか．
            "keywordID":0
          },
        ], 
      theta : 0, //nodeの位置をΘで管理する．
      k : 100,
      keywordID : 1,
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
        }
      });
      return idx;
    },
    //kuromoji.jsを使って形態素解析を行う．
    async analysis() {
      let vm = this;
      kuromoji.builder({ dicPath: "/dict" }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        } else {
          const tokens = tokenizer.tokenize(vm.finalTranscript);
          console.log("finalTranscript = "+vm.finalTranscript);
          //latestをもとにnextを更新
          for (let token of tokens) {
            if (token.pos=="名詞"){
              let keyword = token.basic_form;
              vm.keywords.forEach(function(value){
                if (value.isLatest==true){
                  value.before_next.push(keyword);
                }
              });
            }
          }
          //nextを更新し終えたのでisLatestを更新．
          vm.keywords.forEach(function(value){
            value.isLatest = false;
          })
          //keywordsの更新
          for (let token of tokens) {
            if (token.pos == "名詞") {
              let keyword = token.basic_form;
              let index = vm.check_duplicate(keyword);
              //keywordが重複していた場合は重複した値のプロパティを更新する．
              if (index != -1) { 
                vm.keywords[index].weight += 1;
                vm.keywords[index].isLatest = true;
              }
              //keywordが重複していなかった場合は新規でkeywordを追加する
              else { 
                vm.keywords.push({
                  "keyword":keyword, 
                  "weight":0, 
                  "before_next":[],
                  "after_next":[],
                  "isLatest":true,
                  "isInGraph":false,
                  "keywordID":vm.keywordID,
                });
                vm.keywordID += 1;
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
      this.recognition.onerror = (event) => {
        if (event.error == "no-speech") {
          recognition.start();
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
              'label' : 'data(id)',
              //'text-valign': 'center',
            }
          },
          {
            selector: 'edge',
            style: {
              'width':3,
              'line-color':'#eedcb3',
              'target-arrow-color':'#eedcb3',
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
          position : {  
            x : 150 + Math.cos(this.theta)*this.k, 
            y : 150 + Math.sin(this.theta)*this.k 
          }
        }
      ]);
      this.k = 200 + Math.random()*100;
      this.theta += Math.PI/10;
    },
    update_edges(source_node, target_node) {
      this.cy.add([
        {
          group : 'edges',
          data : {
            source : source_node,
            target : target_node,
          }
        }
      ]);
    },
    update_graph() {
      let vm = this;
      //先にキーワードをノードとして追加する
      vm.keywords.forEach(function(value){
        if (value.isInGraph==false){
          vm.update_nodes(value.keyword);
          value.isInGraph = true;
        }
      });
      
      //keywordsからedgeを生成する
      vm.keywords.forEach(function(value){
        let before_next_temp = value.before_next;
        let after_next = value.after_next;
        let keyword = value.keyword;
        
        //重複する値を除去する
        let before_next = before_next_temp.filter(function(ele, pos){
          return before_next_temp.indexOf(ele)==pos;
        })

        for (let i = 0; i < before_next.length; i++) {
          let is_drawn = false;
          //描画済みのedgeではないか確認する
          for (let j = 0; j < after_next.length; j++) {
            if (before_next[i]==after_next[j]){
              is_drawn = true;
            }
          }
          if (is_drawn==false) {
            vm.update_edges(keyword, before_next[i]);
          }
        }
        //before_nextをafter_nextに更新する.重複する分は今のところ無視
        value.after_next = value.after_next.concat(before_next);
        value.before_next = [];
      })
    },
    zoom_func() {
      let client_w = document.getElementById('cy').clientWidth;
      let client_h = document.getElementById('cy').clientHeight;
      this.cy.zoom({
        level : 2.0,
        renderedPosition: {x: client_w/2, y: client_h/2.5},
      })
    }
  },
  mounted: function() {
    this.init_graph();
    this.zoom_func();
  }
}
</script>
<style scoped>
#cy {
    background-color: darkblue;
    heigth: 100vh;
}
</style>