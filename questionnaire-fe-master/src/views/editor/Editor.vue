<template>
  <el-container class="editor">
    <el-header>
      <div class="editor-header">
        <span style="float: left; font-size: 20px ; color: white">
          <i class="el-icon-edit"></i>
           免费的在线问卷调查系统
        </span>
        <span style="float: right;font-size: 20px; color:white; cursor: pointer; margin-right: 60px" @click="logout()">登出</span>
      </div>
    </el-header>
    <el-container>
      <el-aside width="200px" height="4rem">
        <el-menu @select="handleMenuSelect" default-active="overview">
          <el-menu-item index="overview">
            <i class="el-icon-coin"></i>
            <span>问卷总览</span>
          </el-menu-item>
          <el-menu-item index="create">
            <i class="el-icon-plus"></i>
            <span>新建问卷</span>
          </el-menu-item>
        </el-menu>
      </el-aside>
      <el-container>
        <el-main>
          <router-view></router-view>
          <CreatePaper></CreatePaper>
      </el-main>
      </el-container>
    </el-container>
  </el-container>
</template>

<script>
import CreatePaper from './create/CreatePaper'
import { mapMutations, mapGetters, mapActions } from 'vuex'

export default {
  name: 'Editor',
  components: {
    CreatePaper
  },
  computed: {
    ...mapGetters(['userInfo'])
  },
  methods: {
    ...mapMutations([
      'set_createPaperVisible',
      'set_loginState',
      'set_userInfo'
    ]),
    ...mapActions(['logoutAct']),
    handleMenuSelect(index) {
      if (index === 'overview') {
        const route = this.$route.path
        if (route !== '/editor/overview') {
          this.$router.push({ name: index })
        }
      } else if (index === 'create') {
        this.set_createPaperVisible(true)
      }
    },
    logout() {
      console.log('logout')
      this.logoutAct().then(res => {
        if(res) {
          this.$router.go({ name: 'login' })
        }
      })
      // this.$router.push('/login')
    }
  }
}
</script>

<style>
* {
  font-family: "HarmonyOS Sans SC Black", BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
  Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.editor {
  height: 100%;
  left:0;
  overflow-x: hidden;
}

.editor-header {
  position:fixed; /*pinned*/
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
  z-index:3;
  overflow-x: hidden;

}

.el-aside {
  margin: 0;
}



</style>
