<template>
  <div id="app">
    <p>
      <b style="font-size: 20px">Tetris</b> by
      <i
        ><a
          href="https://www.facebook.com/dung.nguyentien.eddie"
          target="_blank"
          rel="noopener noreferrer"
          >EddieOnTheCode</a
        ></i
      >
    </p>
    <Board :col="col" :row="row" :timeSpeed="timeSpeed" />
  </div>
</template>

<script>
import Board from "./components/Board";

export default {
  name: "App",
  components: {
    Board,
  },
  data() {
    return {
      col: 12,
      row: 20,
      timeSpeed: 300,
    };
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
