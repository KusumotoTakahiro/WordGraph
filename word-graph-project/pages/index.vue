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
          <flag iso="it" />
          <flag iso="gb" />
          <flag iso="us" />
          <div id="cy"></div>
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
      keywords : [],
      cy_nodes : [],
      cy_edges : [],
      cy_count : 0, //graphの頂点数(頂点のid)を管理する変数
      arg_x : 0,
      arg_y : 0,
      edge_id : 0,
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
    //グラフ初期化の関数
    init_graph() {
      this.cy = cytoscape({
        container : document.getElementById('cy'),
        elements : [],
        style: [
          {
            selector: 'node',
            style: {
              'background-color': '#3cb371',
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
      })
    },
    //グラフ更新の関数
    add_node(keyword, exkeyword) {
      cy.add([
        {
          group: 'nodes',
          data: {id : keyword}, position: {x : 100 + this.arg_x, y : 100 + this.arg_y}
        },
        {
          group: 'edges',
          data: {id : this.edge_id, source: exkeyword, target: keyword}
        }
      ]);
      this.arg_x += 20;
      this.arg_y += 20;
      this.edge_id += 1;
    },
    //グラフ生成の関数
    make_graph() {
      let exkeyword = "";
      let keyword = "";
      for (let i = 0; i < this.keywords.length; i++) {
        if (i==0){
          keyword = this.keywords[i];
        }
        else {
          exkeyword = keyword;
          keyword = this.keywords[i];
          this.add_node(keyword, exkeyword);
        }
      }
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
    top: 50px;
    left: 0px;
    text-align: left;
}
</style>