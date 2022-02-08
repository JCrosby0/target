<template>
  <div class="container">
    <nav class="nav">
      <button @click="toggleRules">rules</button>
      <button @click="toggleAbout">about</button>
      <button @click="toggleSolution">solution</button>
    </nav>
    <h1>Target puzzle:</h1>
    <!-- Rules -->
    <div class="rules" v-if="showRules">
      <p class="rules text">
        There are {{ validWords.length }} 'common' words with four or more
        letters that can be made from the letters of this nine letter word. All
        words must contain the central letter.
      </p>
    </div>
    <!-- Square Grid -->
    <div :class="{ square: true, error: guessHasError }">
      <div
        v-for="(letter, index) in randomWordShuffled"
        :key="'letter' + index"
        :class="{ 'letter-box': true, used: guessUsesLetter[index] }"
      >
        <div class="letter">{{ letter }}</div>
      </div>
    </div>
    <!-- Input box -->
    <div class="input">
      <input
        type="text"
        id="theInput"
        v-model="guess"
        autocomplete="off"
        autofocus
        :style="{ background: inputColor }"
        @submit="checkInput"
        @input="updateStyles"
        @keyup.enter="checkInput"
      />
    </div>
    <!-- solution space -->
    <div class="solutions">
      <h2>Words Found: {{ guessArray.length }} ({{ percentFound }}%)</h2>
      <span
        v-for="solution in guessArraySorted"
        :key="solution"
        :style="{
          textTransform: solution.length === 9 ? 'uppercase' : 'lowercase',
        }"
      >
        {{ solution }}
      </span>
      <h2>Other Words: {{ otherArray.length }}</h2>
      <span
        v-for="solution in otherArraySorted"
        :key="solution"
        :style="{
          textTransform: solution.length === 9 ? 'uppercase' : 'lowercase',
        }"
      >
        {{ solution }}
      </span>
    </div>
    <!-- about -->
    <div class="about" v-if="showAbout">
      <p class="text">
        The word list is from the 10,000 most common English words as determined
        by n-gram frequency analysis of the Google's Trillion Word Corpus.
        <a
          href="https://github.com/first20hours/google-10000-english"
          target="_blank"
          >Github Source</a
        >. Other words are checked against a Scrabble wordlist.
      </p>
    </div>
    <!-- solutions -->
    <div class="solutions" v-if="showSolution">
      <h2>Solutions:</h2>
      <button @click="showAnswer = !showAnswer">Target Word</button><br />
      <p v-if="showAnswer">
        The word used to generate the puzzle is: {{ randomWord }}
      </p>
      <button @click="showWords = !showWords">Word List</button>
      <p v-if="showWords">{{ validWordsSorted }}</p>
    </div>
    <button @click="newWord">New Word</button>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

interface StatObject {
  gotLongWord: boolean[];
  gotPercent: boolean[];
  timeToLongWord: number[];
  timeToPercent: number[];
  timeTotal: number[];
}

