<template>
  <div class="main">
    <div class="left-container">
      <div class="game-container">
        <pgn-browser class="pgn-browser" />
        <div class="board-container">
          <div class="board-top">
            <div
              class="chessground"
              @mousewheel.prevent="scroll($event)"
            >
              <ChessGround
                :orientation="orientation"
                @onMove="showInfo"
              />
            </div>
            <EvalBar class="eval-bar" />
          </div>
          <div class="fen">
            FEN <input
              class="input"
              type="text"
              placeholder="fen position"
              :value="fen"
              size="60"
              @change="checkValidFEN"
            >
            <PieceStyleSelector class="piece-style" />
          </div>
        </div>
      </div>
      <EvalPlot class="eval-plot" />
    </div>
    <AnalysisView
      class="analysis-view"
      :reset="resetAnalysis"
      @move-to-start="moveToStart"
      @move-to-end="moveToEnd"
      @move-back-one="moveBackOne"
      @move-forward-one="moveForwardOne"
      @flip-board="flipBoard"
    />
  </div>
</template>

<script>
import AnalysisView from './AnalysisView'
import EvalBar from './EvalBar'
import ChessGround from './ChessGround'
import EvalPlot from './EvalPlot'
import PieceStyleSelector from './PieceStyleSelector'
import Vue from 'vue'
import Module from 'ffish-es6'
import PgnBrowser from './PgnBrowser.vue'
// TODO: use GameInfo component?
// import GameInfo from './GameInfo.vue'

let ffish = null

