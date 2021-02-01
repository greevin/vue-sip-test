<template>
  <div id="app">
    <CallSip msg="Vue JsSIP Test" />
  </div>
</template>

<script>
import CallSip from "./components/CallSip.vue";

export default {
  name: "App",
  components: {
    CallSip,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
