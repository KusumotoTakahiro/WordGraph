<template>
  
</template>
<script>
import kuromoji from 'kuromoji'

export default {
  data() {
    return {
      keywords: [],
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
    //kuromoji.jsを使って形態素解析を行う．
    async analysis() {
      let vm = this;
      kuromoji.builder({ dicPath: "/dict" }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        } 
        else {
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
  }
}
</script>
