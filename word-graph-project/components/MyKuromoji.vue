<template>
  <Mydatabase ref="db"></Mydatabase>
</template>
<script>
import kuromoji from 'kuromoji'
import Mydatabase from './Mydatabase.vue'

export default {
  components: {Mydatabase},
  data() {
    return {
    }
  },
  methods: {
    //keywordがすでに出たものかを確認する関数
    check_duplicate(keyword) {
      const db = this.$refs.db;
      let keywords = db.read_db_all(); //keywordsオブジェクトごと取り出す．
      let key = Object.keys(keywords);
      let weight = 0; //weight : １以上=>重複 ０=>新規単語
      key.forEach(function(value, index){
        if (keyword == value) {
          weight = keywords[index].weight 
        }
      });
      return weight;
    },

    //kuromojiによる形態素解析 input:一文の文章 output:品詞分類後のデータ(analysised_data)
    async analysis(trainscript) {
      let vm = this;
      let analysised_data = [];
      kuromoji.builder({ dicPath: '/dict' }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        }
        else {
          console.log('エラーはなし');
          const tokens = tokenizer.tokenize(trainscript);
          console.log(tokens);
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
          console.log(analysised_data);
        }
      })
      return analysised_data;
    },

    //update_nextではgraphの骨格に当たる名詞のnextを更新している．他の品詞は別で更新してdbに保存する
    //既出単語の次の単語(before_next)の更新 input:現在latestがtrueの単語群とanalysised_data output:なし
    update_next(analysised_data) {
      const db = this.$refs.db;
      let keywords = db.read_db_latest();
      console.log('From MyKuromoji update_next');
      console.log(keywords);
      let key = Object.keys(keywords);
      key.forEach(function(index, value){
        let b_next = keywords[index].before_next;
        for (let data of analysised_data) {
          if (data.position=="noun") {
            b_next.push(data.keyword);
          }
        }
        db.update_db_bnext(value, b_next); //before_nextを更新してdbに保存
        db.update_db_latest(value, false); //latestを更新してdbに保存
      })
    },

    //新規keyword(名詞，形容詞，動詞，副詞)をdbに登録できる形に加工してdbに保存する．
    //input: 新規に登録する単語, dbからのkeyword情報, 話し手　output:なし
    store_keyword(data, b_next, speaker) {
      const vm = this;
      const db = vm.$refs.db;
      switch (data.position) {
        case 'noun': //名詞の場合
          const weight = vm.check_duplicate(data.keyword);
          if (weight!=0) {
            //keywordが既出の場合=>update_db
            db.update_db_weight(data.keyword, weight);
            db.update_db_latest(data.keyword, true);
          }
          else {
            //keywordが新規の場合=>create_db
            db.set_keyword(data.keyword);
            db.set_weight(weight);
            db.set_before_next(b_next);
            db.set_after_next([]);
            db.set_isLatest(true);
            db.set_isInGraph(false);
            db.set_position("noun");
            db.set_speaker(speaker);
            db.create_db();
          }
          break;
        case 'adjective': //形容詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next([]);
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("adjective");
          db.set_speaker(speaker);
          db.create_db();
          break;
        case 'adverb': //副詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next([]);
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("adverb");
          db.set_speaker(speaker);
          db.create_db();
          break;
        case 'verb': //動詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next([]);
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("verb");
          db.set_speaker(speaker);
          db.create_db();
          break;
      }
    },

    //任意のspeakerの一文に対してkeywordを保存する．
    //input: 形態素解析した後の単語群，話し手 output:なし
    process_keyword(analysised_data, speaker) {
      let noun_keyword = [];
      let verb_keyword = [];
      let b_next = [];
      analysised_data.forEach(function(value){
        if (value.position=='noun') {
          noun_keyword.push(value.keyword);
        }
        else if (value.position=='verb') {
          verb_keyword.push(value.keyword);
        }
      })
      analysised_data.forEach(function(value){
        switch (value.position) {
          case 'noun':
            b_next = []
            break;
          case 'adjective':
            b_next = noun_keyword;
            break;
          case 'adverb':
            b_next = verb_keyword;
            break;
          case 'verb':
            b_next = noun_keyword;
            break;
          default:
            b_next = [];
            break;
        }
        this.store_keyword(value, b_next, speaker)
      })
    }
  }
}
</script>
