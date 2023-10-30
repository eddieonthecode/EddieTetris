<template>
  <div class="score">
    <div><b>Your Score</b></div>
    <div
      class="score__number"
      style="background: radial-gradient(#0ff, #9198e5) !important"
    >
      {{ score }}
    </div>
    <div><b>Best Score</b></div>
    <div
      class="score__number"
      style="background: radial-gradient(#ff0, #e66465) !important"
    >
      {{ bestScore }}
    </div>
  </div>
</template>

<script>
export default {
  name: "Score",
  props: {
    score: {
      type: Number,
      default: 0,
    },
    bestScore: {
      type: Number,
      default: 0,
    },
  },
};
</script>

<style lang="scss">
.score {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  width: 100%;
  .score__number {
    margin-top: 10px;
    font-size: 30px;
    padding: 10px;
    background-color: #666;
    border-radius: 4px;
    width: 100%;
    color: #fff;
    box-sizing: border-box;
  }
}
</style>