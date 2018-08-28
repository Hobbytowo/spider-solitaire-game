<template>
  <div class="container">

    <img class="img img--left" src="/images/spider.svg" alt="spider icon">
    <img class="img img--right" src="/images/spider.svg" alt="spider icon">

    <header class="header">
      <h1 class="header__title">Spider Solitaire</h1>
      <button class="header__button" @click="showModalRules = true">?</button>
      <modalRules-component v-if="showModalRules" @close="showModalRules = false"/>
    </header>

    <section class="menu">
      <button @click="addCards" class="menu__item  item item--button" type="button">
        Add cards ({{ cards.length / stacks.length }})
      </button>

      <div class="menu__item item item--score">
        Collected full colors: {{ fullColor }} / {{ nrOfFoundation }}
      </div>

      <div class="menu__item item item--level level">
        <label class="menu__label">Number of colors</label>
        <select class="menu__select" v-model="levelSelected">
          <option
          v-for="nr in levelOptions"
          :value="nr"
          class="menu__option"
          :key="nr"
          @click="clearData(); createCards()">
            {{ nr }}
          </option>
        </select>
      </div>
    </section>

    <div class="stacks">
      <stack-component
        v-for="(stack, i) in stacks"
        :stack="stack"
        :key="i"
        @onCardSelect="onCardSelect"
      />
    </div>

    <modalWin-component
      v-if="showModalWin"
      @close="showModalWin = false"
      @newGame="clearData(); createCards()"
    />
  </div>
</template>

<script>
import _ from 'lodash'
import cardComponent from '@/components/card'
import stackComponent from '@/components/stack'
import modalRulesComponent from '@/components/modalRules'
import modalWinComponent from '@/components/modalWin'
import { CLUB, DIAMOND, SPADE, HEART } from '@/cardTypes'

