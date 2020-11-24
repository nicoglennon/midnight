<template>
  <div id="app">
    <Main msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
import Main from "./components/Main.vue";

export default {
  name: "app",
  components: {
    Main,
  },
};
</script>

<style>
#app {
  font-family: "Press Start 2P", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #333;
  height: 100vh;
  width: 100%;
  background-color: mistyrose;
}
body {
  margin: 0;
  padding: 0;
}
</style>
