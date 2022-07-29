<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="5" lg="5" xl="5">
        <!-- <my-speech-recognition ref="spRecog"></my-speech-recognition>
        <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
        <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn> -->
        <v-btn @click="start_from_excel()" color="green">エクセルを解析</v-btn>
        <v-btn @click="store_result()" color="green">解析結果を保存</v-btn>
        <input type="file" @change="onFileChange" />
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
      </v-col>
      <v-col cols="12" sm="12" md="7" lg="7" xl="7">
        <div id="cy"></div> 
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import * as xlsx from 'xlsx';
import kuromoji from 'kuromoji'
import cytoscape from 'cytoscape';

export default {
  name: 'all_methods',
  data() {
    return {
      excel_data: null,
      talk_title: "test",
      keywords : [
        {
          "keyword" : "start",
          "weight" : 0,
          "before_next" : [],
          "after_next" : [],
          "isLatest" : true,
          "isInGraph" : true,
          "position" : "noun",
          "speaker" : "default",
        }
      ],
      col_define: {
        default_color: '#f8b500',  //山吹色
        color_patarn_1: '',
        color_patarn_2: '',
        color_patarn_3: '',
        color_patarn_4: '',
        color_patarn_5: '',
      },
      k : 100,
      theta : 0,
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
    }
  },
  methods: {
    store_result() {
      //canvasをimageとして保存する方法を探す
    },
    start_from_excel() {
      let vm = this;
      let len = vm.excel_data.length;
      for (let i = 1; i < len; i++) { //0番目は表の項目名のためデータは1番目から
        let sentence = vm.excel_data[i][0];
        let speaker = vm.excel_data[i][1];
        vm.tokenize(sentence)
        .then(vm.categolize)
        .then(vm.update_next)
        .then(vm.update_keywords)
        .then((res)=>{
          vm.update_graph();
          console.log(i + "回目");
          console.log(vm.keywords);
        });
        // console.log('From MydataCenter');
        // await this.$refs.kuromoji.update_next(analysised_data); 
        // console.log('end update next');
        // await this.$refs.kuromoji.process_keyword(analysised_data, speaker);
        // console.log('end promise_keyword');
      }
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
    //keywordsで重複したkeywordかどうか判定する．
    //return : 重複していれば配列の添え字を返す.重複していなければ-1.
    check_duplicate(keyword) {
      let idx = -1;
      this.keywords.forEach(function(value, index){
        if (keyword == value.keyword) {
          idx = index;
        }
      });
      return idx;
    },
    //既出単語のbefore_next(描画前の次単語)を更新
    update_next(analysised_data) {
      return new Promise((resolve)=>{
        let vm = this;
        for (let data of analysised_data) {
          for (let keyword of vm.keywords) {
            if (data.position=='noun' && keyword.position=='noun' && keyword.isLatest==true){
              keyword.before_next.push(data.keyword);
            }
          }
        }
        //一括でfalseに更新
        for (let keyword of vm.keywords) {
          keyword.isLatest = false;
        }
        resolve(analysised_data);
      })
    },
    //既出単語の更新，および新出単語の更新
    update_keywords(analysised_data){
      return new Promise((resolve)=>{
        let vm = this;
        let nouns = [];
        let verbs = [];
        let adverbs = [];
        let adjectives = [];
        //analysised_dataを品詞ごとに分類する
        for (let data of analysised_data) {
          if (data.position == "noun") {
            nouns.push(data.keyword);
          }
          else if (data.position == "verb") {
            verbs.push(data.keyword);
          }
          else if (data.position == "adverb") {
            adverbs.push(data.keyword);
          }
          else if (data.positon == "adjective") {
            adjectives.push(data.keyword);
          }
        }
        //analysised_dataをもとに既出単語の更新，および新出単語の更新を行う．
        for (let data of analysised_data) {
          let index = vm.check_duplicate(data.keyword);
          if (index != -1){
            vm.keywords[index].weight += 1;
            vm.keywords[index].isLatest = true;
          }
          else {
            if (data.position == "noun"){
              vm.keywords.push({
                "keyword":data.keyword, 
                "weight":0, 
                "before_next":verbs.concat(adjectives),
                "after_next":[],
                "isLatest":true,
                "isInGraph":false,
                "position" : "noun",
                "speaker" : "default",
              })
            }
            else if (data.position == "verb") {
              vm.keywords.push({
                "keyword":data.keyword, 
                "weight":0, 
                "before_next":adverbs,
                "after_next":[],
                "isLatest":true,
                "isInGraph":false,
                "position" : "verb",
                "speaker" : "default",
              })
            }
            else if (data.position == "adverb"){
              vm.keywords.push({
                "keyword":data.keyword, 
                "weight":0, 
                "before_next":[],
                "after_next":[],
                "isLatest":true,
                "isInGraph":false,
                "position" : "adverb",
                "speaker" : "default",
              })
            }
            else if (data.position == "adjective"){
              vm.keywords.push({
                "keyword":data.keyword, 
                "weight":0, 
                "before_next":[],
                "after_next":[],
                "isLatest":true,
                "isInGraph":false,
                "position" : "adjective",
                "speaker" : "default",
              })
            }
          }
          resolve(analysised_data);
        }
      })
    },
    init_graph() {
      this.cy = cytoscape({
        container : document.getElementById('cy'),
        elements : [
          {
            data: {id : 'start'}
          },
        ],
        style : [
          {
            selector : 'node',
            style : {
              'background-color' : this.col_define.default_color,
              'color' : 'black',
              'label' : 'data(id)',
            }
          },
          {
            selector: 'edge',
            style : {
              'width':3,
              'line-color':'#ccc',
              'target-arrow-color': '#ccc',
              'target-arrow-shape': 'triangle',
              'curve-style': 'bezier',
            }
          }
        ]
      });
    },
    update_node(keyword) {
      return new Promise((resolve)=>{
        this.cy.add([
          {
            group : 'nodes',
            data : {id : keyword},
            position : {
              x : 150 + Math.cos(this.theta)*this.k,
              y : 150 + Math.sin(this.theta)*this.k,
            }
          }
        ]);
        this.k = 200 + Math.random()*1500;
        this.theta += Math.PI/10;
      })
    },
    update_edges(source_node, target_node) {
      return new Promise((resolve)=> {
        try {
          this.cy.add([
            {
              group : 'edges',
              data : {
                source : source_node,
                target : target_node,
              }
            }
          ]);
        }
        catch(e) {
          //console.log(e);
        }
      })
      
    },
    update_graph() {
      return new Promise((resolve)=>{
        let vm = this;
        //先にキーワードをノードとして追加する
        for (let key of vm.keywords) {
          if (key.isInGraph == false) {
            vm.update_node(key.keyword);
            key.isInGraph = true;
          }
        }

        //keywordからedgeを生成する
        for (let key of vm.keywords) {
          //edgeのsourceとなるkeywordを取り出す．
          let keyword = key.keyword;

          //重複する値の削除して，nextを用意する．
          //indexOf(ele)では，配列においてeleのあるindexのうち最小のものを返す．
          let before_next = key.before_next.filter((ele, pos)=>{
            return key.before_next.indexOf(ele)===pos;
          })
          let after_next = key.after_next.filter((ele, pos)=>{
            return key.after_next.indexOf(ele)===pos;
          })

          for (let i = 0; i < before_next.length; i++) {
            let is_drawn = false;
            //描画済みのedgeではないか確認する
            for (let j = 0; j < after_next.length; j++) {
              if (before_next[i]==after_next[j]) {
                is_drawn = true;
              }
            }
            if (is_drawn == false) {
              vm.update_edges(keyword, before_next[i]);
            }
          }
          //before_nextをafter_nextに更新する.
          key.after_next = after_next.concat(before_next);
          key.before_next = [];
        }
      })
      
    }
  },
  mounted() {
    this.init_graph();
  }
}
</script>

<style scoped>
#cy {
  background-color: #f3f3f2;
  height: calc(100vw / 3);
}
</style>