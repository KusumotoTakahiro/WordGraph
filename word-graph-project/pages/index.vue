<template>
  <div>
    <v-alert class="text-h2 title">Word Graph</v-alert>
    <v-alert class="text-subtitle subtitle">
      selected : {{selected_node}} <br/>
      next_nodes : 
        <v-icon v-if="selected_flag" color="green darken-2"> 
          mdi-star 
        </v-icon> <br/>
      prev_nodes : 
        <v-icon v-if="!selected_flag" color="green darken-2"> 
          mdi-star 
        </v-icon> <br/>
    </v-alert>
    <div id="cy"></div>
    <v-simple-table class="speaker_list" light dense>
      <template v-slot:default>
        <thead>
          <tr>
            <th class="text-center">
              color label
            </th>
            <th class="text-center">
              name
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="item in labels"
            :key="item.speaker"
          >
            <td>
              <v-row justify="center">
                <div class="clabel" :style="{ 'background-color':item.clabel }"></div>
              </v-row>
            </td>
            <td class="text-center">{{ item.speaker }}</td>
          </tr>
        </tbody>
      </template>
    </v-simple-table>
    <!-- 独自context-menu -->
    <v-card 
      class="contextmenu py-0 " 
      v-if="contextmenu" 
      :style="contextmenu_style"
      light
      elevation="10"
    >
      <v-list class="py-0">
        <v-list-item-group>
          <v-list-item @click="make_analysis_contents()">
            <v-list-item-icon>
              <v-icon>
                mdi-chart-box
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                解析結果を表示
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider style="background:gainsboro"></v-divider>
          <v-list-item @click="store_result()">
            <v-list-item-icon>
              <v-icon>
                mdi-download
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                画像で保存
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider style="background:gainsboro"></v-divider>
          <v-list-item @click="$refs.input.click()">
            <v-list-item-icon>
              <v-icon>
                mdi-alpha-j-box
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <!-- <v-file-input
                show-size
                type="file"
                @change="onFileChange2"
                label="jsonファイルを選択してください"
                accept=".json, .csv"
                style="display:none"
              ></v-file-input> -->
              <input
                style="display: none"
                ref="input"
                type="file"
                accept=".json"
                @change="onFileChange3"
              />
              <v-list-item-title>
                JSONから読込
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider style="background:gainsboro"></v-divider>
          <v-list-item @click="clear_graph()">
            <v-list-item-icon>
              <v-icon>
                mdi-cached
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                グラフを初期化
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider style="background:gainsboro"></v-divider>
          <v-list-item @click="removeParentsOfOneChild">
            <v-list-item-icon>
              <v-icon>
                mdi-select-off
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                グループ解除
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider style="background:gainsboro"></v-divider>
          <v-list-item @click="remove_class">
            <v-list-item-icon>
              <v-icon>
                mdi-cancel
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>
                セレクト解除
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-card>
    <!-- 解析結果を表示するダイアログ -->
    <v-dialog 
      v-model="analysis_dialog"
      fullscreen
      transition="dialog-top-transition"
      hide-overlay
      light
    >
      <v-card v-if="loading">
        <v-toolbar dark color="primary">
          <v-toolbar-title>解析結果</v-toolbar-title>
        </v-toolbar>
        <!-- 気まぐれローディング画面 -->
        <div class="loader text-center" text-align="center">
          <div style="font-size:small; color:blue;" class="mt-2">Loading...</div>
        </div>
        <div></div>
      </v-card>
      <v-card v-else="!loading">
        <v-toolbar dark color="primary">
          <v-toolbar-title>解析結果</v-toolbar-title>
        </v-toolbar>
        dialogです
        ここに結果を表示していく
        {{analysis_content}}
        <v-row justify="center">
          <v-btn
            icon
            light
            @click="analysis_dialog=false"
            right
          ><v-icon>mdi-close</v-icon></v-btn>
        </v-row>
      </v-card>
    </v-dialog>
    <v-dialog
      v-model="dialog"
      width="400"
      light
    >
    <v-card style="background:white">
    <v-alert
      border="left"
      color=""
      dense
      width="300"
      icon="mdi-information-outline"
      elevation="2"
      light
      class="mb-0"
    >
      ノード情報
    </v-alert>
      <v-simple-table light>
        <tbody style="color:black">
          <tr>
            <th >keyword</th>
            <td >{{node_info.keyword}}</td>
          </tr>
          <tr>
            <th >最初の発言者</th>
            <td >{{node_info.speaker}}</td>
          </tr>
          <tr>
            <th >tf-idf値</th>
            <td >{{node_info.weight}}</td>
          </tr>
          <tr>
            <th >元ノード</th>
            <td >{{node_info.prev}}</td>
          </tr>
          <tr>
            <th >次ノード</th>
            <td >{{node_info.next}}</td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>
    </v-dialog>
    <v-card v-if="chart" class="dialog" light>
      <v-alert
          border="left"
          color=""
          dense
          width="400"
          icon="mdi-information-outline"
          elevation="2"
        >
        エッジ情報
      </v-alert>
      <v-card-text>
        「{{dataSource}}」から「{{dataTarget}}」への話題転換者
      </v-card-text>
      <chart :chartData="chartData" :options="chartoptions"></chart>
      <v-spacer></v-spacer>
      <v-card-actions>
        <v-row 
          align-content="center"
          justify="space-around"
          class="mt-5"
        >
          <v-btn @click="chart=!chart" text>close</v-btn>
        </v-row>
      </v-card-actions>
    </v-card>
    <!-- <div class="contextmenu" v-if="contextmenu" :style="contextmenu_style">
      <v-row>
        <v-btn @click="save_log()" color="green" disabled>
          <v-icon>mdi-download</v-icon>
          JSONで保存
        </v-btn>
        <v-btn @click="store_result()" color="green" width="200px">
          <v-icon>mdi-download</v-icon>
          画像で保存
        </v-btn>
        <v-btn @click="start_from_json()" color="green" width="200px">
          <v-icon>mdi-upload</v-icon>
          JSONから読込
        </v-btn>
      </v-row>
      <v-row>
        <v-btn @click="clear_graph()" color="green" width="200px">
          <v-icon>mdi-cached</v-icon>
          グラフを初期化
        </v-btn>
        <v-btn @click="removeParentsOfOneChild" color="green" width="200px">
          compoundクリア
        </v-btn>
        <v-btn @click="start_from_excel()" color="green" disabled>
          <v-icon>mdi-microsoft-excel</v-icon>
          excelから解析
        </v-btn>
      </v-row>
      <input type="file" @change="onFileChange" style="color:#FDD"/>
      <v-file-input
        show-size
        type="file"
        @change="onFileChange2"
        label="excelまたはjsonファイルを選択してください"
        accept=".xlsx, .json, .csv"
      ></v-file-input>
    </div> -->
  </div>