export default {
  components: {
    cardComponent,
    stackComponent,
    modalWinComponent,
    modalRulesComponent
  },

  data () {
    return {
      stacks: Array(10).fill().map(() => []),
      cards: [],
      colors: [CLUB, DIAMOND, SPADE, HEART],
      selectedCards: [],
      levelOptions: [1, 2, 4],
      levelSelected: '1',
      fullColor: 0,
      nrOfFoundation: 8,
      nrOfCardsInColor: 13,
      showModalWin: false,
      showModalRules: false
    }
  },
  created () {
    this.createCards()
  },

  methods: {
    createCards () {
      for (let i = 0; i < this.nrOfCardsInColor * this.nrOfFoundation; i++) {
        this.cards.push({
          id: i,
          value: i % this.nrOfCardsInColor,
          color: this.colors[i % this.levelSelected],
          reversed: true,
          stack: -1,
          selected: false,
          empty: false,
          toString () {
            const names = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
            return `${ names[this.value] } ${ this.color } - Stack: ${ this.stack }`
          }
        })
      }
      this.cards = _.shuffle(this.cards)

      // Create stacks
      for (let stackNumber = 0; stackNumber < 4; stackNumber++) {
        for (let i = 0; i < 6; i++) {
          const card = this.cards.pop()
          card.stack = stackNumber
          this.stacks[stackNumber].push(card)
        }
      }

      for (let stackNumber = 4; stackNumber < this.stacks.length; stackNumber++) {
        for (let i = 0; i < 5; i++) {
          const card = this.cards.pop()
          card.stack = stackNumber
          this.stacks[stackNumber].push(card)
        }
      }

      // Reverse last card in stack
      this.stacks.forEach(stack => {
        const lastIndex = stack.length - 1
        stack[lastIndex].reversed = false
      })
    },

    clearData () {
      this.stacks = Array(10).fill().map(() => [])
      this.cards = []
      this.fullColor = 0
    },

    deselectCards () {
      this.stacks.forEach(stack => {
        stack.forEach(card => {
          card.selected = false
        })
      })
      this.selectedCards = []
    },

    createEmptyCard (stack) {
      stack.push({
        stack: this.stacks.indexOf(stack),
        empty: true
      })
    },

    checkCardsToMove (cards) {
      if (cards.length === 1) {
        return true
      }

      // Check color
      const isLikeFirstCardColor = card => card.color === cards[0].color
      const areColorsCorrect = cards.every(isLikeFirstCardColor)

      // Check values
      const firstValue = cards[0].value
      const cardsValues = cards.map(card => card.value)
      const areVlauesCorrect = cardsValues.every((value, index) => {
        return value === firstValue - index
      })

      return areColorsCorrect && areVlauesCorrect
    },

    // Check if cards on stack creating full color in correct order
    checkFullColor (stack) {
      let stackLength = 0
      const currentStackValues = []
      const currentStackColors = []

      stack.forEach(card => {
        if (!card.reversed) {
          stackLength++
          currentStackValues.push(card.value)
          currentStackColors.push(card.color)
        }
      })

      if (stackLength < this.nrOfCardsInColor) {
        return
      }

      let correctValues = 0

      // Check if cards are in good order
      for (let i = stackLength - 1; i > stackLength - this.nrOfCardsInColor; i--) {
        // Check values
        if (currentStackValues[i] + 1 === currentStackValues[i - 1]) {
          correctValues++
        } else {
          return
        }

        // Check colors
        if (currentStackColors[i] !== currentStackColors[i - 1]){
          return
        }
      }

      if (correctValues === this.nrOfCardsInColor - 1) { // Cards in good order
        stack.splice(- this.nrOfCardsInColor)
        if (stack.length === 0) {
          this.createEmptyCard (stack)
        }

        else if (stack[stack.length - 1].reversed === true) {
          stack[stack.length - 1].reversed = false
        }

        this.fullColor++
        this.checkWin()

        }

    },
    // Select card
    onCardSelect (card) {
      if (card.reversed) {
        return
      }

      const isAnyCardSelected = this.stacks.some(stack => {
        return stack.some(card => card.selected)
      })

      if (!isAnyCardSelected && card.empty) {
        return
      }

      if (!isAnyCardSelected) {

          const currentStack = this.stacks[card.stack]
          const cardIndex = currentStack.findIndex(cardInStack => {
            return cardInStack.id === card.id
          })
          const cardsOnTop = currentStack.slice(cardIndex)
          const isCardsSelectCorrect = this.checkCardsToMove(cardsOnTop)

          if (isCardsSelectCorrect) {
            cardsOnTop.forEach(card => card.selected = true)
            this.selectedCards.push(...cardsOnTop)
          } else {
            this.deselectCards()
          }

      } else { // If there is selected card

        // Diselect card if click on earlier selected card
        if (card.selected) {
          this.deselectCards()
          return
        }

        // Return if move on selected card is incorrect
        if (card.value - 1 !== this.selectedCards[0].value && !card.empty) {
          return
        }

        // Move cards
        const stackFrom = this.stacks[this.selectedCards[0].stack]
        const stackTo = this.stacks[card.stack]
        const indexFrom = stackFrom.findIndex(cardInStack => {
          return cardInStack.id === this.selectedCards[0].id
        })

        // Remove emptyCard if it was exist
        if (stackTo[0].empty === true) {
          stackTo.pop()
        }

        const movingCards = stackFrom.splice(indexFrom)
        movingCards.forEach(movingCard => movingCard.stack = card.stack)
        stackTo.push(...movingCards)

        this.checkFullColor(stackTo)

        this.deselectCards()

        // Reverse last card in the stack
        if (stackFrom.length > 0) {
          stackFrom[stackFrom.length - 1].reversed = false
        }
        else { // if moved card was the last one on the stack
          this.createEmptyCard(stackFrom)
        }

      }

    },
    addCards () {
      if (this.cards.length === 0) {
        return
      }

      const isEmptyCard = this.stacks.some(stack => stack[0].empty)
      const stacks = document.querySelector('.stacks')

      if (isEmptyCard) {
        stacks.classList.add('addCardsValidation')
        return
      }

      stacks.classList.remove('addCardsValidation')

      this.stacks.forEach((stack, idx) => {
        const card = this.cards.pop()
        card.stack = idx
        card.reversed = false
        stack.push(card)

        this.checkFullColor(stack)
      })
    },

    checkWin () {
      if (this.fullColor === this.nrOfFoundation) {
        this.showModalWin = true
      }
    }
  }
}
</script>

<style lang="scss">
  * {
    margin: 0;
    box-sizing: border-box;
    text-align: center;
    user-select: none;
    font-family: 'Raleway', sans-serif;
  }

  .container{
    min-height: 100vh;
    position: relative;
  }

  .header{
    display: flex;
    align-items: center;
    margin: 0 auto 25px auto;
    padding: 10px;

    &__title{
      flex: 1;
      font-size: 40px;
      color: #222;
    }

    &__button{
      background-color: #b70000;
      border-radius: 5px;
      padding: 10px;
      width: 40px;
      color: white;
      border: none;
      cursor: pointer;
      transition: all 0.2s;
      &:hover {
        background-color: #444;
      }
    }
  }

  .stacks {
    display: flex;
    justify-content: center;
    position: relative;
  }

  .menu{
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    margin-bottom: 70px;

    &__select{
      margin-left: 5px;
    }
  }

  .item{
    background-color: #b70000;
    border-radius: 5px;
    color: white;
    padding: 20px;
    font-size: 15px;
    &--button {
      border: none;
      cursor: pointer;
      transition: all 0.2s;
      &:hover {
        background-color: #444;
      }
    }
  }


  .addCardsValidation::before{
    content: 'Before add new cards, every stack has to be filled';
    color: red;
    position: absolute;
    top: -40px;
    left: 40%;
  }


  .img{
    position: absolute;
    z-index: -1;
    top: 0;

    &--left{
      left: -10vw;
      width: 70vw;
    }

    &--center{
      left: 55%;
      width: 12vw;
    }

    &--right{
      right: 2vw;
      width: 18vw;
    }
  }

</style>
