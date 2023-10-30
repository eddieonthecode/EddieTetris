<template>
  <div class="board">
    <div class="board__game">
      <!-- <transition-group name="slide" tag="div"> -->
      <div
        class="row"
        v-for="(r, idxRow) in boardData"
        :style="{ 'flex-basis': `${(1 / row) * 100}%` }"
        :key="idxRow"
      >
        <div
          class="col"
          v-for="(c, idxCol) in r"
          :class="{ 'have-block': c }"
          :style="{
            'flex-basis': `${(1 / col) * 100}%`,
            'background-color': c && c.color ? c.color : 'unset',
          }"
          :key="idxCol"
        ></div>
      </div>
      <!-- </transition-group> -->
    </div>
    <div class="board__score">
      <div class="play" @click="isPlaying = !isPlaying">
        {{ isPlaying ? "Pause" : "Play" }}
      </div>
      <Score :score="score" :bestScore="bestScore" />
    </div>
  </div>
</template>

<script>
import { Colors } from "../models/model/Colors";
import { Blocks } from "../models/model/Blocks";
import { Moves } from "../models/enum/Moves";
import Score from "./Score";
export default {
  name: "Board",
  components: {
    Score,
  },
  props: {
    col: {
      type: Number,
      default: 10,
    },
    row: {
      type: Number,
      default: 10,
    },
    timeSpeed: {
      type: Number,
      default: 200,
    },
  },
  data() {
    return {
      boardData: [],
      isPlaying: false,
      isStart: false,
      color: Colors,
      currPosition: {},
      currBlock: [],
      interval: null,
      score: 0,
      bestScore: 0,
    };
  },
  mounted() {
    setInterval(() => {
      this.changeInterval();
    }, 10000);
    this.generateBoardData();
    this.getBestScore();
    window.addEventListener("keydown", (e) => {
      switch (e.key) {
        case "ArrowDown":
          e.preventDefault();
          this.moveBlock(Moves.BottomEnd);
          break;
        case "ArrowRight":
          this.moveBlock(Moves.Right);
          break;

        case "ArrowLeft":
          this.moveBlock(Moves.Left);
          break;
        case "ArrowUp":
          e.preventDefault();
          this.moveBlock(Moves.Rotate);
          break;
        default:
          break;
      }
    });
  },
  methods: {
    changeInterval() {
      var randomTimes = [100, 200, 300];
      var randomTime = randomTimes[Math.random() * (2 - 0) + 0];

      // Moi giay xuong mot lan
      clearInterval(this.interval);
      this.interval = setInterval(() => {
        this.moveBlock(Moves.Bottom);
      }, randomTime);
    },
    /**
     * Lay ra diem cao nhat
     * created by ntdung 11.03.2022
     */
    getBestScore() {
      if (localStorage.getItem("BestScore")) {
        this.bestScore = localStorage.getItem("BestScore");
      }
    },
    /**
     * Fix block vao bang
     * created by ntdung 11.03.2022
     */
    fixBlock(block, position) {
      block.forEach((row, idxRow) => {
        row.forEach((square, idxSquare) => {
          if (square) {
            var rowClone = [...this.boardData[idxRow + position.y]];
            rowClone[idxSquare + position.x] = { color: square, seted: true };
            this.$set(this.boardData, idxRow + position.y, rowClone);
          }
        });
      });
      // Kiem tra an diem
      this.checkGetScore();
      // Kiem tra thua
      if (this.checkEndGame()) {
        alert("End game");
        clearInterval(this.interval);
      } else {
        // Khoi tao lai
        clearInterval(this.interval);
        this.createABlock();
      }
    },
    /**
     * Kiem tra ket thuc tro choi
     * created by ntdung 11.03.2022
     */
    checkEndGame() {
      var valid = false;
      this.boardData[0].forEach((square) => {
        if (square) {
          valid = true;
        }
      });
      return valid;
    },
    /**
     * Kiem tra an diem
     * created by ntdung 11.03.2022
     */
    checkGetScore() {
      var newBoardData = JSON.parse(JSON.stringify(this.boardData));
      newBoardData = newBoardData.filter((row) => {
        return !row.every((square) => square);
      });
      for (var i = newBoardData.length; i < this.row; i++) {
        newBoardData.unshift(Array(this.col).fill(0));
        this.score += 10;
      }

      this.boardData = newBoardData;
    },
    /**
     * Tao mot khoi xep hinh moi
     * created by ntdung 10.03.2022
     */
    createABlock() {
      // Chon 1 khoi trong cac khoi da khai bao
      var randBlock = Blocks[Math.floor(Math.random() * Blocks.length)];
      // Chon 1 cach xoay cua khoi
      var randRotate = Math.floor(Math.random() * 4);
      // Chon 1 mau sac
      var randColor = Colors[Math.floor(Math.random() * Colors.length)];

      for (var rotate = 1; rotate <= randRotate; rotate++) {
        randBlock = this.rotateBlock(randBlock);
      }

      // Chuyen mau khoi
      randBlock.forEach((row, idxRow) => {
        row.forEach((square, idxSquare) => {
          if (square) {
            randBlock[idxRow][idxSquare] = randColor;
          }
        });
      });

      // Dat khoi hien tai
      this.currBlock = randBlock;

      // Dat vi tri hien tai
      this.currPosition = {
        x: Math.floor(this.col / 2 - randBlock[0].length / 2),
        y: 0,
      };

      // Dat khoi vao bang
      this.setBlockPosition(this.currPosition, randBlock);

      // Moi giay xuong mot lan
      this.interval = setInterval(() => {
        this.moveBlock(Moves.Bottom);
      }, this.timeSpeed);
    },
    /**
     * Di chuyen khoi
     * created by ntdung 10.03.2022
     */
    moveBlock(move) {
      switch (move) {
        case Moves.Left:
          if (this.checkValidMove(this.currBlock, this.currPosition, move)) {
            this.setBlockPosition({
              x: this.currPosition.x - 1,
              y: this.currPosition.y,
            });
          }
          break;
        case Moves.Right:
          if (this.checkValidMove(this.currBlock, this.currPosition, move)) {
            this.setBlockPosition({
              x: this.currPosition.x + 1,
              y: this.currPosition.y,
            });
          }
          break;
        case Moves.Bottom:
          if (this.checkValidMove(this.currBlock, this.currPosition, move)) {
            this.setBlockPosition({
              x: this.currPosition.x,
              y: this.currPosition.y + 1,
            });
          } else {
            // Fix vao trong bang
            this.fixBlock(this.currBlock, this.currPosition);
          }
          break;
        case Moves.Rotate:
          // Xoay block
          this.changeBlock(this.currBlock, this.currPosition, false);
          this.currBlock = this.rotateBlock(this.currBlock);
          this.changeBlock(this.currBlock, this.currPosition, true);
          break;
        case Moves.BottomEnd:
          for (
            var vertical = this.currPosition.y;
            vertical <= this.row - this.currBlock.length;
            vertical++
          ) {
            if (
              !this.checkValidMove(
                this.currBlock,
                { x: this.currPosition.x, y: vertical },
                Moves.Bottom
              )
            ) {
              // Xoa block cu
              this.changeBlock(this.currBlock, this.currPosition, false);
              // Fix block moi vao bang
              this.fixBlock(this.currBlock, {
                x: this.currPosition.x,
                y: vertical,
              });
              // Thoat khoi vong lap
              break;
            }
          }
          break;
        default:
          break;
      }
    },
    /**
     * Xoa khoi hoac them khoi
     * @param {boolean} mode xoa hoac them
     * created by ntdung 11.03.2022
     */
    changeBlock(block, position, mode) {
      block.forEach((row, idxRow) => {
        row.forEach((square, idxSquare) => {
          if (square) {
            var rowCloneDelete = [...this.boardData[idxRow + position.y]];
            rowCloneDelete[idxSquare + position.x] = mode
              ? { color: square, seted: false }
              : 0;
            this.$set(this.boardData, idxRow + position.y, rowCloneDelete);
          }
        });
      });
    },
    /**
     * Dat khoi vao trong
     * created by ntdung 10.03.2022
     */
    setBlockPosition(newPosition) {
      // Xoa block cu
      this.changeBlock(this.currBlock, this.currPosition, false);
      // Tao block moi
      this.changeBlock(this.currBlock, newPosition, true);
      this.currPosition = newPosition;
    },
    /**
     * Khoi tao bang trang
     * created by ntdung 10.03.2022
     */
    generateBoardData() {
      this.boardData = [];
      var emplyRow = Array(this.col).fill(0);
      this.boardData = Array(this.row).fill(emplyRow);
    },
    /**
     * Xoay ma tran mang 2 chieu
     * created by ntdung 10.03.2022
     */
    rotateBlock(matrix) {
      var resultBlock = [];

      matrix.forEach((row) => {
        row.forEach((block, index) => {
          if (!resultBlock[index]) resultBlock[index] = [];
          resultBlock[index].unshift(block);
        });
      });
      return resultBlock;
    },
    /**
     * Kiem tra buoc di chuyen hop le
     * created by ntdung 11.03.2022
     */
    checkValidMove(block, position, move) {
      var valid = true;
      // Chon ra vi tri moi va block moi
      var newPosition;
      switch (move) {
        case Moves.Left:
          newPosition = { x: position.x - 1, y: position.y };
          break;
        case Moves.Right:
          newPosition = { x: position.x + 1, y: position.y };
          break;
        case Moves.Bottom:
          newPosition = { x: position.x, y: position.y + 1 };
          break;
        case Moves.Rotate:
          block = this.rotateBlock(block);
          break;
        default:
          break;
      }
      // Kiem tra nam trong bang
      if (
        block &&
        (newPosition.x < 0 ||
          newPosition.x + block[0].length > this.col ||
          newPosition.y + block.length > this.row)
      ) {
        return false;
      }
      // Kiem tra block co trung voi phan da co trong bang chua
      block.forEach((row, idxRow) => {
        row.forEach((square, idxSquare) => {
          if (square) {
            if (
              this.boardData[idxRow + newPosition.y][
                idxSquare + newPosition.x
              ] &&
              this.boardData[idxRow + newPosition.y][idxSquare + newPosition.x]
                .seted
            ) {
              valid = false;
            }
          }
        });
      });
      return valid;
    },
  },
  watch: {
    isPlaying: function (val) {
      if (val) {
        this.createABlock();
      }
    },
  },
};
</script>

