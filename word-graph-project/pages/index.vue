<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="9" lg="9" xl="9">
        <!-- <my-speech-recognition ref="spRecog"></my-speech-recognition>
        <v-btn @click="start_recog()" color="primary">音声認識の開始</v-btn>
        <v-btn @click="stop_recog()" color="red">音声認識の終了</v-btn> -->
        <div class="float-left">
          <v-btn @click="save_log()" color="green">
            <v-icon>mdi-download</v-icon>
            JSONで保存
          </v-btn>
          <v-btn @click="store_result()" color="green">
            <v-icon>mdi-download</v-icon>
            画像で保存
          </v-btn>
          <v-btn @click="start_from_excel()" color="green">
            <v-icon>mdi-microsoft-excel</v-icon>
            excelから解析
          </v-btn>
          <v-btn @click="start_from_json()" color="green">
            <v-icon>mdi-upload</v-icon>
            JSONから読込
          </v-btn>
          <v-btn @click="clear_graph()" color="green">
            <v-icon>mdi-cached</v-icon>
            グラフを初期化
          </v-btn>
          <!-- <input type="file" @change="onFileChange" style="color:#FDD"/> -->
          <v-file-input
            show-size
            type="file"
            @change="onFileChange2"
            label="excelまたはjsonファイルを選択してください"
            accept=".xlsx, .json"
          ></v-file-input>
        </div>
        <!-- <v-card>
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
        </v-card> -->
      </v-col>
      <v-col cols="12" sm="12" md="3" lg="3" xl="3">
        <!-- <v-card style="height: 100%; weight:100%">
          <v-card-text></v-card-text>
        </v-card> -->
      </v-col>
      <v-col cols="12" sm="12" md="12" lg="12" xl="12">
        <div id="cy"></div> 
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import * as xlsx from 'xlsx';
import kuromoji from 'kuromoji'
import cytoscape from 'cytoscape';
import {saveAs} from 'file-saver';
import ver01_temp from './ver01_temp.vue';

