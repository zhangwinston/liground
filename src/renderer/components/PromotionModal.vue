<template>
  <div
    class="prom-container"
  >
    <div
      v-for="(piece, index) in promOptions"
      :key="piece.type"
      :class="[piece.type, side, 'pieceoption', idtocss[index]]"
      @click="close(piece.type)"
    />
  </div>
</template>

<script>
export default {
  name: 'PromotionModal',
  props: {
    promOptions: {
      type: Array,
      default: () => ([
        { type: 'king' },
        { type: 'queen' },
        { type: 'rook' },
        { type: 'bishop' },
        { type: 'knight' }
      ])
    }
  },
  data () {
    return {
      promDir: { queen: 'q', rook: 'r', bishop: 'b', knight: 'n', pawn: 'p', king: 'k' },
      shogiPromDir: { pawn: '=', lance: '=', knight: '=', bishop: '=', silver: '=', rook: '=', ppawn: '+', plance: '+', pknight: '+', pbishop: '+', psilver: '+', prook: '+' },
      idtocss: {
        0: 'one',
        1: 'two',
        2: 'three',
        3: 'four',
        4: 'five'
      }
    }
  },
  computed: {
    side () {
      return this.$store.getters.turn ? 'white' : 'black'
    }
  },

  methods: {
    close (value) {
      if (this.$store.getters.variant === 'shogi') {
        this.$emit('close', this.shogiPromDir[value])
      } else {
        this.$emit('close', this.promDir[value])
      }
    }
  }
}

</script>

<style scoped>
   .pieceoption {
    box-shadow: 0 0 10px black;
    background-size: contain;
    background-repeat: no-repeat;
    width: 100%;
    height: 100%;
    background-color: white;
  }
  .pieceoption:hover {
    background-color: rgb(182, 155, 107);
  }

  .prom-container{

    display: grid;
    grid-template-rows: repeat(5, 20%);
    height: 100%;
  }
  .prom-container:hover {
    cursor: pointer;
  }
  .one{
    grid-row-start: 1
  }
  .two{
    grid-row-start: 2
  }
  .three{
    grid-row-start: 3
  }
  .four{
    grid-row-start: 4
    }
  .five{
    grid-row-start: 5
     }
</style>