<style lang="scss">
.board {
  width: 500px;
  height: 500px;
  border-color: #888;
  border-style: double;
  border-width: 6px;
  border-radius: 4px;
  display: flex;
  padding: 10px;
  .board__game {
    width: 340px;
    height: 100%;
    border: 3px solid #666;
    border-radius: 4px;
    opacity: 0.8;
    background: #ccc;
    display: flex;
    flex-direction: column;
    .row {
      width: 100%;
      display: flex;
      .col {
        /* margin: 1px; */
        border: 2px solid transparent;
        /* transition: 0.1s all linear; */
        &.have-block {
          border: 2px solid #000;
          border-collapse: collapse;
          box-shadow: 4px 4px 8px rgba(0, 0, 0, 0.3);
        }
      }
    }
  }
  .board__score {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 10px 20px;
  }
}
/* Enter and leave animations can use different */
/* durations and timing functions.              */
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
/* >= Tablet */
/* Tablet - PC low resolution */
/* @media (max-width: 1023px) {
  #app {
    margin-top: 0;
    height: 100vh;
  }
  .board {
    width: 100vw;
    flex: 1;
    flex-direction: column-reverse;
  }
} */

/* > PC low resolution */
@media (min-width: 1024px) and (max-width: 1239px) {
}

/* PC hight resolution */
@media (min-width: 1240px) {
}
</style>