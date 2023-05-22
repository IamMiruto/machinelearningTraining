<template>
  <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
  <!-- <linearRegression />
  <div id="chart"></div>
  <div id="chartNonLinear"></div> -->
  <dinoTime1 />
  <!-- <trexComp /> -->
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import dinoTime1 from "./components/dinoTime.vue";
import linearRegression from "./components/linearRegression.vue";
//import an html file as a component
/*eslint-disable*/

export default {
  name: "App",
  components: {
    HelloWorld,
    dinoTime1,
    linearRegression,
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
