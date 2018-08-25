<template lang="html">
  <div class="stack">
    <card-component
      v-for="(card, index) in stack"
      :card="card"
      :key="card.id"
      :style="{ top: -index * 125 + 'px' }"
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
  methods: {
    onCardClick (card) {
      this.$emit('onCardSelect', card)
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
