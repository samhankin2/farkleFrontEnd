<template lang="pug">
v-app
  v-container(fill-height fluid)
    v-row
      p Player 1 Score {{player1.totalScore}}
      v-spacer
      p player {{player}} turn
      v-spacer
      p Player 2 Score {{player2.totalScore}}

    v-row(align="center" justify="center")
      v-col(col="2" v-for="die,i in dice" justify="center" align="center")
        .die
          p(@click="pickup(die,i)") {{die}}
    v-row(align="center" justify="center")
      v-col(col="2" v-for="die,i in inhand" justify="center" align="center")
        .die
          p(@click="remove(die,i)" style="background:darkred") {{die}}
    v-row(align="center" justify="center")  
      v-col(col="4" justify="center" align="center" @click="rollDice" v-if="rollDiceFlag") Roll Dice
      v-col(col="4" justify="center" align="center" @click="bankDice" v-if="roundPoints>0") Bank Points
      v-col(col="4" justify="center" align="center" @click="finishTurn" v-if="roundPoints>0") Finish Turn
    v-row(align="center" justify="center")  
      v-col(col="6" justify="center" align="center" @click="rollDice") Points: {{totalScore}}
      v-col(col="6" justify="center" align="center" @click="rollDice") RoundPoints {{roundPoints?roundPoints + roundPointsCarried:roundPointsCarried}}
    
</template>

