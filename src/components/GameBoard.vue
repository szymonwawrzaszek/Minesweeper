<template>
    <div >
        <div v-if="!gameStarted">
          <p> Width:
            <input type="text" class="form-control width20" v-model="height"></p>  
          <p> Height:  
            <input type="text" class="form-control width20" v-model="width"></p>
          <p> Number of mines:  
            <input type="text" class="form-control width20" v-model="mines"> </p>
          <p> <button class="btn btn-primary" @click="startGame" >Start new game</button> </p>       
        </div>
        <div id="board" v-if="gameStarted">
          <p>
            <div style="display: inline-block; margin: 0;">Time: <span v-if="timerStarted">{{time}}</span><span v-else>{{timeStop}}</span></div>
            <div style="display: inline-block;">Remain mines: {{mines}}</div>
          <p>  
          <div v-for="element in cells" style="clear: both; display: block;" :class="[state]">
              <app-cell v-for="e in element" :cell="e"></app-cell>
          </div>
            <button class="btn btn-primary" @click="stopGame" >Stop Game</button>
        </div>              
    </div>   
</template>

<script>
import Cell from "./cell.vue";

import { eventBus } from "./../main";

export default {
  data: function() {
    return {
      width: 10,
      height: 10,
      time: "",
      timeStop: "00:00",
      seconds: 0,
      minutes: 0,
      mines: 1,
      minesStart: 0,
      state: "",
      leftClicked: 0,
      gameStarted: false,
      timerStarted: false,
      cells: [],
      oneCell: {
        id: 0,
        hasMine: false,
        counterRight: 0,
        counterLeft: 0,
        number: 0,
        white: "",
        currentUrl: ""
      }
    };
  },
  components: {
    "app-cell": Cell
  },
  methods: {
    startGame: function() {
      if (this.mines >= this.width * this.height) {
        alert(
          `Too many mines\nMax number of mines is ${this.width * this.height -
            1}`
        );
        this.mines = this.width * this.height - 1;
      } else if (this.mines < 1) {
        alert("At least one mine must be setted");
        this.mines = 1;
      } else if (this.width > 30) {
        alert(`Max width is 30`);
        this.width = 30;
      } else if (this.height > 16) {
        alert(`Max heigth is 16`);
        this.height = 16;
      } else {
        this.gameStarted = true;
        this.state = "";
        this.minesStart = this.mines;
        this.generateMap();
        this.calculateNumber();
      }
    },
    stopGame: function() {
      this.gameStarted = false;
      this.mines = this.minesStart;
      this.timerStarted = false;
      this.timeStop = "00:00";
    },
    generateMap: function() {
      this.cells = [];
      var cellsRow = [];
      var id = 0;
      var chance = false;
      if (this.mines / (this.width * this.height) < 0.5) chance = false;
      else chance = true;
      for (var i = 0; i < this.width; i++) {
        cellsRow = [];
        for (var j = 0; j < this.height; j++) {
          this.oneCell = {
            id: id,
            hasMine: chance,
            number: 0,
            white: "",
            currentUrl: ""
          };
          cellsRow.push(this.oneCell);
          id++;
        }
        this.cells.push(cellsRow);
      }
      this.mineShuffle();
    },
    calculateNumber() {
      var count = 0;
      for (var i = 0; i < this.width; i++) {
        for (var j = 0; j < this.height; j++) {
          for (var a = -1; a < 2; a++) {
            for (var b = -1; b < 2; b++) {
              if (
                i + a >= 0 &&
                j + b >= 0 &&
                i + a < this.width &&
                j + b < this.height
              ) {
                if (this.cells[i + a][j + b].hasMine == true) count++;
              }
            }
          }
          this.cells[i][j].number = count;
          count = 0;
        }
      }
    },
    mineShuffle: function(chance) {
      var x = 0;
      var y = 0;
      if (this.mines < 0.5 * this.width * this.height) {
        for (var i = 0; i < this.mines; i++) {
          while (1) {
            x = Math.floor(Math.random() * this.width);
            y = Math.floor(Math.random() * this.height);
            if (this.cells[x][y].hasMine == false) {
              this.cells[x][y].hasMine = true;
              break;
            }
          }
        }
      } else {
        for (var i = 0; i < this.width * this.height - this.mines; i++) {
          while (1) {
            x = Math.floor(Math.random() * this.width);
            y = Math.floor(Math.random() * this.height);
            if (this.cells[x][y].hasMine == true) {
              this.cells[x][y].hasMine = false;
              break;
            }
          }
        }
      }
    },
    reveal: function(x, y) {
      //console.log(x, y);
      for (var a = -1; a < 2; a++)
        for (var b = -1; b < 2; b++) {
          if ((a == 0) & (b == 0)) continue;
          this.revealNext(x + a, y + b);
        }
    },
    revealNext: function(x, y) {
      if (x < 0 || x >= this.width || y < 0 || y >= this.height) return;
      var cell = this.cells[x][y];
      if (cell.white == "white") return;
      if (cell.number > 0) {
        cell.white = "white";
        return;
      }
      if (cell.currentUrl != "") this.mines += 1;
      cell.currentUrl = "";
      cell.white = "white";
      this.reveal(x, y);
    }
  },

  created() {
    eventBus.$on("mineCounterChange", num => {
      if (num == 0) this.mines -= 1;
      else this.mines += 1;
    }),
      eventBus.$on("checkWin", none => {
        for (var i = 0; i < this.width; i++) {
          for (var j = 0; j < this.height; j++) {
            if (
              this.cells[i][j].hasMine == false &&
              this.cells[i][j].white == ""
            ) {
              return;
            } else if (this.cells[i][j].hasMine == true) continue;
          }
        }
        alert("You win this time!\nCongratulation!");
        this.state = "stop";
        this.timeStop = this.time;
        this.timerStarted = false;
      }),
      eventBus.$on("white", id => {
        var x = Math.floor(id / this.height);
        var y = id % this.height;
        this.reveal(x, y);
      }),
      eventBus.$on("timeStart", none => {
        if (this.timerStarted == false) {
          if (this.time == "") this.time = "00:00";
          else this.seconds = -1;
          this.minutes = 0;
          this.timerStarted = true;
        }
      }),
      eventBus.$on("loose", id => {
        this.timeStop = this.time;
        this.timerStarted = false;
        this.state = "stop";
      });
  },
  watch: {
    time: function() {
      var instance = this;
      setTimeout(function() {
        var min;
        instance.seconds += 1;
        if (instance.minutes < 10) {
          min = `0${instance.minutes}:`;
        } else {
          min = `${instance.minutes}:`;
        }
        if (instance.seconds < 10) {
          instance.time = `${min}0${instance.seconds}`;
        } else {
          instance.time = `${min}${instance.seconds}`;
        }
        if (instance.seconds == 59) {
          instance.seconds = 0;
          instance.minutes += 1;
        }
      }, 1000);
    },
    miness: function() {
      if (this.mines != 0) return;
      else {
        for (var i = 0; i < this.width; i++) {
          for (var j = 0; j < this.height; j++) {
            console.log(this.cells[i][j].hasMine);
            if (
              this.cells[i][j].hasMine == false &&
              this.cells[i][j].white == ""
            ) {
              //alert(i * 10 + j);
              console.log(this.cells[i][j]);

              return;
            } else if (this.cells[i][j].hasMine == true) continue;
          }
        }
        alert("You win this time!\nCongratulation!");
        this.state = "stop";
        this.timeStop = this.time;
        this.timerStarted = false;
      }
    }
  }
};
</script>

<style scoped>
#board {
  border: 1px solid black;
  margin: 0;
}
.width20 {
  width: 20%;
}
.stop {
  pointer-events: none;
}
</style>
