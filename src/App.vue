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
  font-family: "Xanh Mono", monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  height: 100vh;
  width: 100%;
  /* font-family: "Xanh Mono"; */
  background-color: #633636;
}
body {
  margin: 0;
  padding: 0;
}
</style>
