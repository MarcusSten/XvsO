<template>
  <div id="app">
    <div class="container" :style="{background: 'linear-gradient(90deg, rgb(46 0 0) ' + percentBackgroundX + '%, rgb(0 0 0)' + percentBackgroundCenter + '%, rgb(0 55 3) ' + percentBackgroundO + '%)'}">
      <!-- <div class="logo"><span class="x-word">X</span> vs <span class="o-word">O</span></div> -->
      <div class="logo"><img src="./assets/logoOS.png" alt=""></div>
      <div>Welcome to Naughts and Crosses</div>
      <div class="counter-wrapper">
        <div>
          <p><span class="x-word">X</span> Player</p>
          <p>{{ counterFirstPlayer }}</p>
        </div>
        <p>VS</p>
        <div>
          <p><span class="o-word">O</span> Player</p>
          <p>{{ counterSecondPlayer }}</p>
        </div>
      </div>
      <div class="wrapper-box">
      <div v-for="(item, index) in items" :key="item.id" :class="{ 'disabled': item.isClicked }">
        <div class="box" @click="submitBox(item, index)">
          <div :class="{'landingAnimation': item.animation}">
            {{ item.name }}
          </div>
        </div>
      </div>
      <div class="victory-line" :class="`line${lineTr}`"></div>
      </div>
      <div class="message">
        {{ message }}
      </div>
      <div>
        <p>{{ countGame }} <span v-if="countGame === 1">game</span><span v-else>games</span> played</p>
      </div>
      <div class="button-wrapper">
          <div @click="resumeGame()" class="resume-btn" v-if="counterFirstPlayer !== 0 || counterSecondPlayer !== 0 || count === 9">
          Resume Game
        </div>
        <div @click="resetCount()" class="resume-btn" v-if="counterFirstPlayer !== 0 || counterSecondPlayer !== 0">
          Count Reset
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      currentPlayer: 'playerX',
      playerX: [],
      playerO: [],
      countGame: 0,
      percentBackgroundX: 5,
      percentBackgroundCenter: 50,
      percentBackgroundO: 95,
      counterFirstPlayer: 0,
      counterSecondPlayer: 0,
      clickedIndex: null,
      message: 'Player turn: X',
      isDisabled: false,
      lineTr: '',
      isVictoryFirstPlayer: false,
      isVictorySecondPlayer: false,
      count: 0,
      items: [
        { id: 1, name: '', isClicked: false, animation: false},
        { id: 2, name: '', isClicked: false, animation: false},
        { id: 3, name: '', isClicked: false, animation: false},
        { id: 4, name: '', isClicked: false, animation: false},
        { id: 5, name: '', isClicked: false, animation: false},
        { id: 6, name: '', isClicked: false, animation: false},
        { id: 7, name: '', isClicked: false, animation: false},
        { id: 8, name: '', isClicked: false, animation: false},
        { id: 9, name: '', isClicked: false, animation: false},
      ],
      victoryRules: [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9],
        [3, 5, 7],
        [1, 5, 9],
      ],
    };
  },
  methods: {
    submitBox(el, index) {
      if (!el.isClicked && !this.isDisabled) {
        el.isClicked = true;
        this.clickedIndex = index;
        if (this.currentPlayer === 'playerX') {
          el.name = 'X'
        } else {
          el.name = 'O'
        }
        this[this.currentPlayer].push(el.id)

        this.items.forEach((item, i) => {
          if (i === index || item.isClicked) {
            item.animation = true
          }
        })
        this.switchPlayer()
        this.checkForVictory()
      }
    },
    switchPlayer() {
      this.currentPlayer = (this.currentPlayer === 'playerX') ? 'playerO' : 'playerX'
      if (!this.isVictoryFirstPlayer || !this.isVictorySecondPlayer) {
        this.message = 'Player turn: ' + this.currentPlayer.replace('player', '')
      }
      if (this.count === 8 && !this.checkForVictory) {
        this.isDisabled = true
        this.message = 'Standoff'
      } else if (this.count === 8 && this.checkForVictory) {
        this.countGame ++
      } 
      this.count ++
      console.log(this.count)
    },
    checkForVictory() {
      this.isVictoryFirstPlayer = this.victoryRules.some((rule) =>
        rule.every((position) => this.playerX.includes(position))
      );
      this.isVictorySecondPlayer = this.victoryRules.some((rule) =>
        rule.every((position) => this.playerO.includes(position))
      );

      if (this.isVictoryFirstPlayer) {
        this.message = 'Player X wins!'
        this.isDisabled = true
        this.counterFirstPlayer ++
        this.percentBackgroundX += 5
        this.percentBackgroundCenter += 5
        this.percentBackgroundO += 5
        this.countGame ++
        this.victoryLine()
        return true
      } else if (this.isVictorySecondPlayer) {
        this.message = 'Player O wins!'
        this.isDisabled = true
        this.counterSecondPlayer ++
        this.percentBackgroundO -= 5
        this.percentBackgroundCenter -= 5
        this.percentBackgroundX -= 5
        this.countGame ++
        this.victoryLine()
        return true
      } else {
        return false
      }    
    },
    victoryLine() {
      let winingPositions = this.isVictoryFirstPlayer ? this.playerX : this.playerO;
      for (let rule of this.victoryRules) {
        if (rule.every(position => winingPositions.includes(position))) {
          this.lineTr = rule.join('');
        }
      }
    },
    resumeGame() {
      if (this.countGame % 2 === 0) {
        this.currentPlayer = 'playerX'
      } else {
        this.currentPlayer = 'playerO'
      }
      this.playerX = []
      this.playerO = []
      this.message = 'Player turn: ' + this.currentPlayer.replace('player', '')
      this.isDisabled = false
      this.lineTr = ''
      this.isVictoryFirstPlayer = false
      this.isVictorySecondPlayer = false
      this.count = 0

      this.$nextTick(() => {
        this.resetAnimation();
      });

      this.items.forEach(item => {
        item.name = ''
        item.isClicked = false
      });

      this.hideVictoryLines()
    },
    resetAnimation() {
      this.items.forEach((item) => {
        item.animation = false
      })
      console.log(this.items)
    },
    resetCount () {
      this.counterFirstPlayer = 0
      this.counterSecondPlayer = 0
      this.countGame = 0
      this.percentBackgroundX = 5,
      this.percentBackgroundCenter = 50,
      this.percentBackgroundO = 95,
      this.resumeGame() 
    },
    hideVictoryLines() {
      for (let i = 1; i <= 9; i++) {
        if (this.$refs['line' + i] && this.$refs['line' + i][0]) {
          this.$refs['line' + i][0].style.display = 'none';
        }
      }
    }
  }
}
</script>

