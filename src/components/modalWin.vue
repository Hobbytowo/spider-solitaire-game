<template lang="html" type="text/x-template" id="modal-template">
  <transition name="modal">
    <div class="modal__mask">
      <div class="modal__wrapper">
        <div class="modal__container">

          <div class="modal__header">
            <h3 class="modal__title">
              Congratulations!
            </h3>
            <button
              type="button"
              class="modal__button button button--exit"
              @click="$emit('close')">
                x
            </button>
          </div>

          <div class="modal__body">
            <slot name="body">
              You have won this game!
            </slot>
          </div>

          <div class="modal__footer">
            <slot name="footer">
              <button
                @click="$emit('close'); $emit('newGame')"
                class="modal__button button button--newGame"
                type="button">
                New Game
              </button>
            </slot>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: "modalWin-component"
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
      width: 400px;
      margin: 0px auto;
      padding: 20px 30px;
      background-color: #fff;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
      transition: all .3s ease;
      font-family: Helvetica, Arial, sans-serif;
    }

    &__header{
      display: flex;
      align-items: center;
    }

    &__title {
      margin-top: 0;
      flex: 1;
    }

    &__body {
      margin: 50px 0;
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

    &--newGame{
      background-color: #79ed6f;
      border: 2px solid #03a500;
      padding: 10px 22px;
      &:hover{
        background-color: #03a500;
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
