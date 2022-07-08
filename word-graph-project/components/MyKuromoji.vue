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
    async check_duplicate(keyword) {
      const db = this.$refs.db;
      let keywords = await db.read_db_all(); //keywordsオブジェクトごと取り出す．
      let key = Object.keys(keywords);
      console.log('check_duplicate');
      console.log('key='+key);
      let weight = 0; //weight : １以上=>重複 ０=>新規単語
      key.forEach(function(value, index){
        if (keyword == value) {
          weight = keywords[index].weight 
        }
      });
      return weight;
    },

    //kuromojiによる形態素解析 input:一文の文章 output:品詞分類後のデータ(analysised_data)
    analysis(transcript) {
      let analysised_data = [];
      kuromoji.builder({ dicPath: '/dict' }).build((err, tokenizer) => {
        if (err) {
          console.log(err);
        }
        else {
          const tokens = tokenizer.tokenize(transcript);
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
          return analysised_data;
        }
      })
    },

    //update_nextではgraphの骨格に当たる名詞のnextを更新している．他の品詞は別で更新してdbに保存する
    //既出単語の次の単語(before_next)の更新 input:現在latestがtrueの単語群とanalysised_data output:なし
    async update_next(analysised_data) {
      const db = this.$refs.db;
      let keywords = await db.read_db_latest();
      //keyにNullが入る場合にエラーになっているので例外処理が必要
      console.log('From MyKuromoji update_next');
      try {
        Object.keys(keywords).forEach(value=>{
          if (keywords[value].position=='noun') {
            let b_next = keywords[value].before_next;
            const isLatest_flag = false; //trueなら次の名詞があるのでOK，falseなら次の名詞がないのでNG
            for (let data of analysised_data) {
              console.log(data);
              if (data.position=="noun") {
                console.log(!b_next.hasOwnProperty(data.keyword));
                //新キーワードが未登録ならオブジェクトに追加する．
                if (!b_next.hasOwnProperty(data.keyword)){
                  // apple:false の形でobjectに追加．
                  b_next[data.keyword] = false;
                  isLatest_flag = true;
                }
              }
            }
            //↓↓↓↓↓もっと良い非同期処理を待つ書き方にするべき↓↓↓↓↓↓
            console.log(b_next);
            db.update_db_bnext(value, b_next); //before_nextを更新してdbに保存
            if (isLatest_flag) {
              db.update_db_latest(value, false); //latestを更新してdbに保存
            }
          }
        })
      }catch(e) {
        console.log('error = '+e.message);
        console.log('try文が実行されてない？');
      }
    },

    //新規keyword(名詞，形容詞，動詞，副詞)をdbに登録できる形に加工してdbに保存する．
    //input: 新規に登録する単語, dbからのkeyword情報, 話し手　output:なし
    async store_keyword(data, b_next, speaker) {
      console.log('From MyKuromoji store_keyword');
      const vm = this;
      const db = vm.$refs.db;
      switch (data.position) {
        case 'noun': //名詞の場合
          const weight = await vm.check_duplicate(data.keyword);
          console.log('weight='+weight);
          if (weight!=0) {
            //keywordが既出の場合=>update_db
            await db.update_db_weight(data.keyword, weight);
            await db.update_db_latest(data.keyword, true);
          }
          else {
            //keywordが新規の場合=>create_db
            db.set_keyword(data.keyword);
            db.set_weight(weight);
            db.set_before_next(b_next);
            db.set_after_next({'none':true});
            db.set_isLatest(true);
            db.set_isInGraph(false);
            db.set_position("noun");
            db.set_speaker(speaker);
            await db.create_db();
          }
          break;
        case 'adjective': //形容詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next({'none':true});
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("adjective");
          db.set_speaker(speaker);
          await db.create_db();
          break;
        case 'adverb': //副詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next({'none':true});
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("adverb");
          db.set_speaker(speaker);
          await db.create_db();
          break;
        case 'verb': //動詞の場合
          db.set_keyword(data.keyword);
          db.set_weight(weight);
          db.set_before_next(b_next);
          db.set_after_next({'none':true});
          db.set_isLatest(true);
          db.set_isInGraph(false);
          db.set_position("verb");
          db.set_speaker(speaker);
          await db.create_db();
          break;
      }
      console.log('success!! store keyword')
    },

    //任意のspeakerの一文に対してkeywordを保存する．
    //input: 形態素解析した後の単語群，話し手 output:なし
    async process_keyword(analysised_data, speaker) {
      console.log('From MyKuromoji process_keyword');
      let noun_keyword = []; //一時的にanalysised_dataから名詞だけの配列を保存する．
      let verb_keyword = []; //一時的にanalysised_dataから動詞だけの配列を保存する．
      for (let data in analysised_data) {
        if (data.position == 'noun') {
          noun_keyword.push(data.keyword);
        }
        else if (data.positon == 'verb') {
          verb_keyword.push(data.keyword);
        }
      }
      for (let data in analysised_data) {
        let b_next = {'none':true};
        switch (data.position) {
          case 'noun':
            //b_nextはそのまま
            break;
          case 'adjective':
            for (let nk in noun_keyword) {
              b_next[nk] = false; //falseは未描画の意味．trueで描画済み
            }
            break;
          case 'adverb':
            for (let vk in verb_keyword) {
              b_next[vk] = false;
            }
            break;
          case 'verb':
            for (let nk in noun_keyword) {
              b_next[nk] = false;
            }            
            break;
          default:
            //b_nextはそのまま
            break;
        }
        await this.store_keyword(data, b_next, speaker)
      }
    },
    async db_test() {
      //データベースにつなぐ(Mydatabase.vueにつないでいるだけですが)
      const vm = this;
      const db = vm.$refs.db;
      //一旦全データ削除
      db.delete_db();

      //書き込み
      db.set_keyword('sample');
      db.set_weight(0);
      db.set_before_next({'none':true});
      db.set_after_next({'none':true});
      db.set_isLatest(true);
      db.set_isInGraph(false);
      db.set_position("noun");
      db.set_speaker("default");
      await db.create_db();

      //ループ書き込み
      for (let i = 1; i < 10; i++) {
        let isLatest = false;
        if (i%2==0) {
          isLatest = true;
        }
        db.set_keyword('sample'+String(i));
        db.set_weight(0);
        db.set_before_next({'none':true});
        db.set_after_next({'none':true});
        db.set_isLatest(isLatest);
        db.set_isInGraph(false);
        db.set_position("noun");
        db.set_speaker("default");
        await db.create_db();
      }

      //更新
      await db.update_db_bnext('sample3', 
      {
        'none':true,
        'test1':false,
        'test2':false,
        'test3':false,
      }); //before_nextを更新してdbに保存

      //読み取り
      //isLatest=trueのみ読み取り
      let keywords = await db.read_db_latest();
      console.log('keys='+Object.keys(keywords));
      //全データ読み取り
      let keywords_all = await db.read_db_all();
      console.log('keys='+Object.keys(keywords_all))
    }
  }
}
</script>
