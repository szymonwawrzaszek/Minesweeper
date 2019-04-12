<template>
                <p class="cell bgimg-1 "  @click.left="cellClickLeft" @click.right="cellClickRight"  :class="[cell.white]" >
                    <img :src="cell.currentUrl" alt="">
                    <span  ><span v-if="cell.number > 0" >{{cell.number}}</span></span>
                </p>
        
</template>
<script>
import { eventBus } from "./../main";

export default {
  props: ["cell"],
  data: function() {
    return {
      mine: "",
      counterLeft: 0,
      counterRight: 0,
      flagUrl: require("./../assets/flag.png"),
      questionUrl: require("./../assets/questionMark.png"),
      mineUrl: require("./../assets/mine.png")
    };
  },
  methods: {
    cellClickLeft: function() {
      if (this.mine == "mine") {
        eventBus.$emit("loose", this.cell.id);
        this.cell.currentUrl = this.mineUrl;
        alert("You Loose!\nHehe xd");
        return;
      }
      this.counterLeft++;
      eventBus.$emit("timeStart", 0);
      if (this.cell.number == 0) eventBus.$emit("white", this.cell.id);

      if (this.counterRight == 1) this.mineCounterChange();
      
      this.cell.white = "white";
      this.mine = "";
      this.cell.currentUrl = "";
      eventBus.$emit("checkWin", 0);
    },
    cellClickRight: function() {
      if (this.counterRight == 0) {
        this.cell.currentUrl = this.flagUrl;
        this.mineCounterChange();
      }
      if (this.counterRight == 1) {
        this.cell.currentUrl = this.questionUrl;
        this.mineCounterChange();
      }
      if (this.counterRight == 2) {
        this.cell.currentUrl = "";
        this.counterRight = -1;
      }
      this.counterRight++;
      eventBus.$emit("checkWin", 0);
    },
    mineCounterChange: function() {
      eventBus.$emit("mineCounterChange", this.counterRight);
    }
  },
  created: function() {
    if (this.cell.hasMine == true) {
      this.mine = "mine";
    } else this.mine = "";
  }
};
</script>

<style>
.cell {
  width: 55px;
  height: 55px;
  margin: 0;
  border: 1px solid rebeccapurple;
  background-color: blue;
  float: left;
}
.cell:hover {
  cursor: pointer;
  background-color: aqua;
}
.mine {
  background-image: url("../assets/mine.png");
  position: relative;
  background-repeat: no-repeat;
  background-position: center;
}
.bgimg-1 {
  position: relative;
  background-repeat: no-repeat;
  background-position: center;
}
.white {
  background-color: azure;
  font-size: 30px;
  text-align: center;
  pointer-events: none;
}
</style>