export default {
  name: 'GameBoards',
  components: {
    AnalysisView,
    EvalBar,
    ChessGround,
    PieceStyleSelector,
    EvalPlot,
    // GameInfo,
    PgnBrowser
  },
  data () {
    return {
      positionInfo: '',
      game: null,
      resetAnalysis: false
    }
  },
  computed: {
    variant () {
      return this.$store.getters.variant
    },
    orientation () {
      return this.$store.getters.orientation
    },
    moves () {
      return this.$store.getters.moves
    },
    fen () {
      return this.$store.getters.fen
    },
    currentMove () { // this returns the current half-move or -1 at the start of the game
      const fen = this.$store.getters.fen
      for (const move of this.moves) {
        if (move.fen === fen) {
          return move.ply - 1
        }
      }
      return -1
    }
  },
  beforeCreate () {
    console.log('beforeCreate()')
    new Module().then(loadedModule => {
      ffish = loadedModule
      console.log(`initialized ${ffish} ${loadedModule}`)
      const game = new ffish.Board()
      console.log(game.fen())
    })
  },
  mounted () { // EventListener für Keyboardinput, ruft direkt die jeweilige Methode auf
    window.addEventListener('keydown', (event) => {
      const keyName = event.key
      if (keyName === 'ArrowUp') {
        this.moveToStart()
      }
      if (keyName === 'ArrowDown') {
        this.moveToEnd()
      }
      if (keyName === 'ArrowLeft') {
        this.moveBackOne()
      }
      if (keyName === 'ArrowRight') {
        this.moveForwardOne()
      }
    }, false)
  },
  methods: {
    scroll (event) { // also moves back and forth when being slightly next to the board and for example over the pockets
      if (event.deltaY < 0) {
        this.moveBackOne()
      } else {
        this.moveForwardOne()
      }
    },
    moveToStart () { // this method returns to the starting point of the current line
      const board = new ffish.Board(this.variant)
      const startFen = board.fen()
      this.$store.dispatch('fen', startFen)
    },
    moveToEnd () { // this method moves to the last move of the current line
      if (this.currentMove >= this.moves.length - 1) {
        return
      }
      this.$store.dispatch('fen', this.moves[this.moves.length - 1].fen)
    },
    moveBackOne () { // this method moves back one move in the current line
      const num = this.currentMove
      if (num === -1) {
        return
      }
      if (num === 0) {
        const board = new ffish.Board(this.variant)
        const startFen = board.fen()
        this.$store.dispatch('fen', startFen)
        return
      }
      this.$store.dispatch('fen', this.moves[num - 1].fen)
    },
    moveForwardOne () { // this method moves forward one move in the current line
      const num = this.currentMove
      if (num >= this.moves.length - 1) {
        return
      }
      if (num === -1) {
        this.$store.dispatch('fen', this.moves[0].fen)
        return
      }
      if (num === 0) {
        this.$store.dispatch('fen', this.moves[1].fen)
        return
      }
      this.$store.dispatch('fen', this.moves[num + 1].fen)
    },
    flipBoard () {
      if (this.variant === 'racingkings') {
        return
      }
      if (this.orientation === 'white') {
        this.$store.dispatch('orientation', 'black')
      } else {
        this.$store.dispatch('orientation', 'white')
      }
    },
    selectPocketPiece (piece) {
      this.$store.commit('selectPocketPiece', ['boardA', piece.type])
    },
    deselectPocketPieces () {
      this.$store.commit('selectPocketPiece', ['boardA', ''])
    },
    getBoardPos (event) {
      if (event.explicitOriginalTarget.className === 'cg-board' && this.selectedPockedPiece.boardA !== '') {
        // get click field
        const x = Math.floor(event.layerX / 40)
        const y = Math.floor(event.layerY / 40)
        // var stringPos = y * 9 + x

        const letters = { 0: 'a', 1: 'b', 2: 'c', 3: 'd', 4: 'e', 5: 'f', 6: 'g', 7: 'h' }
        let pieceCode = Vue.methds.pieceTypeToShort(this.selectedPockedPiece.boardA)

        pieceCode = { type: pieceCode, color: this.turnColor.charAt(0) }
        this.$store.dispatch('insertPieceAtPosition', ['boardA', pieceCode, letters[x] + (8 - y)])
      } else {
        this.deselectPocketPieces()
      }
    },
    showInfo (event) {
      console.log(`showInfo: ${this.fen}`)
      // this.$store.dispatch('fen', event['fen'])
      console.log(`fen: ${this.$store.getters.fen}`)
      // const newMove = event.history[event.history.length - 1]
      console.log(`event.history: ${event.history}`)

      if (this.$store.getters.active) {
        this.$store.dispatch('stopEngine')
        this.$store.dispatch('position')
        this.$store.dispatch('goEngine')
      }
    },
    drawArrow (event) {
      console.log(`event: ${event}`)
    },
    checkValidFEN (event) {
      if (ffish.validateFen(event.target.value, this.variant) === 1) {
        this.$store.dispatch('fen', event.target.value)
      } else {
        console.log(`invalid fen: ${event.target.value}`)
      }
      this.resetAnalysis = !this.resetAnalysis
    }

  }
}
</script>

<style scoped>
.main {
  min-width: 0;
  min-height: 0;
  margin: 0 20px 20px 20px;
  display: flex;
  flex-direction: row;
  justify-content: center;
  flex: 1 1 auto;
}
.left-container {
  display: flex;
  flex-direction: column;
  flex: 0 1 50%;
}
.analysis-view {
  min-width: 0;
  margin-left: 15px;
  display: flex;
  flex-direction: column;
  flex: 0 1 50%;
}
.game-container {
  min-height: 0;
  display: flex;
  flex-direction: row;
  flex: 1 1 auto;
}
.pgn-browser {
  border: 1px solid black;
  border-radius: 4px;
  flex-grow: 0;
}
.board-container {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}
.board-container .board-top {
  display: flex;
  flex-direction: row;
  align-items: stretch;
  justify-content: stretch;
}
.chessground {
  flex-grow: 1;
}
.eval-bar {
  margin: 10px;
  flex-grow: 0;
}
.fen {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.fen .input {
  font-size: 12pt;
  max-width: 60vw;
}
.piece-style {
  margin-top: 10px;
  width: 300px;
}

</style>