</template>

<script>
import * as xlsx from 'xlsx';
import kuromoji from 'kuromoji'
import cytoscape from 'cytoscape';
import {saveAs} from 'file-saver';
import ver01_temp from './ver01_temp.vue';
import sample1 from '../assets/sample1.json';
import chart from '~/components/chart.vue';

export default {
  components: { ver01_temp, chart },
  name: 'all_methods',
  data() {
    return {
      selected_node: null,
      analysis_dialog: false,
      analysis_content: null, //オブジェクト形式で格納する
      loading: true,
      next_nodes: [],
      prev_nodes: [],
      selected_flag: null,
      contextmenu: false,
      contextmenu_style:{'left':'20px', 'top':'20px'},
      node_info: null,
      dialog:false,
      excel_data: null,
      csv_data: [],
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
        '#f08080',  //ピンク color4
        '#0D9FF2',  //水色 color5
        '#8a2be2',  //紫     color3
        '#3cb371',  //緑     color1 
        '#0000ff',  //青     color2
        '#0DF2A9',  //黄緑 color6
        '#F20D37',  //赤 color7
        '#5D0212',  //赤褐色 color8
      ],
      labels: [],
      k : 100,
      theta : 0,
      removeEmptyParents : false, //demoサイトから引用
      chart: false,
      chartData: null,
      chartoptions: {
        responsive: true,
        maintainAspectRatio: false,
      },
      dataSource: '',
      dataTarget: '',
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
      console.log('start_from_excel');
      let vm = this;
      let cnt = 0;
      for (let data of vm.csv_data) { //0番目は表の項目名のためデータは1番目から
        let sentence = data.word;
        let speaker = data.speaker;
        vm.tokenize([sentence, speaker])
        .then(vm.categolize)
        .then(vm.update_next)
        .then(vm.update_keywords)
        .then((res)=>{
          vm.update_graph();
        });
        if (cnt===150) break;
        else cnt+=1;
      }
    },
    //もとは引数なしでthis.keywordsだったが，直接引数でkeywordsをとることにした．
    start_from_json(keywords) {
      let vm = this;
      new Promise((resolve)=>{
        //先にキーワードをノードとして追加する
        console.log(1);
        for (let key of keywords) {
          vm.update_node(key.keyword, key.weight, key.speaker, key.speakers);
        }
        //keywordからedgeを生成する 
        for (let key of keywords) {
          //おそらく，重複削除のために行っていたが，今回は重複削除済みのためいらない．
          // let next_nodes = key.after_next.filter((ele, pos)=>{
            //indexofでその配列内で特定の要素が初めて出現したときのindexを返す．
          //   return key.after_next.indexOf(ele)===pos; 
          // })
          for (let i = 0; i < key.after_next.length; i++) {
            //引数はsource_node, target_node, s=>tの話題転換者
            vm.update_edges(key.keyword, key.after_next[i], key.next_speakers[key.after_next[i]]);
          }
        }
        vm.create_speaker_label();
        vm.set_graph_style();
        this.analysis_content = null; //解析結果の初期化
        this.loading = true;
        resolve('ok');
      })
    },
    //現在のスピーカーを一覧表示するテーブルのデータを作成
    create_speaker_label() {
      console.log(this.speakers)
      this.speakers.forEach(speaker => {
        let data = {}
        data.speaker = speaker;
        data.clabel = this.clabel_setter(speaker);
        this.labels.push(data);
      })
    },
    //グラフのスタイルの設定
    set_graph_style() {
      return new Promise((resolve)=> {
        console.log(2);
        this.cy.style()
        .selector('node:parent')
        .style('label', '')
        .style('background-color', '#dcdcdc')
        .style('border-color', 'red')
        .style('border-style', 'dashed')
        .selector('.cdnd-grabbed-node')
        .style('background-color', 'red')
        .selector('.cdnd-drop-sibling')
        .style('background-color', 'red')
        .selector('.cdnd-drop-target')
        .style('border-color', 'red')
        .style('border-style', 'dashed')
        .selector('.unselected')
        .style('opacity', '0.1')
        .update()

        let options = {
          name: 'concentric',

          fit: true, // whether to fit to viewport
          padding: 30, // fit padding
          boundingBox: undefined, // constrain layout bounds; { x1, y1, x2, y2 } or { x1, y1, w, h }
          animate: false, // whether to transition the node positions
          animationDuration: 5000, // duration of animation in ms if enabled
          animationEasing: undefined, // easing of animation if enabled
          animateFilter: function ( node, i ){ return true; }, // a function that determines whether the node should be animated.  All nodes animated by default on animate enabled.  Non-animated nodes are positioned immediately when the layout starts
          ready: undefined, // callback on layoutready
          stop: undefined, // callback on layoutstop
          transform: function (node, position ){ return position; } // transform a given node position. Useful for changing flow direction in discrete layouts 
        };

        this.cy.layout( options ).run();

      });
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
      this.update_node("start", 0, "default", ["default"]);
      //clabel_setterの初期化
      this.speakers = ['default'];
      this.labels = [];
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
      console.log(files);
      let vm = this;
      if (files) {
        //csvファイルを読み込んだとき
        if (files.name.indexOf('.csv') > -1) {
          vm.get_csv_data(files)
          .then(vm.process_csv_data)
          .then(vm.start_from_json())
        }
        //jsonファイルを読み込んだとき
        else if (files.name.indexOf('.json') > -1) {
          vm.clear_graph();
          vm.get_json_data(files)
        }
      }
    },
    //input用に作り直し
    onFileChange3(event) {
      this.file = event.target.files ? event.target.files[0] : null;
      if (this.file) {
        this.clear_graph();
        this.get_json_data(this.file)
        .then(res=>{
          this.start_from_json(res)
          this.keywords = res;
        })
        .then(this.set_graph_style())
      }
    },
    //jsonファイルからデータを読み取るメソッド
    get_json_data(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(JSON.parse(e.target.result));
        reader.onerror = () => reject(error)
        reader.readAsText(file);
      })
    },
    //csvファイルからデータを読み取るメソッド
    get_csv_data(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result.split('\r\n'));
        reader.onerror = () => reject(error)
        reader.readAsText(file);
      })
    },
    //get_csv_dataから来た結果をここで扱えるdataに変換するメソッド
    process_csv_data(res) {
      let vm = this;
      let result = res;
      let header = result[0].split(',')
      result.shift();
      vm.csv_data = result.map(item=>{
        let datas = item.split(',');
        let temp = {};
        for (const index in datas) {
          let key = header[index];
          temp[key] = datas[index];
        }
        return temp;
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
    get_props(speakers) {
      let result = {};
      for (let i = 0; i < speakers.length; i++) {
        if (Object.keys(result).indexOf(speakers[i].toString()) === -1) {
          //console.log('first ', speakers[i].toString());
          result[speakers[i].toString()] = 1;
        }
        else {
          //console.log('second ', speakers[i].toString());
          result[speakers[i].toString()] += 1;
        }
      }
      return result;
    },
    makeIMG(len, speaker, speakers) {
      const img = document.createElement('canvas');
      img.setAttribute('width', len);
      img.setAttribute('height', len);
      const context = img.getContext("2d");
      let num = 0;
      for (const value of Object.values(speakers)) {
        num += Number(value);
      }

      //console.log(speakers)

      let angle_now = 0, s_angle = 0, e_angle = 0, x = 0, y = 0, r = 0;
      for (const [key, value] of Object.entries(speakers)) {
        //console.log(key, value);
        context.beginPath();
        angle_now = (value / num) * 360;
        s_angle = e_angle;
        e_angle += angle_now * Math.PI /180;
        x = len/2; //円弧の中心のx座標 一辺の半分を指定
        y = len/2; //円弧の中心のy座標 一辺の半分を指定
        r = len/2; //円弧の半径
        context.arc(x, y, r, s_angle, e_angle, false);
        context.lineTo(len/2, len/2);
        context.fillStyle = this.clabel_setter(key.toString());
        context.fill();
        context.strokeStyle = 'white';
        context.lineWidth = 4;
        context.stroke();
      }

      context.beginPath();
      context.arc(len/2, len/2, len/3, 0, 2*Math.PI, false);
      
      context.fillStyle = this.clabel_setter(speaker.toString())
      context.fill();
      context.strokeStyle = 'white';
      context.lineWidth = 5;
      context.stroke();

      return {'img':img.toDataURL(), 'width':len, 'height':len}
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
              'font-size': '100px',
              'font-family': 'serif',
            }
          },
          {
            selector: 'edge',
            style : {
              //'width':1,
              'line-color':'#ccc',
              'target-arrow-color': '#ccc',
              'target-arrow-shape': 'triangle',
              'arrow-scale': '4',
              'arrow-shape': 'vee',

              'curve-style': 'bezier',
            }
          }, 
          {
            selector: 'node:parent',
            style: {
              'label': ''
            }
          },
          {
            selector: '.cdnd-grabbed-node',
            style: {
              'background-color': 'red'
            }
          },

          {
            selector: '.cdnd-drop-sibling',
            style: {
              'background-color': 'red'
            }
          },

          {
            selector: '.cdnd-drop-target',
            style: {
              'border-color': 'red',
              'border-style': 'dashed'
            }
          }
        ],
        wheelSensitivity: 0.1, //zoom sensitivity 0 ~ 1 is slower, 1 ~ is faster
      });
    },
    update_node(keyword, weight, speaker, speakers) {
      return new Promise((resolve)=>{
        //30+weight*10が半径
        let result = this.makeIMG(30 + weight*10, speaker, this.get_props(speakers));
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
              //'background-color' : this.clabel_setter(speaker),
              'background-image': result.img, //ここで円グラフを表示する
              'width': result.width,
              'height' : result.height,
            }
          }
        ]);
        this.k = 200 + Math.random()*1500;
        this.theta += Math.PI/10;
      })
    },
    update_edges(source_node, target_node, next_speakers) {
      return new Promise((resolve)=> {
        //next_speakersの長さ兼，EdgeのWidth
        let width = next_speakers.length;

        //自己ループは今回無視
        if (source_node===target_node) {
          return;
        }

        //next_speakersの割合を取得
        let label = [];
        let ratio = {};
        next_speakers.forEach(speaker=>{
          if (label.includes(String(speaker))) {
            ratio[String(speaker)] += 1;
          }
          else {
            label.push(String(speaker));
            ratio[String(speaker)] = 1;
          }
        })
        let positions = "0% ";
        let now = 0;
        Object.keys(ratio).forEach(speaker => {
          now += ratio[speaker] / width
          Math.round(now).toString() + "% ";
        })

        //色を作成
        let colors = "";
        let color = "";
        label.forEach(speaker => {
          color = this.clabel_setter(speaker);
          colors += color +  " "
        }) 
        
        //styleの作成
        let edge_style = {}
        if (label.length===1) {
          //発言者は一人だから
          edge_style['line-color'] = color;
          edge_style['target-arrow-color'] = color;
        }
        else {
          //発言者は二人以上
          edge_style['line-fill'] = 'linear-gradient';
          edge_style['line-gradient-stop-colors'] = colors;
          edge_style['line-gradient-stop-positions'] = positions;
        }
        edge_style['width'] = 3 * Math.pow(2, width); //最小値は3
        edge_style['arrow-scale'] = width;

        //edgeの作成
        try {
          this.cy.add([
            {
              group : 'edges',
              data : {
                source : source_node,
                target : target_node,
              },
              style: edge_style,
            }
          ]);
        }
        catch(e) {
          console.log(e);
          //console.log('source: ', source_node, ', target: ', target_node)
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
      for (let weight = 0; weight < 100; weight++) {
        this.resize_node(weight);
      }
    },
    remove_class() {
      let vm = this;
      vm.cy.edges().forEach(edge => {
        if (edge.hasClass('unselected')) {
          edge.removeClass('unselected');
        }
        if (edge.hasClass('selected')) {
          edge.removeClass('selected');
        }
      })
      vm.cy.nodes().forEach(node => {
        if (node.hasClass('unselected')) {
          node.removeClass('unselected');
        }
      })
    },
    //tapしたnodeとそこから繋がるEdgeを強調する
    highlight_nodes_and_edges() {
      let vm = this;
      this.cy.on('tap', 'node', function(evt){
        //同じnodeを連続してtapした場合 nextとprevを入れ替える
        if (vm.selected_node === evt.target.data().id) {
          console.log('change');
          vm.selected_flag = !vm.selected_flag;
        }
        //別のnodeをtapした場合 
        else {
          console.log('new')
          vm.selected_flag = true;
        }
        //console.log(vm.selected_flag);
        //現在のclassを消去する
        vm.remove_class();
        let source_node = evt.target;
        let target_nodes = [];
        let source_nodes = [];
        let edges = source_node._private.edges;
        let edges1 = [];
        let edges2 = [];
        //ここでedgesのselectedをClassに追加する
        for (let i = 0; i < edges.length; i++) {
          //edges[i].addClass('selected');
          if (edges[i]._private.data.target !== source_node.data().id) {
            target_nodes.push(edges[i]._private.data.target);
            edges1.push(edges[i].data().id);
          }
          else {
            source_nodes.push(edges[i]._private.data.source);
            edges2.push(edges[i].data().id);
          }
        }
        //edgesのうちselected以外のものをunselectedにする
        vm.cy.edges().forEach(edge=>{
          if (vm.selected_flag) {
            if (!edges1.includes(edge.data().id)) {
              edge.addClass('unselected');
            }
          }
          else {
            if (!edges2.includes(edge.data().id)) {
              edge.addClass('unselected');
            }
          }
        })
        //tapしたnode以外を透明にする
        vm.cy.nodes().forEach(node => {
          if (node!==source_node) {
            if (vm.selected_flag) {
              if (!target_nodes.includes(node.data().id)) {
                node.addClass('unselected');
              }
            }
            else {
              if (!source_nodes.includes(node.data().id)) {
                node.addClass('unselected');
              }
            }
          } 
        })
        vm.selected_node = source_node.data().id;
        vm.next_nodes = target_nodes;
        vm.prev_nodes = source_nodes;
      })
    },
    graph_event_tap() {
      let vm = this;
      this.cy.on('dbltap', 'node', function(evt){
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
        //現在参照しているノードの割り出し
        let now = vm.keywords.find(e=>e.keyword === node.id());
        console.log(now);
        let resouce_rate = next_nodes.length != 0 ? prev_nodes.length/next_nodes.length : 0;
        // let node_info = "keyword = " + node.id() 
        //               + " \nweight = " + node._private.data.weight
        //               + " \nspeaker = " + now.speaker
        //               + "\nprev = " + prev_nodes
        //               + "\nnext = " + next_nodes
        //               + "\n資源率 = " + resouce_value;
        let node_info = {
          "keyword":node.id(),
          "weight":node._private.data.weight,
          "speaker":now.speaker,
          "prev":prev_nodes,
          "next":next_nodes,
          "resouce_rate":resouce_rate,
        }
        console.log(node_info);
        //alert(node_info);
        vm.node_info = node_info;
        vm.dialog = true;
        // console.log(node_info);
      });
    },
    set_data(labels, backgroundColor, data) {
      console.log('set_data')
      this.chart = !this.chart;
      this.chartData = {
        labels: labels,
        datasets: [
          {
            label: 'Data One',
            backgroundColor: backgroundColor,
            borderColor: 'transparent' ,
            data: data,
          }
        ]
      }
    },
    graph_event_tap_edge() {
      let vm = this;
      this.cy.on('tap', 'edge', function(evt){
        let label = [];
        let chartdata = {};
        //edgeのevent情報を取得
        let data = evt.target._private.data
        //console.log(evt);
        //console.log(data.id);
        //console.log('source: '+data.source);
        //console.log('target: '+data.target);
        vm.dataSource = data.source;
        vm.dataTarget = data.target;
        //元ノードをKeywordsから検索
        let source = null;
        vm.keywords.forEach((keyword)=>{
          if (keyword.keyword == data.source) {
            source = keyword;
          }
        })
        let nxsp = source.next_speakers;
        //console.log(nxsp)
        Object.keys(nxsp).forEach((key) => {
          if (key===data.target) {
            //console.log(key);
            //console.log(nxsp[key]); //この時点では重複は消えていない
            nxsp[key].forEach((key2) => {
              //console.log(key2)
              let speaker = String(key2);
              if (label.includes(speaker)) { //speaker in label は使えなかった
                console.log('sp in data')
                chartdata[speaker] += 1;
              }
              else {
                console.log('sp not in data')
                label.push(speaker);
                chartdata[speaker] = 1;
              }
            })
          }
        });
        //console.log(chartdata)
        let l = [];
        let b = [];
        let d = [];
        Object.keys(chartdata).forEach(speaker => {
          l.push(speaker);
          b.push(vm.clabel_setter(speaker));
          d.push(chartdata[speaker]);
        })
        vm.set_data(l, b, d)
      });
    },
    //demoサイトからそのまま引用
    isParentOfOneChild(node) {
      return node.isParent() && node.children().length === 1;
    },
    removeParent(parent) {
      parent.children().move({parent:null});
      parent.remove();
    },
    removeParentsOfOneChild() {
      this.cy.nodes().filter(this.isParentOfOneChild).forEach(this.removeParent);
    },
    move_contextmenu(x, y) {
      //初期値として，マウスの位置を左上にContextMenuを表示する
      this.contextmenu_style.left = x.toString() + "px";
      this.contextmenu_style.top = y.toString() + "px";
      //もし画面からはみ出した場合はその分ずらして表示する
      let wj = Number(this.$vuetify.breakpoint.width) - Number(x+350)
      let hj = Number(this.$vuetify.breakpoint.height) - Number(y+350);
      if (wj < 0) {
        //console.log('width over!!');
        this.contextmenu_style.left = Number(x + wj).toString() + "px";
      }
      if (hj < 0) {
        //console.log('height over!!');
        this.contextmenu_style.top = Number(y + hj).toString() + "px";
      }
    },
    make_analysis_contents() {
      //解析結果を作成していく
      this.analysis_dialog = true;
      //はじめての場合だけ実行する
      this.analysis_content = {};
      this.loading = true;
      setTimeout(()=>{
        this.analysis_content['message'] = 'テスト';
        this.loading = false;
      }, 4000)
    }
  },
  computed: {
  },
  watch: {
    labels: function() {
      console.log('labels are deleted!!')
      console.log(this.labels);
    }
  },
  mounted() {
    let vm = this;
    this.init_graph();
    this.graph_event_tap();
    this.graph_event_tap_edge();
    this.keywords = sample1;
    this.start_from_json(this.keywords);
    this.set_graph_style();
    this.cy.fit();
    const options = {
      grabbedNode: node => true, // filter function to specify which nodes are valid to grab and drop into other nodes
      dropTarget: (dropTarget, grabbedNode) => true, // filter function to specify which parent nodes are valid drop targets
      dropSibling: (dropSibling, grabbedNode) => true, // filter function to specify which orphan nodes are valid drop siblings
      newParentNode: (grabbedNode, dropSibling) => ({}), // specifies element json for parent nodes added by dropping an orphan node on another orphan (a drop sibling). You can chose to return the dropSibling in which case it becomes the parent node and will be preserved after all its children are removed.
      boundingBoxOptions: { // same as https://js.cytoscape.org/#eles.boundingBox, used when calculating if one node is dragged over another
        includeOverlays: false,
        includeLabels: true
      },
      overThreshold: 10, // make dragging over a drop target easier by expanding the hit area by this amount on all sides
      outThreshold: 10 // make dragging out of a drop target a bit harder by expanding the hit area by this amount on all sides
    };
    this.cy.compoundDragAndDrop(options);
    this.cy.on('remove', (event)=>{
      if (vm.removeEmptyParents ) {
        vm.removeParentsOfOneChild();
      }
    });
    this.cy.on('cdndout', (event, dropTarget) => {
      if( vm.removeEmptyParents && vm.isParentOfOneChild(dropTarget) ){
        vm.removeParent(dropTarget);
      }
    });
    //独自コンテキストメニューを追加
    window.addEventListener('contextmenu',(e)=>{
      e.preventDefault(); //元のコンテキストメニューを無効化
      this.contextmenu = false;
      this.move_contextmenu(e.pageX, e.pageY);
      this.contextmenu = true;
    });
    //コンテキストメニューの解除をクリックイベントに登録
    window.addEventListener('click',(e)=>{
      if ((e.target).toString()===('[object HTMLDivElement]')) {
        vm.remove_class();
        vm.removeParentsOfOneChild();
      }
      if ((e.target).toString()===('[object HTMLTableCellElement]')) {
        vm.remove_class();
        vm.removeParentsOfOneChild();
      }
      this.contextmenu = false;
    });

    //ダブルクリックしたnodeとそこから繋がるEdgeを強調する
    this.highlight_nodes_and_edges();

    //nodeとedge以外をtapしたときにselectedを解除
    this.cy.on('tap', (e)=>{
      if (Object.keys(e.target._private).includes('container')) {
        vm.remove_class();
        vm.removeParentsOfOneChild();
      }
    })

    //selectのオプション　defaultはsingle　addtiveは複数選択可能
    //this.cy.selectionType('additive');
  }
}
</script>

<style scoped>
#cy {
  background-color: #f3f3f2;
  height: calc(100vh);
}

