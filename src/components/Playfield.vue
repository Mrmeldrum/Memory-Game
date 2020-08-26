<template>
    <div class="playfield">
      <div v-if="loading" class="spinner">
        <div>
            <img src="https://raw.githubusercontent.com/daanfl/match-match-vue/master/src/assets/spinner.gif" />
        </div>
        <span>Loading Game...</span>
      </div>

      <div v-else>
        <div class="statistics">
            <span>Turns: {{turns}}</span>
            <span>Pairs: {{pairsMatched}} of {{pairs}}</span>
            <button v-if="pairsMatched === pairs" @click="resetGame">New Game</button>
        </div>

        <Card v-for="card in deck" :key="card.number" :card="card" />
      </div>
    </div>
</template>

<script>
import axios from 'axios'
import Card from './Card.vue'

export default {
    props: {
        pairs: Number
    },
    components: {
        Card
    },
    data: () => {
    return {
      deck: [],
      loading: true,
      openedCards: [],
      pairsMatched: 0,
      pairVisibleInMiliseconds: 1500,  
      images: [],
      picsumApiUrl: 'https://picsum.photos/200',
      turns: 0,
    }
  },
  mounted() {
    this.getImages()

    this.$on('onCardOpen', function(card) {
        this.openCard(card)
    });
  },
  watch: {
    images: function(val) {
      if (val.length === this.pairs) {
        this.generateCards()
        this.loading = false
      }
    }
  },
  methods: {
    getImages: async function() {
      while (this.images.length < this.pairs) {
        const response = await axios.get(this.picsumApiUrl)

        if (response.request.responseURL) {
          this.images.push(response.request.responseURL)
        }
      }
    },
    cardsMatch: function(cards) {
        return cards[0].pair === cards[1].pair
    },
    closeCards: function() {
        this.deck.forEach((card) => {
            card.open = false
        })
    },
    generateCards: function() {
      let cards = []
      let cardNumber = 0

      this.images.forEach((image, key) => {
        for (let i = 0; i<2; i++) {
          cardNumber += 1

          cards.push({
            number: cardNumber,
            pair: key,
            image: image,
            open: false,
            matched: false
          })
        }
      })
      this.deck = this.shuffleDeck(cards)
    },
    openCard: function(card) {
        if (this.openedCards.length === 2) {
            return
        }

        this.deck.forEach((element, index) => {
            if (element.number === card.number) {
                this.deck[index].open = true
                return
            }
        })

        this.openedCards.push(card)

        if (this.openedCards.length === 2) {
            setTimeout(() => {
            this.handlePossibleMatch(this.openedCards)
            }, this.pairVisibleInMiliseconds)
        }
    },
    shuffleDeck: function(cards) {
        return cards
            .map(a => [Math.random(), a])
            .sort((a, b) => a[0] - b[0])
            .map(a => a[1])
    },
    resetGame: function() {
        this.generateCards()
        this.turns = 0
        this.pairsMatched = 0
    },
    handlePossibleMatch: function(openedCards) {
        if (this.cardsMatch(openedCards)) {
            const openedCardNumbers = [openedCards[0].number, openedCards[1].number]

            this.deck.forEach((element, index) => {
                if (openedCardNumbers.includes(element.number)) {
                    this.deck[index].open = false
                    this.deck[index].matched = true
                }
            })

            this.pairsMatched += 1
        } else {
            this.closeCards()
        }
        this.turns += 1
        this.openedCards = []
    }
    
  }
}
</script>

<style scoped>
  * {
    box-sizing: border-box;
  }
  .spinner {
    width: 100%;
    text-align: center;
  }
  .spinner span {
    font-size: 24px;
  }
  .statistics span {
    display: inline-block;
    font-size: 24px;
    margin: 25px;
  }
</style>