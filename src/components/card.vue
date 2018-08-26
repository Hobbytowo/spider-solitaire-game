<template lang="html">
  <div class="flipper-wrap" :class="{ 'flipper-wrap--reversed': card.reversed }">
    <div class="flipper">

      <div
        class="card card__front"
        :class="{ 'card--selected': card.selected, 'card--empty': card.empty }">
        <span class="name name--up">
          {{ name }}
          <img v-if="!card.empty" class="symbol symbol--small" :src="`/images/${ color }.svg`">
        </span>
        <img v-if="!card.empty" class="symbol" :src="`/images/${ color }.svg`">
        <span class="name name--down">
          {{ name }}
          <img v-if="!card.empty" class="symbol symbol--small" :src="`/images/${ color }.svg`">
        </span>
      </div>

      <div class="card card__back">
      </div>

    </div>
  </div>
</template>

<script>
export default {
  props: {
    card: { type: Object, required: true }
  },
  computed: {
    name () {
      const names = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
      return names[this.card.value]
    },
    color () {
      return this.card.color
    }
  }
}
</script>

<style lang="scss" scoped>
  .flipper {
    transition: 0.6s;
    transform-style: preserve-3d;
    position: relative;
    width: 100px;
    height: 150px;
    margin: 5px;
  }

  .flipper-wrap {
    position: relative;
    perspective: 1000;

    &--reversed .flipper {
      transform: rotateY(180deg);
      margin: -13px 5px;
    }
  }

  .card {
    backface-visibility: hidden;
    position: absolute;
    top: 0;
    left: 0;

    width: 100px;
    height: 150px;
    border: 2px solid grey;
    border-radius: 10px;
    vertical-align: center;
    font-size: 20px;

    &__front {
      background: white;
      display: flex;
      justify-content: center;
      align-items: center;
      cursor: pointer;
    }

    &__back {
      transform: rotateY(180deg);
      background: repeating-linear-gradient(
        -55deg,
        #222,
        #222 10px,
        #333 10px,
        #333 20px
      );
    }

    &--selected {
      border-color: #b70000;
    }

    &--empty{
      background-color: #eee;
    }
  }
  .symbol {
    width: 55px;
    &--small{
      width: 14px;
      position: relative;
      top: 2px;
    }
  }
  .name {
    position: absolute;
    &--up {
      top: 3px;
      left: 5px;
    }
    &--down {
      bottom: 3px;
      right: 5px;
      transform: rotate(180deg);
    }
  }
</style>