.dialog {
  z-index: 5;
  position: fixed;
  inset: 0;
  margin: auto;
  width: 600px;
  height: 600px;
}

.contextmenu {
  z-index: 5;
  position: fixed;
  inset: 20px; /* top, right, bottom left を一括指定するもの */
  width: 300px;
  height: 230px;
}

.speaker_list {
  z-index: 5;
  position: fixed;
  top: 10px;
  right: 0px;
  width: 200px;
  /* height: 500px;  */ /* 勝手にテーブルサイズに調節されるためなし */
  background: rgba(243, 243, 242, 0); /* 背景のみ透過 */
}

.title {
  z-index: 5;
  position: fixed;
  inset: 10px;
  width: 400px;
  height: 100px;
  color: #E6E6E6;
  text-shadow: -1px -1px 0px #000000, 1px 1px #ffffff;
  background: rgba(243, 243, 242, 0); /* 背景のみ透過 */
  font-weight: bold;
}

.subtitle {
  z-index: 5;
  position: fixed;
  inset: 10px;
  top: 70px;
  width: 400px;
  height: 100px;
  color: black;
  background: rgba(243, 243, 242, 0); /* 背景のみ透過 */
  font-weight: bold;
}

.clabel {
  height: 30px;
  width: 30px;
  margin: 5px;
}

