<template>
  <div class="header">
    <img src="../assets/header.png" @click="toUrl" alt="">
  </div>
</template>
<script>
  export default {
    name: 'v-header',
    props: {
      url: {
        type: String
      }
    },
    methods: {
      toUrl: function () {
        if (this.url) {
          this.$router.push(`/introduce/${this.url}`)
        }
      }
    }
  }
</script>
<style scoped>
  .header{
    height: 100px;
    width: 100%;
    padding: 0;
    z-index: 1;
  }
  .header img{
    height: 100%;
    width: 100%;
  }
</style>
