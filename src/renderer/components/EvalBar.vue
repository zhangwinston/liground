<template>
  <div
    class="progress"
    :class="{ flip : $store.getters.orientation == 'black' }"
  >
    <div
      class="progress-value"
      :style="{ height: filterSectionHeight }"
    />
  </div>
</template>

<script>
export default {
  name: 'EvalBar',
  computed: {
    filterSectionHeight () {
      return `${Math.min(100, Math.max(0, 100 - this.$store.getters.cpforWhitePerc * 100))}%`
    }
  }
}
</script>

<style scoped>

.progress {
  position: relative;
  width: 17px;
  padding: 0 0px;
  display: flex;
  justify-content: flex-start;
  background: #fff;
  border: 1px solid #222;
  border-radius: 5px 5px 5px 5px;
  overflow: hidden;
}
.flip {
  transform: scaleY(-1);
}
.progress-value {
  width: 100%;
  height: 50%;
  background: #000;
  transition: height .25s;
}
</style>
