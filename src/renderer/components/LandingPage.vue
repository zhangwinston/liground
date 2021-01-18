<template>
  <div class="app">
    <MenuBar class="menubar" />
    <GameBoards class="gameboards" />
  </div>
</template>

<script>
import GameBoards from './GameBoards'
import MenuBar from './MenuBar.vue'

export default {
  name: 'LandingPage',
  components: {
    MenuBar,
    GameBoards
  },
  methods: {
    open (link) {
      this.$electron.shell.openExternal(link)
    }
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: #f9f9f9;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
.app {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  font-weight: 800;
  font-size: 12pt;
}
.menubar {
  text-align: center;
  margin-bottom: 10px;
}
</style>
