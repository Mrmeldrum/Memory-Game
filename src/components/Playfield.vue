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
      picsimApiUrl: 'https://picsum.photos/200',
      turns: 0,
    }
  },
  mounted() {
    this.getImages()
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
    }
    
  },
  watch: {
    images: function(val) {
      if (val.length === this.pairs) {
        this.genrateCards()
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
 * {
     box-sizing: border-box;
 }
</style>