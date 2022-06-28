<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="12" md="12" lg="12" xl="12">
      <v-card elevation="24" outlined shaped >
        <v-card-title id="title">Word Graph Project</v-card-title>
        <v-card-text>
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
          this is a word-graph-project.
        </v-card-text>
      </v-card>
      <v-card id="form">
        <v-card-subtitle>これから始める会話に名前をつけてください</v-card-subtitle>
        <v-container>
          <v-form v-model="valid" ref="form">
          <v-text-field
          v-model="talk_title"
          :rules="nameRules"
          :counter="20"
          label="talk_title" 
          autofocus
          ></v-text-field>
          <v-btn @click="go_to_main()"  
        class="light-green lighten-5 black--text"
        outlined
        id="submit">対話を開始</v-btn>
        </v-form>
        </v-container>
        <Mydatabase ref="db"></Mydatabase>
      </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import Mydatabase from '../components/Mydatabase.vue'

export default {
  components: { Mydatabase },
  name: 'IndexPage',
  layout: "welcome",
  data() {
    return {
      valid: false,
      talk_title:'',
      nameRules: [
        v => !!v || 'talk_title is required',
        v => v.length <= 20 || 'talk_title must be less than 20 characters',
      ],
    }
  },
  methods: {
    go_to_main() {
      if (this.$refs.form.validate()){
        //パリデーションが通った時だけ行われる処理
        const db = this.$refs.db;
        db.init_talk(this.talk_title);
        db.set_keyword("start");
        db.set_isLatest(true);
        db.set_isInGraph(true);
        db.set_position("noun");
        db.set_speaker("default");
        db.create_db();
        this.$router.push('/inspire');
      }
    },
  },
}
</script>
<style scoped>
#title {
  font-size: 40px;
  color: #F1F8E9;
  font-family: serif;
}
#form {
  text-align: center;
  margin: auto;
  width: calc(100%/3*2);
  height: calc(100%/3*2);
  margin-top: 4rem;

}
#submit {
  font-family: serif;
  font-size: 1.2rem;
}
</style>