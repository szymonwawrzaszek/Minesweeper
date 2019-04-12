<template>
    <div class="container">
        <app-header/>
        <hr>
       <app-GameBoard></app-GameBoard>
        <hr>
        <!-- <app-footer></app-footer>  --> 
        </div>
</template>

<script>
import Header from "./Header.vue";
import Footer from "./Footer.vue";
import GameBoard from "./components/GameBoard.vue";

export default {
  components: {
    "app-header": Header,
    "app-footer": Footer,
    "app-GameBoard": GameBoard
  },
  data: function() {
    return {};
  }
};
</script>

<style>
.container {
  margin: 20px 20px 20px 40px;
}
.buttonsContainerExternal {
  text-align: center;
}
.buttonsContainerInternal {
  margin: 0 auto;
  display: inline-block;
}
button {
  margin: 0px 0px 5px 10px;
}
</style>
