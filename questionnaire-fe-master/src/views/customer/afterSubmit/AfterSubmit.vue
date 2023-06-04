<template>
  <div class="bg">
    <el-header>
      <div class="editor-header">
        <span style="float: left; font-size: 20px ; color:white">
          <i class="el-icon-edit"></i>
           免费的在线问卷调查系统
        </span>
      </div>
    </el-header>
    <div class="customer">
      <div class="container">
        <h1 class="title">完成填写！</h1>
        <el-button type="primary" @click="back">再填一次</el-button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  created() {
    const isRefreshed = localStorage.getItem('isRefreshed');
    if (!isRefreshed) {
      localStorage.setItem('isRefreshed', true);
      location.reload(); // 自动刷新页面
    } else {
      localStorage.removeItem('isRefreshed');
    }
  },
  methods: {
    back() {
      const paperId = this.$route.params.paperId
      console.log(paperId)
      this.$router.push({ name: 'customer', params: { paperId } })
    }
  }
}
</script>
<style>
.bg {
  background-color: #f8f8f8;
  height: 100vh;
  width: 100vw; /* 添加全视口宽度 */
  display: flex; /* 添加弹性布局 */
  justify-content: center; /* 添加水平居中对齐 */
  align-items: center; /* 添加垂直居中对齐 */
}

.customer {
  text-align: center;
  height: calc(100vh - 4rem); /* 添加计算高度 */
  display: flex; /* 添加弹性布局 */
  align-items: center; /* 添加垂直居中对齐 */
}

.container {
  background-color: #ffffff;
  border-radius: 10px;
  padding: 60px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 400px;
  max-width: 100%;
}

.title {
  font-size: 40px;
  color: #333;
  margin-bottom: 20px;
}

.el-button {
  margin-top: 20px;
  border-radius: 20px;
  padding: 10px 20px;
  font-size: 16px;
  background-color: #409eff;
}

.el-button:hover {
  background-color: #66b1ff;
}

.editor-header {
  position:fixed;
  width:100%;
  left: 0;
  top: 0;
  display: flex;
  height: 4rem;
  padding: 0 2rem;
  font-weight: 700;
  line-height: 4rem;
  background-color: #ff6600;
  justify-content: space-between;
  overflow-x: hidden; /* 添加隐藏超出部分 */
  z-index: 999; /* 添加最上层显示 */
}

.editor-header > * {
  width: auto; /* 添加自动宽度 */
  padding: 0; /* 添加无内边距 */
}
</style>
