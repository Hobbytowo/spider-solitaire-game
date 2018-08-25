<template>
  <div class="container">
    <h1 class="title">Spider Solitaire</h1>
    <button @click="addCards" class="button" type="button">
      Add cards ({{ cards.length / 10 }})
    </button>
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
      selectedCards: []
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
    // Select card
    onCardSelect (card) {
      if (card.reversed) {
        return
      }

      const isAnyCardSelected = this.stacks.some(stack => {
        return stack.some(card => card.selected)
      })

      if (isAnyCardSelected) {
        // Diselect card if click on earlier selected card
        if (card.selected) {
          this.deselectCards()
          return
        }

        // Check if move on selected card is possible
        if (card.value - 1 !== this.selectedCards[0].value) {
          return
        }

        // Move cards
        const stackFrom = this.stacks[this.selectedCards[0].stack]
        const stackTo = this.stacks[card.stack]
        const indexFrom = stackFrom.findIndex(cardInStack => {
          return cardInStack.id === this.selectedCards[0].id
        })

        const movingCards = stackFrom.splice(indexFrom)
        movingCards.forEach(movingCard => movingCard.stack = card.stack)
        stackTo.push(...movingCards)

        this.deselectCards()
        // Reverse last card in the stock
        if (stackFrom.length > 0) {
          stackFrom[stackFrom.length - 1].reversed = false
        }

      } else { // If there is no selected card

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
  }

  .button {
    width: 150px;
    height: 30px;
    background-color: #b70000;
    border-radius: 5px;
    border: none;
    color: white;
    cursor: pointer;
    transition: all 0.2s;
    margin-bottom: 30px;
    &:hover {
      background-color: #444;
    }
  }
</style>
