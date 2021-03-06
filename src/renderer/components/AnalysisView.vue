<template>
  <div class="analysis">
    <AnalysisHead />
    <AnalysisEvalRow id="game_clock" />
    <EngineStats />
    <div
      v-if="active"
      class="processing-bar"
    />
    <PVLines class="panel" />
    <div class="game-window panel noselect">
      <div
        v-for="move in moves"
        :key="move.ply"
      >
        <div class="move-field">
          <div
            v-if="move.ply % 2 == 1"
            class="float-left-child move-number"
          >
            {{ (move.ply+1) / 2 }}.
          </div>
          <div
            class="float-left-child move-name"
            :class="{ active : move.fen != $store.getters.lastFen && move.fen == $store.getters.fen}"
            @click="updateBoard(move)"
          >
            {{ move.name }}
          </div>
        </div>
      </div>
    </div>
    <JumpButtons
      @flip-board="$emit('flip-board', 0)"
      @move-to-start="$emit('move-to-start',0)"
      @move-back-one="$emit('move-back-one',0)"
      @move-forward-one="$emit('move-forward-one',0)"
      @move-to-end="$emit('move-to-end',0)"
    />
    <game-info id="gameinfo" />
  </div>
</template>

<script>
import AnalysisHead from './AnalysisHead'
import AnalysisEvalRow from './AnalysisEvalRow'
import JumpButtons from './JumpButtons'
import EngineStats from './EngineStats'
import PVLines from './PVLines'
import GameInfo from './GameInfo.vue'

export default {
  name: 'AnalysisView',
  components: {
    AnalysisHead, AnalysisEvalRow, JumpButtons, EngineStats, PVLines, GameInfo
  },
  props: {
    reset: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      cmd: ''
    }
  },
  computed: {
    active () {
      return this.$store.getters.active
    },
    stdIO () {
      return this.$store.getters.stdIO
    },
    message () {
      return this.$store.getters.message
    },
    counter () {
      return this.$store.getters.counter
    },
    multipv () {
      return this.$store.getters.multipv
    },
    pv () {
      return this.$store.getters.pv
    },
    cp () {
      return this.$store.getters.cpForWhiteStr
    },
    pv2 () {
      return this.$store.getters.pv2
    },
    cp2 () {
      return this.$store.getters.cpForWhiteStr
    },
    moves () {
      return this.$store.getters.moves
    }
  },
  watch: {
    reset: function () {
      this.$store.commit('resetMultiPV')
    }
  },
  methods: {
    updateBoard (move) {
      this.$store.dispatch('fen', move.fen)
    },
    onKeyup (event) {
      if (event.key === 'Enter') {
        this.$store.dispatch('sendEngineCommand', this.cmd)
        this.cmd = ''
      }
    }
  }
}
</script>

<style scoped>
input {
  font-size: 11pt;
}
#game_clock {
  margin-top: -10px;
  margin-bottom: -20px;
}
.game-window {
  height: 20%;
  overflow-y: scroll;
}
.panel {
  border-radius: 3px 3px 3px 3px;
  border: 1px solid #888;
  font-family: sans-serif;
  font-weight: 200;
}
.panel + .panel {
  margin-top: 7px;
}
.console-log {
  border-radius: 3px 3px 3px 3px;
  border-color: #888;
  border-width: 1px;
  border-style: solid;
  font-family: monospace;
  font-weight: 100;
  font-size: 8pt;
  height: 20%;
  overflow-y: scroll;
  white-space: nowrap;
  text-align: left;
}
p {
  margin: 0px;
}
.multipv-line:hover {
  background-color: #d3e1eb;
  cursor: pointer;
}
.multipv-line {
  border-width: 1px;
  border-color: #333;
  border-top: 0;
  border-right: 0;
  border-left: 0;
  border-style: solid;
  text-align: left;
}
.multipv-eval {
  padding: 5px;
  font-weight: 1000;
  font-family: sans-serif;
  width: 70px;
  text-align: center;
}
.move-field {
  color: #333;
  color: black;
  background-color: #111;
  text-align: center;
  font-size: 12pt;
  font-family: 'NOTO CHESS', sans-serif;
  padding: 0px 0px 0px 0px;
  border-color: #222;
  border-width: 1px;
  margin-right: 5px;
  pointer-events: none;
  min-width: 20px;
  white-space: nowrap;
  main-max-width: auto;
  gauge-gap: 17px;
}
.move-field:hover .move-name {
  background-color: #2196F3;
  cursor: pointer;
  border-radius: 4px 4px 4px 4px;
  color: #fff;
}
.move-field:hover .active {
  background-color: #2196F3;
  cursor: pointer;
  border-radius: 4px 4px 4px 4px;
  color: #fff;
}
.move-field .active{
  background-color:#444;
  border-radius: 4px 4px 4px 4px;
  color: #fff;
}
.grid-parent {
  align-items: center;
}
.move-number {
  color: #777;
}
.move-name {
  margin-right: 4px;
  pointer-events: auto;
}
.processing-bar {
  height: 5px;
  background-color: #6ca040;
  background-color: #2196F3;
  background-image: url('../assets/images/analysis/bar-highlight.png');
  animation: bar-anim 1000s linear infinite;
  transition: width 1s;
}
@keyframes bar-anim {
  0% {
      background-position: 0 0;
  }
  100% {
      background-position: 100000px 0;
  }
}
#gameinfo {
  height: auto;
  margin: 1em 0em;
  border: 1px solid black;
  border-radius: 5px;
}

</style>
