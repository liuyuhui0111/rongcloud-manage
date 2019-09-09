<template>
  <div class="question">
    <div class="header">
      <div class="back">
        <span class="iconfont iconfanhui"></span>
        返回
      </div>
      请将用户提交的问题进行分类
    </div>
  </div>
</template>
<script>
export default {
  name: 'question',
  data() {
    return {
      name: 'question',
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {

    },
  },
};
</script>
<style scoped>
.question{
  position: absolute;
  top:0;
  left: 0;
  bottom: 0;
  right: 0;
  background: #fff;
  z-index: 100;
}
</style>
