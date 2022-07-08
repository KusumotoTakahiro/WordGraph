<template>
  
</template>
<script>
export default {
  data() {
    return {
      talk_title: "test", //本当は""で空の文字列にしておくが，検証用にtestとしておく．検証が終わったらもとに戻せ．
      keyword : {
        "keyword" : "",     //単語
        "weight" : 0,       //単語の出現回数による重みづけ
        "before_next" : {'none':true}, //この単語の次の単語 (描画前)
        "after_next" : {'none':true},  //この単語の次の単語 (描画後)
        "isLatest" : true,  //最新の単語かどうか
        "isInGraph" : false, //グラフに描画したかどうか
        "position" : "",     //品詞
        "speaker" : "",      //話者
      },
      read_keywords : {"keywords":null},
      s : '',
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
    async create_db() {
      let vm = this;
      try {
        const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
        await talk_ref.child(vm.keyword.keyword).set({
          "weight": vm.keyword.weight,
          "before_next": vm.keyword.before_next,
          "after_next": vm.keyword.after_next,
          "isLatest": vm.keyword.isLatest,
          "isInGraph": vm.keyword.isInGraph,
          "position": vm.keyword.position,
          "speaker": vm.keyword.speaker,
        });
        console.log('保存したのは'+vm.keyword.keyword);
      }
      catch(err){
        console.log(err);
      }
    },

    //isLatest＝trueのデータをデータベースから読み込んで値を返すメソッド
    async read_db_latest() {
      let vm = this;
      let keywords = null;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      await talk_ref.orderByChild('isLatest').startAt(true).endAt(true).once('value', 
      (snapshot) => {
        keywords = JSON.stringify(snapshot.val());
        //vm.$set(vm.read_keywords,"keywords", keywords);
        // console.log('From Mydatabase');
        // console.log('talk_title : '+vm.talk_title);
        // console.log(keywords);
      })
      // console.log('Here');
      // console.log(JSON.parse(keywords)); 
      return JSON.parse(keywords);
    },

    //全データをデータベースから読み込んで値を返すメソッド
    async read_db_all() {
      let keywords = null;
      let vm = this;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      await talk_ref.once('value', function(snapshot){
        keywords = snapshot.val();
      })
      return keywords;
    },

    //weightの値を更新するメソッド
    async update_db_weight(keyword, weight){
      let vm = this;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      await talk_ref.child(keyword).update({
        "weight": weight,
      });
    },

    //before_nextの値を更新するメソッド
    async update_db_bnext(keyword, b_next) {
      let vm = this;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      await talk_ref.child(keyword).update({
        "before_next": b_next,
      });
      console.log('update_db_bnext');
    },

    //isLatestの値を更新するメソッド
    async update_db_latest(keyword, flag){
      let vm = this;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      await talk_ref.child(keyword).update({
        "isLatest": flag,
      });
      console.log('update_db_latest');
    },

    //指定されたkeywordを削除するメソッド
    async delete_db(){
      let vm = this;
      const talk_ref = this.$fire.database.ref('talks/'+vm.talk_title);
      talk_ref.remove()
      .then(()=>{
        console.log('全データ削除');
      })
    }
  },
  
  //mountedの処理
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