<style>
html {
  height: 100%;
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  justify-content: center;
  margin: 0;
  height: 100%;
  width: 100%;
}

#app {
  width: 100%;
  height: 100%;
}

h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #d0d0d0;
  height: 100%;
}

.logo {
  margin: 30px;
}

.logo img {
  width: 100%;
}

.x-word {
  color: red;
  font-weight: bold;
}

.o-word {
  color: green;
  font-weight: bold;
}

.counter-wrapper {
  display: flex;
  justify-content: space-between;
  color: #d0d0d0;
  font-size: 20px;
  width: 300px;
  margin: 0 auto;
}

.button-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.wrapper-box {
  display: flex;
  justify-content: center;
  width: 300px;
  flex-wrap: wrap;
  border: 1px solid #d0d0d0;
  position: relative;
}

.box {
  width: 98px;
  height: 98px;
  border: 1px solid #d0d0d0;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  font-size: 30px;
}

.disabled {
  pointer-events: none; 
  opacity: 0.7; 
}

.victory-line {
  background-color: #d0d0d0;
  height: 3px;
  width: 275px;
  display: none;
  position: absolute;
}

.message {
  margin-top: 30px;
  color: #d0d0d0;
  font-size: 20px;
}

.resume-btn {
  border: 1px solid #d0d0d0;
  padding: 10px;
  border-radius: 5px;
  margin-top: 30px;
  cursor: pointer;
  font-weight: bold;
  margin: 0 20px;
  transition: filter 0.3s;
}

.resume-btn:is(:hover, :focus-visible) {
  filter: brightness(115%)
}

.resume-btn:active {
  filter: brightness(85%)
}

.landingAnimation {
  transition-duration: 2s;
  transform: scale(1.5);
  font-weight: bold;
}

.backgroundAnimation {
  transition: background 2s ease-in-out;
}

.line123 {
  display: block;
  top: 47px;
}

.line456 {
  display: block;
  top: 147px;
}

.line789 {
  display: block;
  top: 247px;
}

.line159 {
  display: block;
  transform: rotate(45deg);
  top: 147px;
  width: 355px;
}

.line357 {
  display: block;
  transform: rotate(135deg);
  top: 147px;
  width: 355px;
}

.line147 {
  display: block;
  transform: rotate(90deg);
  left: -87px;
  top: 147px;
}

.line258 {
  display: block;
  transform: rotate(90deg);
  left: 13px;
  top: 147px;
}

.line369 {
  display: block;
  transform: rotate(90deg);
  left: 113px;
  top: 147px;
}

@media screen and (max-height: 700px) {
  .container {
    height: auto;
  }
}
</style>