export default Vue.extend({
  async asyncData() {
    const text = await fetch("/wordlist.txt").then((response) =>
      response.text()
    );
    const wordlist: string[] = text
      .split("\n") // identify the newline character and split
      .map((word) => word.toUpperCase());
    const scrabble = await fetch("/wordlist_scrabble.txt").then((response) =>
      response.text()
    );
    const dictionary: string[] = scrabble
      .split("\n")
      .map((word) => word.toUpperCase());
    return {
      wordlist: wordlist as string[],
      dictionary: dictionary as string[],
    };
  },
  data() {
    const message: string = "This is a message";
    const length: number = 9;
    const showRules: boolean = false;
    const showAbout: boolean = false;
    const showAnswer: boolean = false;
    const showWords: boolean = false;
    const showSolution: boolean = false;
    const groupByLength: number[] = [4, 5, 6, 7, 8, 9];
    return {
      message,
      length,
      showRules,
      showAbout,
      showSolution,
      showAnswer,
      showWords,
      wordlist: [] as string[],
      dictionary: [] as string[],
      random: Math.random(),
      guess: "" as string,
      guessArray: [] as string[],
      otherArray: [] as string[],
      inputColor: "#efefef",
    };
  },
  methods: {
    updateStyles(event: InputEvent): void {
      const target = event.target as HTMLInputElement;
      const inputLetter = event.data;
      const inputWord = target.value;
    },
    toggleAbout(): void {
      this.showAbout = !this.showAbout;
    },
    toggleRules(): void {
      this.showRules = !this.showRules;
    },
    toggleSolution(): void {
      this.showSolution = !this.showSolution;
    },
    newWord(): void {
      this.random = Math.random();
      this.guess = "";
      this.guessArray = [];
      this.guessHasError = false;
      this.otherArray = [];
      this.showAnswer = false;
      this.showWords = false;
    },
    checkInput(): void {
      const word = this.guess.toUpperCase();
      // word exists in google common list
      if (this.validWords.includes(word) && !this.guessArray.includes(word)) {
        this.guessArray.push(word);
        this.setInputColor("#8f8");
      }
      // word exists in scrabble dictionary
      else if (
        this.validDictionary.includes(word) &&
        !this.otherArray.includes(word)
      ) {
        this.setInputColor("#ff8");
        this.otherArray.push(word);
      }
      // word doesn't exist
      else {
        this.setInputColor("#f88");
      }
    },
    setInputColor(color: string): void {
      this.inputColor = color;
      setTimeout(this.resetInput, 500);
    },
    resetInput(): void {
      this.inputColor = "#efefef";
      this.guess = "";
    },
    sortLengthAlpha(a: string, b: string): number {
      if (a.length > b.length) return 1;
      if (a.length < b.length) return -1;
      if (a > b) return 1;
      if (a < b) return -1;
      return 0;
    },
    wordFilter(word: string): boolean {
      // remove short words
      if (word.length < 4) return false;
      // remove words that don't have the desired letter
      if (!word.includes(this.middleLetter)) return false;
      // check every letter in the word
      return [...word].every((letter: string) => {
        // if the letter is in the randomword
        if (this.randomWord.includes(letter)) {
          // check that we have enough letters in the random word
          const noLetterInWord: number = [...word].filter(
            (l: string) => l === letter
          ).length;
          const noLetterInRandomWord: number = [...this.randomWord].filter(
            (l: string) => l === letter
          ).length;
          if (noLetterInRandomWord >= noLetterInWord) return true;
        }
        return false;
      });
    },
  },
  computed: {
    guessUsesLetter(): boolean[] {
      return this.randomWordShuffled.map((letter, index, array) => {
        return (
          this.guess.toUpperCase().includes(letter) &&
          array.slice(index).filter((l) => l === letter).length <=
            this.guess
              .toUpperCase()
              .split("")
              .filter((l) => l === letter).length
        );
      });
    },
    guessHasError(): boolean {
      const gUS = this.guess.toUpperCase().split("") as string[];
      return !gUS.every((letter) => {
        return (
          this.randomWord.includes(letter) && // word contains the letter
          gUS.filter((l) => l === letter).length <= // word contains more of the letter than guess
            this.randomWord.split("").filter((l) => l === letter).length
        );
      });
    },
    percentFound(): number {
      const percent =
        100 *
        (this.guessArray.filter((word) => !!word).length /
          this.validWords.length);
      return +percent.toFixed(2);
    },
    inputClass(): object {
      return {
        background: this.inputColor,
      };
    },
    listNine(): Array<string> {
      const result: Array<string> = this.wordlist.filter(
        (word: string) => word.length === this.length
      );
      return result;
    },
    randomWord(): string {
      const randIndex = Math.floor(this.random * this.listNine.length);
      return this.listNine[randIndex];
    },
    randomWordShuffled(): Array<string> {
      return [...this.randomWord].sort(() => (Math.random() > 0.5 ? 1 : -1));
    },
    middleLetter(): string {
      return this.randomWordShuffled[4];
    },
    // filter the word list down to candidate words
    // cull words in stages to minimize duplication of checks
    validWords(): string[] {
      return this.wordlist.filter(this.wordFilter);
    },
    validDictionary(): string[] {
      return this.dictionary.filter(this.wordFilter);
    },
    validWordsSorted(): string[] {
      return [...this.validWords].sort(this.sortLengthAlpha);
    },
    guessArraySorted(): string[] {
      return [...this.guessArray].sort(this.sortLengthAlpha);
    },
    otherArraySorted(): string[] {
      return [...this.otherArray].sort(this.sortLengthAlpha);
    },
  },
});
</script>

<style scoped>
h1 {
  font-size: xx-large;
}
.rules {
  font-size: medium;
}
.nav {
  color: grey;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
.nav > button {
  border: 0;
}
.about {
  font-size: small;
}
.container {
  max-width: 320px;
  margin: auto;
  text-align: center;
}
.text {
  text-align: left;
  margin-bottom: 1rem;
}
button {
  border: 1px grey solid;
  padding: 0.5rem 1rem;
  margin-bottom: 0.25rem;
  margin-right: 0.25rem;
}
#theInput {
  border: 2px grey solid;
  padding: 0.5rem 1rem;
  margin: 1rem;
}
.square {
  width: 180px;
  height: 180px;
  border: 2px grey solid;
  margin: 2rem auto;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}

@keyframes shake {
  33% {
    transform: translateX(0.5rem);
  }
  66% {
    transform: translateX(-0.5rem);
  }
}
.error {
  border: 2px red solid;
  animation-name: shake;
  animation-duration: 150ms;
  animation-iteration-count: 2;
}

.letter-box {
  position: relative;
  flex: 1 1 33%;
  border: 1px grey solid;
  box-sizing: border-box;
  text-align: center;
}
.error > .letter-box {
  border: 1px red solid;
}
.letter {
  margin: 0;
  font-size: x-large;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.used {
  background-color: yellow;
}
.letter-box:nth-of-type(5) {
  background: black;
  color: white;
}
.letter-box:nth-of-type(4) + .used > .letter {
  color: yellow;
}
.solutions {
  padding: 1rem;
}
</style>
