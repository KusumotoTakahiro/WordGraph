<template>
  <Mydatabase ref="my_database"></Mydatabase>
</template>
<script>
import kuromoji from 'kuromoji'
import Mydatabase from './Mydatabase.vue'

export default {
  components: {Mydatabase},
  data() {
    return {
      keywords: [
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
    }
  },
  methods: {
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
    //kuromojiによる形態素解析 input:一文の文章 output:品詞分類後のデータ
    async analysis(trainscript) {
      let vm = this;
      let analysised_data = [];
      kuromoji.builder({ dicPath: '/dict' }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        }
        else {
          const tokens = tokenizer.tokenize(trainscript);
          for (let token of tokens) {
            switch (token) {
              case '名詞':
                analysised_data.push({
                  keyword: token.basic_form,
                  position: 'noun',
                });
                break;
              case '形容詞':
                analysised_data.push({
                  keyword: token.basic_form,
                  position: 'adjective',
                });
                break;
              case '副詞':
                analysised_data.push({
                  keyword: token.basic_form,
                  position: 'adverb',
                });
                break;
              case '動詞':
                analysised_data.push({
                  keyword: token.basic_form,
                  position: 'verb',
                });
                break;
              default:
                //その他の品詞として無視（格納しない)
            }
          }
        }
      })
      return analysised_data;
    },
    //次の単語(next)の更新 input:現在latestがtrueの単語群 output:next更新後のkeyword配列
    update_next() {
    
    },
    //lateset属性の更新
    update_latest() {

    },
    //keyword配列に格納
    store_keyword() {
      
    },
  }
}
</script>