.selected {
  background: red;
}

/* レインボー */
.rainbow {
  text-indent: 0em;
  -webkit-animation: load5 1.1s infinite ease;
  animation: load5 1.1s infinite ease;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
}

/* ローディング画面 */
.loader {
  margin: 0px;
  font-size: 30px;
  width: 2.5em;
  height: 2.5em;
  border-radius: 50%;
  position: absolute;
  top: calc(50% - 1.25em);
  left: calc(50% - 1.25em);
  text-indent: 0em;
  -webkit-animation: load5 1.1s infinite ease;
  animation: load5 1.1s infinite ease;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
}
@-webkit-keyframes load5 {
  0%,
  100% {
    box-shadow: 0em -2.6em 0em 0em rgba(249, 31, 6, 1), 1.8em -1.8em 0 0em rgba(237, 18, 193, 0.7), 2.5em 0em 0 0em rgba(177, 17, 238, 0.7), 1.75em 1.75em 0 0em rgba(15, 44, 240, 0.7), 0em 2.5em 0 0em rgba(12, 238, 243, 0.7), -1.8em 1.8em 0 0em rgba(87, 243, 12, 0.7), -2.6em 0em 0 0em rgba(230, 241, 14, 0.7), -1.8em -1.8em 0 0em rgba(249, 146, 7, 0.7);
  }
  12.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(249, 146, 7, 0.7), 1.8em -1.8em 0 0em rgba(249, 31, 6, 1), 2.5em 0em 0 0em rgba(237, 18, 193, 0.7), 1.75em 1.75em 0 0em rgba(177, 17, 238, 0.7), 0em 2.5em 0 0em rgba(15, 44, 240, 0.7), -1.8em 1.8em 0 0em rgba(12, 238, 243, 0.7), -2.6em 0em 0 0em rgba(87, 243, 12, 0.7), -1.8em -1.8em 0 0em rgba(230, 241, 14, 0.7);
  }
  25% {
    box-shadow: 0em -2.6em 0em 0em rgba(230, 241, 14, 0.7), 1.8em -1.8em 0 0em rgba(249, 146, 7, 0.7), 2.5em 0em 0 0em rgba(249, 31, 6, 1), 1.75em 1.75em 0 0em rgba(237, 18, 193, 0.7), 0em 2.5em 0 0em rgba(177, 17, 238, 0.7), -1.8em 1.8em 0 0em rgba(15, 44, 240, 0.7), -2.6em 0em 0 0em rgba(12, 238, 243, 0.7), -1.8em -1.8em 0 0em rgba(87, 243, 12, 0.7);
  }
  37.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(87, 243, 12, 0.7), 1.8em -1.8em 0 0em rgba(230, 241, 14, 0.7), 2.5em 0em 0 0em rgba(249, 146, 7, 0.7), 1.75em 1.75em 0 0em rgba(249, 31, 6, 1), 0em 2.5em 0 0em rgba(237, 18, 193, 0.7), -1.8em 1.8em 0 0em rgba(177, 17, 238, 0.7), -2.6em 0em 0 0em rgba(15, 44, 240, 0.7), -1.8em -1.8em 0 0em rgba(12, 238, 243, 0.7);
  }
  50% {
    box-shadow: 0em -2.6em 0em 0em rgba(12, 238, 243, 0.7), 1.8em -1.8em 0 0em rgba(87, 243, 12, 0.7), 2.5em 0em 0 0em rgba(230, 241, 14, 0.7), 1.75em 1.75em 0 0em rgba(249, 146, 7, 0.7), 0em 2.5em 0 0em rgba(249, 31, 6, 1), -1.8em 1.8em 0 0em rgba(237, 18, 193, 0.7), -2.6em 0em 0 0em rgba(177, 17, 238, 0.7), -1.8em -1.8em 0 0em rgba(15, 44, 240, 0.7);
  }
  62.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(15, 44, 240, 0.7), 1.8em -1.8em 0 0em rgba(12, 238, 243, 0.7), 2.5em 0em 0 0em rgba(87, 243, 12, 0.7), 1.75em 1.75em 0 0em rgba(230, 241, 14, 0.7), 0em 2.5em 0 0em rgba(249, 146, 7, 0.7), -1.8em 1.8em 0 0em rgba(249, 31, 6, 1), -2.6em 0em 0 0em rgba(237, 18, 193, 0.7), -1.8em -1.8em 0 0em rgba(177, 17, 238, 0.7);
  }
  75% {
    box-shadow: 0em -2.6em 0em 0em rgba(177, 17, 238, 0.7), 1.8em -1.8em 0 0em rgba(15, 44, 240, 0.7), 2.5em 0em 0 0em rgba(12, 238, 243, 0.7), 1.75em 1.75em 0 0em rgba(87, 243, 12, 0.7), 0em 2.5em 0 0em rgba(230, 241, 14, 0.7), -1.8em 1.8em 0 0em rgba(249, 146, 7, 0.7), -2.6em 0em 0 0em rgba(249, 31, 6, 1), -1.8em -1.8em 0 0em rgba(237, 18, 193, 0.7);
  }
  87.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(237, 18, 193, 0.7), 1.8em -1.8em 0 0em rgba(177, 17, 238, 0.7), 2.5em 0em 0 0em rgba(15, 44, 240, 0.7), 1.75em 1.75em 0 0em rgba(12, 238, 243, 0.7), 0em 2.5em 0 0em rgba(87, 243, 12, 0.7), -1.8em 1.8em 0 0em rgba(230, 241, 14, 0.7), -2.6em 0em 0 0em rgba(249, 146, 7, 0.7), -1.8em -1.8em 0 0em rgba(249, 31, 6, 1);
  }
}
@keyframes load5 {
  0%,
  100% {
    box-shadow: 0em -2.6em 0em 0em rgba(249, 31, 6, 1), 1.8em -1.8em 0 0em rgba(237, 18, 193, 0.7), 2.5em 0em 0 0em rgba(177, 17, 238, 0.7), 1.75em 1.75em 0 0em rgba(15, 44, 240, 0.7), 0em 2.5em 0 0em rgba(12, 238, 243, 0.7), -1.8em 1.8em 0 0em rgba(87, 243, 12, 0.7), -2.6em 0em 0 0em rgba(230, 241, 14, 0.7), -1.8em -1.8em 0 0em rgba(249, 146, 7, 0.7);
  }
  12.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(249, 146, 7, 0.7), 1.8em -1.8em 0 0em rgba(249, 31, 6, 1), 2.5em 0em 0 0em rgba(237, 18, 193, 0.7), 1.75em 1.75em 0 0em rgba(177, 17, 238, 0.7), 0em 2.5em 0 0em rgba(15, 44, 240, 0.7), -1.8em 1.8em 0 0em rgba(12, 238, 243, 0.7), -2.6em 0em 0 0em rgba(87, 243, 12, 0.7), -1.8em -1.8em 0 0em rgba(230, 241, 14, 0.7);
  }
  25% {
    box-shadow: 0em -2.6em 0em 0em rgba(230, 241, 14, 0.7), 1.8em -1.8em 0 0em rgba(249, 146, 7, 0.7), 2.5em 0em 0 0em rgba(249, 31, 6, 1), 1.75em 1.75em 0 0em rgba(237, 18, 193, 0.7), 0em 2.5em 0 0em rgba(177, 17, 238, 0.7), -1.8em 1.8em 0 0em rgba(15, 44, 240, 0.7), -2.6em 0em 0 0em rgba(12, 238, 243, 0.7), -1.8em -1.8em 0 0em rgba(87, 243, 12, 0.7);
  }
  37.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(87, 243, 12, 0.7), 1.8em -1.8em 0 0em rgba(230, 241, 14, 0.7), 2.5em 0em 0 0em rgba(249, 146, 7, 0.7), 1.75em 1.75em 0 0em rgba(249, 31, 6, 1), 0em 2.5em 0 0em rgba(237, 18, 193, 0.7), -1.8em 1.8em 0 0em rgba(177, 17, 238, 0.7), -2.6em 0em 0 0em rgba(15, 44, 240, 0.7), -1.8em -1.8em 0 0em rgba(12, 238, 243, 0.7);
  }
  50% {
    box-shadow: 0em -2.6em 0em 0em rgba(12, 238, 243, 0.7), 1.8em -1.8em 0 0em rgba(87, 243, 12, 0.7), 2.5em 0em 0 0em rgba(230, 241, 14, 0.7), 1.75em 1.75em 0 0em rgba(249, 146, 7, 0.7), 0em 2.5em 0 0em rgba(249, 31, 6, 1), -1.8em 1.8em 0 0em rgba(237, 18, 193, 0.7), -2.6em 0em 0 0em rgba(177, 17, 238, 0.7), -1.8em -1.8em 0 0em rgba(15, 44, 240, 0.7);
  }
  62.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(15, 44, 240, 0.7), 1.8em -1.8em 0 0em rgba(12, 238, 243, 0.7), 2.5em 0em 0 0em rgba(87, 243, 12, 0.7), 1.75em 1.75em 0 0em rgba(230, 241, 14, 0.7), 0em 2.5em 0 0em rgba(249, 146, 7, 0.7), -1.8em 1.8em 0 0em rgba(249, 31, 6, 1), -2.6em 0em 0 0em rgba(237, 18, 193, 0.7), -1.8em -1.8em 0 0em rgba(177, 17, 238, 0.7);
  }
  75% {
    box-shadow: 0em -2.6em 0em 0em rgba(177, 17, 238, 0.7), 1.8em -1.8em 0 0em rgba(15, 44, 240, 0.7), 2.5em 0em 0 0em rgba(12, 238, 243, 0.7), 1.75em 1.75em 0 0em rgba(87, 243, 12, 0.7), 0em 2.5em 0 0em rgba(230, 241, 14, 0.7), -1.8em 1.8em 0 0em rgba(249, 146, 7, 0.7), -2.6em 0em 0 0em rgba(249, 31, 6, 1), -1.8em -1.8em 0 0em rgba(237, 18, 193, 0.7);
  }
  87.5% {
    box-shadow: 0em -2.6em 0em 0em rgba(237, 18, 193, 0.7), 1.8em -1.8em 0 0em rgba(177, 17, 238, 0.7), 2.5em 0em 0 0em rgba(15, 44, 240, 0.7), 1.75em 1.75em 0 0em rgba(12, 238, 243, 0.7), 0em 2.5em 0 0em rgba(87, 243, 12, 0.7), -1.8em 1.8em 0 0em rgba(230, 241, 14, 0.7), -2.6em 0em 0 0em rgba(249, 146, 7, 0.7), -1.8em -1.8em 0 0em rgba(249, 31, 6, 1);
  }
}


</style>