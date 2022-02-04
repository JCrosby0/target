<template>
  <div class="root">
    <!-- <p>Select words of length:</p>
    <input v-model="length" type="number"> -->
  <!-- <p>
    There are {{listNine.length}} words of length {{ length }}
    </p> -->
    <!-- <p>Shuffling remaining letters: {{randomWordShuffled}}</p>
    <p>Middle Letter letter: {{middleLetter}}</p> -->
    <p>Target puzzle: All words must contain the central letter</p>
    <div class="square flex flex-wrap border border-blue">
      <div v-for="(letter, index) in randomWordShuffled" :key="'letter'+index" class="letter-box">
        <div class="letter">
          {{letter}}
        </div>
      </div>
    </div>
    <p>
      There are {{ validWords.length }} words with four or more letters that can be made from these letters.
    </p>
    <button @click="showAnswer = !showAnswer">Toggle Answer</button>
    <p v-if="showAnswer">
      The word used to generate the puzzle is: {{randomWord}}
    </p>
    <button @click="showWords = !showWords">Toggle Words</button>
    <p v-if="showWords">{{validWordsSorted}}</p>

  </div>
</template>

<script lang="ts">
// import Vue from 'vue'
// export default Vue.extend({})
import Vue from 'vue'

export default Vue.extend({
  async asyncData() {
    const text = await fetch('/wordlist.txt').then(response => response.text())
    const wordlist: string[] = text.split(text[2])
    return {
      wordlist
    }
  },
  data () {
    const message: string = 'This is a message'
    const length: number = 9
    const showAnswer: boolean = false
    const showWords: boolean = false
    const groupByLength: number[] = [4, 5, 6, 7, 8, 9]
    return {
      message,
      length,
      showAnswer,
      showWords
    }
  },
  computed: {
    listNine() {
      const result: Array<string> = this.wordlist.filter((word: string) => word.length===this.length)
      return result
    },
    randomWord() {
      const randIndex = Math.floor(Math.random() * this.listNine.length)
      return this.listNine[randIndex]
    },
    randomWordShuffled() {
      return [...this.randomWord].sort(() => (Math.random() > 0.5) ? 1 : -1)
    },
    middleLetter() {
      return this.randomWordShuffled[4]
    },
    // filter the word list down to candidate words
    // cull words in stages to minimize duplication of checks
    validWords() {
      return this.wordlist.filter((word: string[]) => {
        // remove short words
        if (word.length < 4) return false
        // remove words that don't have the desired letter
        if (!word.includes(this.middleLetter)) return false
        // check every letter in the word
        return [...word].every((letter: string) => {
          // if the letter is in the randomword
          if (this.randomWord.includes(letter)) {
            // check that we have enough letters in the random word
            const noLetterInWord: number = [...word].filter((l: string) => l === letter).length
            const noLetterInRandomWord: number = [...this.randomWord].filter((l: string) => l === letter).length
            if (noLetterInRandomWord >= noLetterInWord) return true
          } 
          return false
        })
      })
    },
    validWordsSorted() {
      return [...this.validWords].sort((a: string, b: string) => {
        if (a.length > b.length) return 1
        if (a.length < b.length) return -1
        if (a > b) return 1
        if (a < b) return -1
        return 0
      })
    }
  }
})

</script>

<style scoped>
button {
  border: 1px grey solid;
  padding: 0.5rem 1rem;
}
.square {
  width: 180px;
  height: 180px;
  border: 1px grey solid;
  margin: 2rem;
}

.letter-box {
  position: relative;
  width: 33.3%;
  height: 33.3%;
  border: 1px grey solid;
  box-sizing: border-box;
  text-align: center;
}
.letter {
  margin: 0;
  font-size: x-large;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.letter-box:nth-of-type(5){
  background: black;
  color: white;

}

</style>
