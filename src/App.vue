<template>
  <div id="app">
    <MainMap />
  </div>
</template>

<script>
import MainMap from "@/components/MainMap";

export default {
  components: {
    MainMap,
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Poppins:100,200,300,400,500,600,700,800,900");

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Poppins", sans-serif;
}

body {
  background: #000;
}
</style>
