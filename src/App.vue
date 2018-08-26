<template>
  <div class="container">
    <h1 class="title">Spider Solitaire</h1>
    <section class="menu">
      <button @click="addCards" class="menu__button" type="button">
        Add cards ({{ cards.length / 10 }})
      </button>
      <div class="menu__score">
        Full colors: {{ fullColor }} / 8
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
  </div>
</template>

<script>
import _ from 'lodash'
import cardComponent from '@/components/card'
import stackComponent from '@/components/stack'
import { CLUB, DIAMOND, SPADE, HEART } from '@/cardTypes'

export default {
  components: {
    cardComponent,
    stackComponent
  },
  data () {
    return {
      stacks: Array(10).fill().map(() => []),
      cards: [],
      colors: [CLUB, DIAMOND, SPADE, HEART],
      selectedCards: [],
      fullColor: 0
    }
  },
  created () {
    // Create cards
    for (let i = 0; i < 13 * 8; i++) {
      this.cards.push({
        id: i,
        value: i % 13,
        color: this.colors[i % 4],
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
    for(let stackNumber = 0; stackNumber < 4; stackNumber++){
      for (let i = 0; i < 6; i++) {
        const card = this.cards.pop()
        card.stack = stackNumber
        this.stacks[stackNumber].push(card)
      }
    }

    for (let stackNumber = 4; stackNumber < 10; stackNumber++) {
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

  methods: {
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
    checkFullColor (stack) {
      let stackLength = 0
      const currentStackValues = []
      const fullColorLength = 13

      stack.forEach(card => {
        if (!card.reversed) {
          stackLength++
          currentStackValues.push(card.value)
        }
      })

      if (stackLength < fullColorLength) { // Not enough cards
        return
      } else {
        let correct = 0

        // Check if cards are in good order
        for (let i = stackLength - 1; i > stackLength - 13; i--) {
          if (currentStackValues[i] + 1 === currentStackValues[i - 1]) {
            correct++
          } else {
            return
          }
        }

        if (correct === 12) { // Cards in good order
          stack.splice(-13)

          if (stack.length === 0) {
            this.createEmptyCard (stack)
          }
          else if (stack[stack.length - 1].reversed === true) {
            stack[stack.length - 1].reversed = false
          }

          this.fullColor++

        }
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
      this.stacks.forEach((stack, idx) => {
        const card = this.cards.pop()
        card.stack = idx
        card.reversed = false
        stack.push(card)
      })
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
  }

  .title{
    font-size: 30px;
    padding: 20px;
  }

  .stacks {
    display: flex;
    justify-content: center;
  }

  .menu{
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-bottom: 20px;

    &__button {
      width: 150px;
      background-color: #b70000;
      border-radius: 5px;
      border: none;
      color: white;
      cursor: pointer;
      transition: all 0.2s;
      padding: 10px;
      &:hover {
        background-color: #444;
      }
    }

    &__score{
      background-color: #b70000;
      border-radius: 5px;
      border: none;
      color: white;
      padding: 10px;
    }
  }


</style>
