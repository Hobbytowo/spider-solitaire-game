<template lang="html" type="text/x-template" id="modal-template">
  <transition name="modal">
    <div class="modal__mask">
      <div class="modal__wrapper">
        <div class="modal__container">

          <div class="modal__header">
            <h3 class="modal__title">
              How to play?
            </h3>
            <button
              type="button"
              class="modal__button button button--exit"
              @click="$emit('close')">
                x
            </button>
          </div>

          <div class="modal__body body">
            <slot name="body">
              <h4 class="body__title">Object of the game</h4>
              <p class="body__par">
                The goal is to assemble 13 cards of a suit, in ascending sequence from ace through king, on top of a pile. Whenever a full suit of 13 cards is so assembled, it is lifted off and discarded from the game. The game is won if all eight suits are played out.
              </p>
              <h4 class="body__title">The play</h4>
              <p class="body__par">
                The top card of a pile may be moved, together with all face-up cards below it that follow in ascending suit and sequence.
              </p>
              <p class="body__par">
                A sequence of available cards may be broken at any point by leaving some cards behind. Example: If a pile from top down shows 4, 5, 6, 7, either the first one, two, or three cards may be moved as a unit, but the 7 may not be moved until the covering three cards are removed. When all face-up cards on a pile are removed, the next card below is turned face up and becomes available.
              </p>
              <p class="body__par">
                A movable unit of cards may be placed either in a space or on a card of the next-higher rank to the bottom card of the unit, regardless of color or suit. Example: If the bottom card of a unit is the J, it may be moved onto any one of the four queens.
              </p>
              <p class="body__par">
                A king can be moved only onto a space. Alternatively, the spaces may be filled with any movable unit.
              </p>
              <p class="body__par">
                When all possible or desired moves come to a standstill, the player deals another row of ten cards face up. However, before such a deal may be made, all spaces must be filled. The final deal consists of only four cards, which are placed on the first four piles.
              </p>
            </slot>
          </div>

        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: "modalRules-component"
}
</script>

<style lang="scss" scoped>

  .modal{

    &__mask {
      position: fixed;
      z-index: 9998;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, .5);
      display: table;
      transition: opacity .3s ease;
    }

    &__wrapper {
      display: table-cell;
      vertical-align: middle;
    }

    &__container {
      width: 80vw;
      max-height: 98vh;
      margin: 0px auto;
      padding: 20px 30px;
      background-color: #fff;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
      transition: all .3s ease;
      font-family: Helvetica, Arial, sans-serif;
      overflow-y: auto;
    }

    &__header{
      display: flex;
      align-items: center;
      padding-bottom: 20px;
      border-bottom: 2px solid gray;
    }

    &__title {
      margin-top: 0;
      flex: 1;
    }

  }

  .body{
    padding-top: 30px;
    &__par{
      padding: 10px;
    }
  }

  .button{
    padding: 5px 12px;
    border-radius: 5px;
    cursor: pointer;

    &--exit{
      background-color: #b70000;
      color: white;
      border: none;
      &:hover{
        background-color: #333;
      }
    }
  }

  .modal-enter, .modal-leave-active {
    opacity: 0;
  }

  .modal-enter .modal-container,
  .modal-leave-active .modal-container {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }

</style>