<script>
export default {
  /*eslint-disable*/
  name: "App",

  data: () => ({
    dice: [],
    inhand: [],
    totalScore: 0,
    rollAgain: false,
    player: 1,
    roundPointsCarried: 0,
    rollDiceFlag: true,
    player1: {
      totalScore: 0
    },
    player2: {
      totalScore: 0
    }
  }),
  computed: {
    roundPoints: function() {
      let minus = 0;
      let sortedDice = this.inhand.sort();
      if (this.inhand.length > 0) {
        // the only two that are unique for full hands are 6 of a kind and 6 in a row
        if (sortedDice.length === 6) {
          if (sortedDice[0] === sortedDice[5]) {
            this.rollAgain = true;
            return 3000;
          }
          let inorder = true;
          for (let i = 0; i < sortedDice.length - 2; i++) {
            if (sortedDice[i] < sortedDice[i + 1]) {
              inorder = true;
            } else {
              inorder = false;
              break;
            }
          }
          if (inorder) {
            this.rollAgain = true;
            return 1500;
          }
          let pairsCount = 0;
          let lastPair = 0;
          for (let i = 0; i < sortedDice.length - 1; i += 2) {
            if (
              sortedDice[i] === sortedDice[i + 1] &&
              sortedDice[i] !== lastPair
            ) {
              pairsCount++;
              lastPair = sortedDice[i];
            }
          }
          if (pairsCount === 3) {
            this.rollAgain = true;
            return 1500;
          }
        }

        let matrixOfDice = sortedDice.reduce((acc, val) => {
          if (val in acc) {
            acc[val]++;
          } else {
            acc[val] = 1;
          }
          return acc;
        }, {});
        let points = 0;
        let arrayCount = Object.entries(matrixOfDice).sort((a, b) => {
          if (a[1] === b[1]) {
            return 0;
          } else {
            return a[1] > b[1] ? -1 : 1;
          }
        });
        let filterNum = 0;
        if (arrayCount[0][1] === 5) {
          points += 2000;
          filterNum = arrayCount[0][0];
        }
        if (arrayCount[0][1] === 4) {
          minus = 4;
          points += 1000;
          filterNum = arrayCount[0][0];
        }
        console.log(arrayCount);
        if (arrayCount[0][1] === 3) {
          minus = 3;
          points += arrayCount[0][0] != "1" ? arrayCount[0][0] * 100 : 300;

          filterNum = arrayCount[0][0];
        }

        if (arrayCount.length > 1) {
          if (arrayCount[1][1] === 3) {
            points += arrayCount[1][0] != "1" ? arrayCount[1][0] * 100 : 300;
            this.rollAgain = true;
            return points;
          }
        }

        let x = sortedDice.filter(die => {
          return die != filterNum;
        });
        let count = 0;
        for (let j = 0; j < x.length; j++) {
          if (x[j] === 1) {
            count++;
            points += 100;
          }
          if (x[j] === 5) {
            count++;
            points += 50;
          }
        }
        // console.log(minus);
        // console.log(count);
        // console.log(this.dice.length);
        if (minus + count === this.dice.length + this.inhand.length) {
          this.rollAgain = true;
        }

        return points;
      }
    }
  },
  methods: {
    pickup: function(die, i) {
      let newDice = [...this.dice];
      newDice.splice(i, 1);
      this.dice = newDice;

      this.inhand.push(die);
    },
    rollDice: function() {
      //gotoplayer2
      if (!this.rollAgain) {
        // this.player = this.player == 1 ? 2 : 1;
      } else {
        this.roundPointsCarried = this.roundPoints;
      }
      this.rollAgain = false;
      let newDice = [];
      this.inhand = [];
      for (let i = 0; i < 6; i++) {
        newDice.push(Math.floor(Math.random() * 6) + 1);
      }

      this.dice = newDice;
      this.rollDiceFlag = false;
      console.log(this.isGameOver());
      if (this.isGameOver()) {
      }
    },
    bankDice: function() {
      this.roundPointsCarried += this.roundPoints;
      this.inhand = [];
      let diceToRoll = this.rollAgain ? 6 : this.dice.length;
      let newDice = [];
      for (let i = 0; i < diceToRoll; i++) {
        newDice.push(Math.floor(Math.random() * 6) + 1);
      }
      this.rollAgain = false;
      this.dice = newDice;
      console.log(this.isGameOver());
      if (this.isGameOver()) {
        this.player = this.player == 1 ? 2 : 1;
        this.roundPointsCarried = 0;
        // this.rollDice();
        // this.dice = [];
        this.rollDiceFlag = true;
      }
    },
    finishTurn: function() {
      this["player" + this.player].totalScore +=
        this.roundPoints + this.roundPointsCarried;
      this.roundPointsCarried = 0;
      this.dice = [];
      this.inhand = [];
      this.player = this.player == 1 ? 2 : 1;
      this.rollDiceFlag = true;
    },
    isGameOver: function() {
      // if there is a 1 or 5 no game over
      if (this.dice.findIndex(die => die === 1 || die === 5) == -1) {
        return true;
      } else {
        return false;
      }

      let sortedDice = this.dice.sort();

      console.log(sortedDice);

      //3 of a kind
      for (let i = 0; i < sortedDice.length - 2; i++) {
        if (
          sortedDice[i] === sortedDice[i + 1] &&
          sortedDice[i] === sortedDice[i + 2]
        ) {
          return false;
        }
      }

      //12345
      if (sortedDice.length === 6) {
        let inorder = true;
        for (let i = 0; i < sortedDice.length - 2; i++) {
          if (sortedDice[i] < sortedDice[i + 1]) {
            inorder = true;
          } else {
            inorder = false;
            break;
          }
        }
        if (inorder) {
          return false;
        }
      }

      //3 pairs
      let pairsCount = 0;
      let lastPair = 0;
      for (let i = 0; i < sortedDice.length - 1; i += 2) {
        if (sortedDice[i] === sortedDice[i + 1] && sortedDice[i] !== lastPair) {
          pairsCount++;
          lastPair = sortedDice[i];
        }
        return pairsCount !== 3;
      }
    },
    remove: function(die, i) {
      this.dice.push(die);
      this.inhand.splice(i, 1);
    },
    test: function() {
      let sortedDice = this.inhand.sort();
      // the only two that are unique for full hands are 6 of a kind and 6 in a row
      if (sortedDice.length === 6) {
        if (sortedDice[0] === sortedDice[5]) {
          return 3000;
        }
        let inorder = true;
        for (let i = 0; i < sortedDice.length - 2; i++) {
          if (sortedDice[i] < sortedDice[i + 1]) {
            inorder = true;
          } else {
            inorder = false;
            break;
          }
        }
        if (inorder) {
          return 1500;
        }
        let pairsCount = 0;
        let lastPair = 0;
        for (let i = 0; i < sortedDice.length - 1; i += 2) {
          if (
            sortedDice[i] === sortedDice[i + 1] &&
            sortedDice[i] !== lastPair
          ) {
            pairsCount++;
            lastPair = sortedDice[i];
          }
        }
        if (pairsCount === 3) {
          return 1500;
        }
      }

      let matrixOfDice = sortedDice.reduce((acc, val) => {
        if (val in acc) {
          acc[val]++;
        } else {
          acc[val] = 1;
        }
        return acc;
      }, {});
      let points = 0;
      let arrayCount = Object.entries(matrixOfDice).sort((a, b) => {
        if (a[1] === b[1]) {
          return 0;
        } else {
          return a[1] > b[1] ? -1 : 1;
        }
      });
      let filterNum = 99999;
      if (arrayCount[0][1] === 5) {
        points += 2000;
        filterNum = arrayCount[0][0];
      }
      if (arrayCount[0][1] === 4) {
        points += 1000;
        filterNum = arrayCount[0][0];
      }

      if (arrayCount[0][1] === 3) {
        points += arrayCount[0][0] * 100;
        console.log(arrayCount);
        filterNum = arrayCount[0][0] !== 1 ? arrayCount[0][0] * 100 : 300;
      }

      if (arrayCount.length > 1) {
        if (arrayCount[1][1] === 3) {
          points += arrayCount[1][0] !== 1 ? arrayCount[0][0] * 100 : 300;
          return points;
        }
      }
      console.log(filterNum);
      let x = sortedDice.filter(die => {
        return die != filterNum;
      });
      console.log(x);
      for (let j = 0; j < x.length; j++) {
        console.log("lko");
        if (x[j] === 1) {
          points += 100;
        }
        if (x[j]) {
          points += 50;
        }
      }

      return points;
    }
  }
};
</script>

<style scoped>
.die {
  width: 40px;
  height: 40px;
  background: red;
  color: white;
  text-align: center;
  line-height: 40px;
  border-radius: 2px;
}
</style>
