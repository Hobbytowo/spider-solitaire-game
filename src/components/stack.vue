<template lang="html">
  <div class="stack">
    <card-component
      v-for="(card, index) in stack"
      :card="card"
      :key="card.id"
      :style="{ top: -index * topStackIdx + 'px' }"
      @click.native="onCardClick(card)"
    />
  </div>
</template>

<script>
import cardComponent from '@/components/card'
export default {
  components: {
    cardComponent
  },
  props: {
    stack: { type: Array, required: true }
  },
  data () {
    return {
      topStackIdx: 125,
      topStackIdxLg: 125,
      topStackIdxMd: 97,
      topStackIdxSm: 70
    }
  },
  created () {
    this.onResize()
    window.addEventListener('resize', this.onResize)
  },

  methods: {
    onCardClick (card) {
      this.$emit('onCardSelect', card)
    },

    onResize () {
      let width = window.innerWidth

      if (width > 1155) {
        if (this.topStackIdx !== this.topStackIdxLg) {
          this.topStackIdx = this.topStackIdxLg
        }
        return
      }

      if (width <= 889) {
        if (this.topStackIdx !== this.topStackIdxSm) {
          this.topStackIdx = this.topStackIdxSm
        }
        return
      }

      if (width <= 1155) {
        if (this.topStackIdx !== this.topStackIdxMd) {
          this.topStackIdx = this.topStackIdxMd
        }
        return
      }
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
