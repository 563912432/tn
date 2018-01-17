<template>
  <div class="footer">
    <div class="f-title">
      本课程版权所有人为“哼哼成长”　盗用必究
    </div>
  </div>
</template>
<script>
  export default {
    name: 'footer'
  }
</script>
<style scoped>
  .footer {
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    height: 40px;
    font-size: 12px;
    background-color: #4293cc;
    z-index: 1;
  }
  .footer .f-title{
    flex: 1;
    text-align: center;
    padding: 5px;
  }
</style>