export default {
  components: { ver01_temp },
  name: 'all_methods',
  data() {
    return {
      excel_data: null,
      talk_title: "",
      node_info: "text",
      speakers : ['default'],
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
      col_define: [
        '#f8b500',  //山吹色 default
        '#3cb371',  //緑     color1 
        '#0000ff',  //青     color2
        '#8a2be2',  //紫     color3
        '#f08080',  //ピンク color4
      ],
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
      let blob = this.cy.png(
        {
          output:'blob',
          bg:'white',
          full:true,
        }
      );
      let link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = 'graph.png';
      link.click();
      URL.revokeObjectURL(link.href);
    },
    save_log() {
      const fileName = "talk.json";  // 保存するJSONファイルの名前 今後this.talk等で変更可能にするかもしれない．
      const data = JSON.stringify(this.keywords); // データをJSON形式の文字列に変換する。
      const blob = new Blob([data], {
        type : "application/json"
      })
      saveAs(blob, fileName);
    },
    start_from_excel() {
      let vm = this;
      let len = vm.excel_data.length;
      for (let i = 1; i < len; i++) { //0番目は表の項目名のためデータは1番目から
        let sentence = vm.excel_data[i][0];
        let speaker = vm.excel_data[i][1];
        vm.tokenize([sentence, speaker])
        .then(vm.categolize)
        .then(vm.update_next)
        .then(vm.update_keywords)
        .then((res)=>{
          vm.update_graph();
          // console.log(i + "回目");
          // console.log(vm.keywords);
        });
      }
    },
    start_from_json() {
      let vm = this;
      //先にキーワードをノードとして追加する
      for (let key of vm.keywords) {
        vm.update_node(key.keyword, key.weight, key.speaker);
      }
      //ノードをweight(出た回数)に応じてリサイズする
      vm.resize_nodes();
      //keywordからedgeを生成する 
      for (let key of vm.keywords) {
        let after_next = key.after_next.filter((ele, pos)=>{
          return key.after_next.indexOf(ele)===pos;
        })
        for (let i = 0; i < after_next.length; i++) {
          vm.update_edges(key.keyword, key.after_next[i]);
        }
      }
    },
    clear_graph() {
      //keywordsの初期化
      this.keywords = [
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
      ];
      this.k = 100;
      this.theta = 0;
      for (let key of this.keywords) {
        this.cy.remove( key.keyword );
      }
      this.update_node("start", 0, "default");
    },
    // 通常のinputバージョン，input(typeがfile)では，eventが引数として入ってくる．
    // onFileChange(event) {
    //   console.log(event);
    //   this.file = event.target.files ? event.target.files[0] : null;
    //   if (this.file) {
    //     const reader = new FileReader();
    //     reader.onload = (e) => {
    //       const bstr = e.target.result;
    //       const wb = xlsx.read(bstr, {type: 'binary'});
    //       const wsname = wb.SheetNames[0];
    //       const ws = wb.Sheets[wsname];
    //       this.excel_data = xlsx.utils.sheet_to_json(ws, {header: 1}); 
    //     }
    //     reader.readAsBinaryString(this.file);
    //   }
    // },
    //vuetifyのv-file-inputでは，eventではなくfilesが入ってくるため書き換えた．
    onFileChange2(files) {
      let vm = this;
      if (files) {
        //excelファイルを読み込んだとき
        if (files.name.indexOf('.xlsx') > -1) {
          const reader = new FileReader();
          reader.onload = (e) => {
            const bstr = e.target.result;
            const wb = xlsx.read(bstr, {type: 'binary'});
            const wsname = wb.SheetNames[0];
            const ws = wb.Sheets[wsname];
            this.excel_data = xlsx.utils.sheet_to_json(ws, {header: 1}); 
          }
          reader.readAsBinaryString(files);
        }
        //jsonファイルを読み込んだとき
        else if (files.name.indexOf('.json') > -1) {
          vm.get_file_data(files)
          .then((res)=>{
            vm.keywords = res;
          })
        }
      }
    },
    //jsonファイルからデータを読み取るメソッド
    get_file_data(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(JSON.parse(e.target.result));
        reader.onerror = () => reject(error)
        reader.readAsText(file);
      })
    },
    hiragana_fillter(keyword) {
      keyword = (keyword==null)?"":keyword;
      if(keyword.match(/^[ぁ-んー　]*$/) && keyword.length === 1){    //"ー"の後ろの文字は全角スペースです。
        return true;
      }
      else{
        return false;
      }
    },
    tokenize(arrayData) {
      return new Promise((resolve)=>{
        kuromoji.builder({ dicPath: '/dict' }).build((err, tokenizer) => {
          if (err) {
            console.log(err);
          }
          else {
            //arrayData[0]はsentenceのこと
            const tokens = tokenizer.tokenize(arrayData[0]);
            resolve([tokens,arrayData[1]]);
          }
        })
      })
    },
    categolize(arrayData) {
      return new Promise((resolve)=> {
        let analysised_data = [];
        let tokens = arrayData[0];
        for (let token of tokens) {
          let flag = this.hiragana_fillter[token.basic_form];
          if (!flag) {
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
          
        }
        resolve([analysised_data, arrayData[1]]);
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
    update_next(arrayData) {
      return new Promise((resolve)=>{
        let vm = this;
        let analysised_data = arrayData[0];
        let speaker = arrayData[1];
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
        resolve([analysised_data, speaker]);
      })
    },
    //既出単語の更新，および新出単語の更新
    update_keywords(arrayData){
      return new Promise((resolve)=>{
        let vm = this;
        let analysised_data = arrayData[0];
        let speaker = arrayData[1];
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
                "speaker" : speaker || "default",
              })
            }
            // else if (data.position == "verb") {
            //   vm.keywords.push({
            //     "keyword":data.keyword, 
            //     "weight":0, 
            //     "before_next":adverbs,
            //     "after_next":[],
            //     "isLatest":true,
            //     "isInGraph":false,
            //     "position" : "verb",
            //     "speaker" : speaker || "default",
            //   })
            // }
            // else if (data.position == "adverb"){
            //   vm.keywords.push({
            //     "keyword":data.keyword, 
            //     "weight":0, 
            //     "before_next":[],
            //     "after_next":[],
            //     "isLatest":true,
            //     "isInGraph":false,
            //     "position" : "adverb",
            //     "speaker" : speaker || "default",
            //   })
            // }
            // else if (data.position == "adjective"){
            //   vm.keywords.push({
            //     "keyword":data.keyword, 
            //     "weight":0, 
            //     "before_next":[],
            //     "after_next":[],
            //     "isLatest":true,
            //     "isInGraph":false,
            //     "position" : "adjective",
            //     "speaker" : speaker || "default",
            //   })
            // }
          }
          resolve(analysised_data);
        }
      })
    },
    clabel_setter(speaker){
      let is_exist = false;
      for (let sp of this.speakers){
        if (sp===speaker){
          is_exist = true;
        }
      }
      if (!is_exist){
        this.speakers.push(speaker);
      }
      return this.col_define[this.speakers.indexOf(speaker)]
    },
    init_graph() {
      this.cy = cytoscape({
        container : document.getElementById('cy'),
        elements : [
          {
            data: {
              id : 'start',
              weight : 0,
            }
          },
        ],
        style : [
          {
            selector : 'node',
            style : {
              'background-color' : this.col_define[0],
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
    update_node(keyword, weight, speaker) {
      return new Promise((resolve)=>{
        this.cy.add([
          {
            group : 'nodes',
            data : {
              id : keyword, 
              weight: weight,
            },
            position : {
              x : 150 + Math.cos(this.theta)*this.k,
              y : 150 + Math.sin(this.theta)*this.k,
            },
            style : {
              'background-color' : this.clabel_setter(speaker),
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
            vm.update_node(key.keyword, key.weight, key.speaker);
            key.isInGraph = true;
          }
        }
        //ノードをweight(出た回数)に応じてリサイズする
        vm.resize_nodes();

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
    },
    resize_node(weight) {
      let collection = this.cy.elements('node[weight = ' + weight + ']');
      let r = 30 + weight*10;
      collection.css('width',r);
      collection.css('height',r);
    },
    resize_nodes() {
      for (let weight = 0; weight < 50; weight++) {
        this.resize_node(weight);
      }
      this.resize_node(51);
    },
    graph_event_tap() {
      this.cy.on('tap', 'node', function(evt){
        let node = evt.target;
        let next_nodes = "";
        let prev_nodes = "";
        for (let i = 0; i < node._private.edges.length; i++){
          let edge_source = node._private.edges[i]._private.data.source;
          let edge_target = node._private.edges[i]._private.data.target;
          if (edge_source != node.id()) {
            prev_nodes += edge_source + ", ";
          }
          if (edge_target != node.id()) {
            next_nodes += edge_target + ", ";
          }
        }
        let resouce_value = next_nodes.length != 0 ? prev_nodes.length/next_nodes.length : 0;
        let node_info = "keyword = " + node.id() 
                      + " \nweight = " + node._private.data.weight
                      + "\nprev = " + prev_nodes
                      + "\nnext = " + next_nodes
                      + "\n資源率 = " + resouce_value;
        alert(node_info);
        // console.log(node_info);
      });
    }
  },
  computed: {
  },
  mounted() {
    this.init_graph();
    this.graph_event_tap();
  }
}
</script>

<style scoped>
#cy {
  background-color: #f3f3f2;
  height: calc(100vw / 3);
}
</style>