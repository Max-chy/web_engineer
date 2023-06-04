<template>
  <div class='question'>
    {{ question }}
  </div>
</template>

<script>
export default {
  props: {
    question: Object
  },
  watch: {
    $route(to, from) {
      window.location.reload(); //监测到路由发生跳转时刷新一次页面
    },
  },
  mounted() {
    console.log('question')
    console.log(this.question)
  }
}
</script>

<style>
.question {
  min-height: 200px;
}
</style>
