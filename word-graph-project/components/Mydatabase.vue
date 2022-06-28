<template>
  
</template>
<script>
export default {
  data() {
    return {
      keyword : {
        "keyword" : "",     //単語
        "weight" : 0,       //単語の出現回数による重みづけ
        "before_next" : [], //この単語の次の単語 (描画前)
        "after_next" : [],  //この単語の次の単語 (描画後)
        "isLatest" : true,  //最新の単語かどうか
        "isInGraph" : false, //グラフに描画したかどうか
        "position" : "",     //品詞
        "speaker" : "",      //話者
      }
    }
  },
  methods: {
    set_keyword(keyword) {
      this.keyword = keyword;
    },
    push_keyword() {
      try {
        this.$fire.database.push({
          "keyword" : this.keyword.keyword,     
          "weight" : this.keyword.weight,       
          "before_next" : this.keyword.before_next, 
          "after_next" : this.keyword.after_next,  
          "isLatest" : this.keyword.isLatest,  
          "isInGraph" : this.keyword.isInGraph, 
          "position" : this.keyword.position, 
          "speaker" : this.keyword.speaker,
      }).then(response => {
        console.log(response, "push a data to the database");
      })
      }
      catch(err){
        console.log(err);
      }
    },
    keyword_search_bykeyword(keyword) {
    },
    keyword_update(keyword) {
    },
    keyword_search_latest(){
    }
  },
  mounted() {
    let keyword = {
      "keyword" : "start",
      "weight" : 0,
      "before_next" : [], 
      "after_next" : [],  
      "isLatest" : true,  
      "isInGraph" : true, 
      "position" : "noun", 
      "speaker" : "default",
    }
    this.set_keyword(keyword);
    this.push_keyword();
  }
}
</script>
<style scoped>

</style>
