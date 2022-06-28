<template>
  <div>
    <button @click="increment">send</button>
    <span>count:</span>
    <span>{{ count }}</span>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  mounted() {

    // RealtimeDatabaseの更新を検知
    this.$fire.database.ref('count').on('value', (snapshot) => {
      this.count = snapshot.val();
    })
  },
  methods: {

    // RealtimeDatabaseを更新
    increment() {
      this.$fire.database.ref('count').set(
        ++this.count
      );
    }
  }
}
</script>