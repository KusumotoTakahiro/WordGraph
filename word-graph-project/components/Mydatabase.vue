<template>
  
</template>
<script>
export default {
  data() {
    return {
      talk_title: "",
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
    init_talk(talk_title){
      this.talk_title = talk_title;
    },
    set_keyword(keyword) {
      this.keyword.keyword = keyword;
    },
    set_weight(weight) {
      this.keyword.weight = weight;
    },
    set_before_next(before_next) {
      this.keyword.before_next = before_next;
    },
    set_after_next(after_next) {
      this.keyword.after_next = after_next;
    },
    set_isLatest(isLatest) {
      this.keyword.isLatest = isLatest;
    },
    set_isInGraph(isInGraph) {
      this.keyword.isInGraph = isInGraph;
    },
    set_position(position) {
      this.keyword.position = position;
    },
    set_speaker(speaker) {
      this.keyword.speaker = speaker;
    },
    //databaseに現在保存されているkeywordを一語ずつ登録するメソッド
    create_db() {
      let vm = this;
      try {
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        talk_ref.child(vm.keyword.keyword).set({
          "weight": vm.keyword.weight,
          "before_next": vm.keyword.before_next,
          "after_next": vm.keyword.after_next,
          "isLatest": vm.keyword.isLatest,
          "isInGraph": vm.keyword.isInGraph,
          "position": vm.keyword.position,
          "speaker": vm.keyword.speaker,
        });
      }
      catch(err){
        console.log(err);
      }
    },
    read_db() {
    },
    update_db() {
    },
    delete_db(){
    }
  },
  mounted() {
    // RealtimeDatabaseの更新を検知
    this.$fire.database.ref('talk/'+this.talk_title).on('value', (snapshot) => {
      console.log(snapshot.val());
    });
  }
}
</script>
<style scoped>

</style>
