<template>
  <div class="container">
    <nav class="nav">
      <button @click="toggleRules">rules</button>
      <button @click="toggleAbout">about</button>
      <button @click="toggleSolution">solution</button>
    </nav>
    <div class="main">
      <h1>Target Puzzle</h1>
      <!-- Rules -->
      <div class="rules" v-if="showRules">
        <p class="rules text">
          There are {{ validWords.length }} 'common' words with four or more
          letters that can be made from the letters of this nine letter word.
          All words must contain the central letter.
        </p>
      </div>
      <!-- Square Grid -->
      <div :class="{ square: true, error: guessHasError }">
        <div
          v-for="(letter, index) in randomWordShuffled"
          :key="'letter' + index"
          :class="{ 'letter-box': true, used: usedLetters[index] }"
          @click="clickLetter(letter, index)"
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
          :style="{ background: inputColor, width: 'clamp(180px, 90%, 320px)' }"
          placeholder="Begin Typing..."
          @submit="checkInput"
          @keydown.enter.prevent="checkInput"
        />
        <button class="button tick" @click="checkInput">&#10003;</button>
        <!--tick-->
        <button class="button cross" @click="resetInput">&#10799;</button>
        <!--cross-->
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
          The word list is from the 10,000 most common English words as
          determined by n-gram frequency analysis of the Google's Trillion Word
          Corpus.
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
    <div class="footer relative">
      <CrosbySolutions />
    </div>
    <div class="modal" v-if="showSuccess">
      <Success
        :word="randomWord"
        :time="elapsedTime"
        :streak="streak"
        :fastest="fastest"
        :total="totalWords"
        :solved="totalSolved"
        @newWord="newWord"
        @sameWord="hideSuccess"
      />
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import CrosbySolutions from "@/components/crosby-solutions.vue";
import Success from "@/components/Success.vue";
interface StatObject {
  longWordStreak: number;
  gotLongWord: boolean[];
  gotPercent: boolean[];
  timeToLongWord: number[];
  timeToPercent: number[];
  timeTotal: number[];
}

interface HighlightObject {
  letter: string;
  letterCount: number;
  index: number;
  highlighted: boolean;
}
/**
 * function to check if all elements of first array are in a second, when elements may repeat
 * @param {string[]} source source to check against target
 * @param {string[]} target target for source to be checked against
 * @return {boolean}
 */
const allTheseAreInThat = function (
  source: string[],
  target: string[]
): boolean {
  if (source.length === 0) {
    return true;
  }
  // check if a letter is not in target, before checking count of every letter
  if (source.some((letter: string) => !target.includes(letter))) {
    return false;
  }
  return source
    .map((element: string, index: number, array: string[]) => {
      // for every element in the source, find its count and the count of the element in the target.
      const [sourceElementCount, targetElementCount] = [array, target].map(
        (a: string[]) => a.filter((l: string) => l === element).length
      );
      return sourceElementCount <= targetElementCount;
    })
    .every((l: boolean) => l);
};

/**
 * count how many times a letter is in an array of letters
 */
const countLetters = function (letter: string, letterArray: string[]): number {
  return letterArray.filter((l: string) => l === letter).length;
};
/**
 * function to sort guess list by length then alphabetically
 */
const sortLengthAlpha = function (a: string, b: string): number {
  if (a.length > b.length) return 1;
  if (a.length < b.length) return -1;
  if (a > b) return 1;
  if (a < b) return -1;
  return 0;
};

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
    /**
     * filter the word list to nine-letter words
     */
    const listNine: Array<string> = wordlist.filter(
      (word: string) => word.length === 9
    );

    return {
      wordlist,
      listNine,
      dictionary,
    };
  },
  components: {
    CrosbySolutions,
    Success,
  },
  data() {
    const showRules: boolean = false;
    const showAbout: boolean = false;
    const showAnswer: boolean = false;
    const showWords: boolean = false;
    const showSuccess: boolean = false;
    const showSolution: boolean = false;
    const groupByLength: number[] = [4, 5, 6, 7, 8, 9];
    const startTime: number = Date.now();
    return {
      showRules,
      showAbout,
      showSuccess,
      showSolution,
      showAnswer,
      showWords,
      startTime,
      clickedIndex: null as number | null,
      usedLetters: [] as boolean[],
      solved: false as boolean,
      streak: 0 as number,
      totalWords: 0 as number,
      totalSolved: 0 as number,
      fastest: null as number | null,
      elapsedTime: "" as string | number | unknown,
      // from async
      wordlist: [] as string[],
      dictionary: [] as string[],
      listNine: [] as string[],
      // user inputs, reset by newWord
      random: 0 as number,
      guess: "" as string,
      guessArray: [] as string[],
      otherArray: [] as string[],
      inputColor: "#efefef",
    };
  },
  mounted(): void {
    /**
     * Use the 'new Word' method when page is loaded to ensure consistent approach
     */
    this.newWord();
  },
  methods: {
    returnFocus(): boolean {
      if (document.activeElement !== document.getElementById("nextButton")) {
        document.getElementById("theInput")?.focus();
      }
      return false;
    },
    toggleHighlight(index: number): void {
      // editing content of an array, ensure reactivity
      Vue.set(this.usedLetters, index, !this.usedLetters[index]);
    },
    highlightLetter(letter: string, highlight: boolean): void {
      // if the input was clicked, use that index, else find an index mathching letter and desired highlight change
      const index =
        this.clickedIndex ||
        this.randomWordShuffled
          // find an index that is not highlighted
          .findIndex((l: string, i: number) => {
            return l === letter && this.usedLetters[i] === !highlight;
          });
      // if we have an index, toggle the highlight
      index >= 0 && this.toggleHighlight(index);
      // finished with clickdIndex, set null ahead of next input
      this.clickedIndex = null;
    },
    resetHighlight(): void {
      this.usedLetters = Array(9).fill(false);
    },
    hideSuccess(): void {
      let active = document.activeElement as HTMLElement;
      active.blur();
      this.showSuccess = false;
      this.returnFocus();
    },
    clickLetter(letter: string, index: number): void {
      // set the clicked index for highlight letter to pick the correct index
      this.clickedIndex = index;
      // allow the user to (re) click on the last letter in the guess to remove it from the guess
      if (this.usedLetters[index]) {
        // if the index clicked is highlighted
        if (this.guess.slice(-1).toUpperCase() === letter.toUpperCase()) {
          // remove the letter from guess
          this.guess = this.guess.slice(0, -1);
        }
      } else {
        // add clicked letter to the guess
        this.guess = this.guess + letter;
      }
      // to prevent loss of focus
      this.returnFocus();
    },
    /**
     * toggle app state
     */
    toggleAbout(): void {
      this.showAbout = !this.showAbout;
    },
    toggleRules(): void {
      this.showRules = !this.showRules;
    },
    toggleSolution(): void {
      this.showSolution = !this.showSolution;
    },
    /**
     * reset for new attempt
     */
    newWord(): void {
      // streak & stats
      if (!this.solved) {
        this.streak = 0; // will show as 1 when it is
      }
      this.totalWords++;

      // hide success window if coming from there
      this.hideSuccess();

      // reset for new attempt
      this.solved = false;
      this.guess = "";
      this.guessArray = [];
      this.otherArray = [];
      this.resetHighlight();

      // hide answers for new attempt
      this.showAnswer = false;
      this.showWords = false;
      this.showSolution = false;

      // generate new word
      this.random = Math.random();

      // start timer
      this.startTime = Date.now();
      this.elapsedTime = "";

      // set focus on input
      this.returnFocus();
    },
    /**
     * triggered when user guesses the 9 letter word
     */
    successRoutine(type: number = 9): void {
      // show feedback
      this.showSuccess = true;
      if (type === 9) {
        // calculate time elapsed and store to top scope to pass to component
        const nineElapsed = Date.now() - this.startTime;
        this.elapsedTime = +nineElapsed;
        console.log("nineElapsed: ", nineElapsed);
        this.solved = true;
        this.totalSolved++;
        this.streak++;

        // store fastest time
        if (this.fastest === null || nineElapsed < this.fastest) {
          this.fastest = nineElapsed;
        }
      }
      document.getElementById("nextButton")?.focus();
    },
    /**
     * input validation
     * fired on 'enter' key or input for submit
     * checks if
     * - word is in common word list,
     * - else is in dictionary,
     * - else doesn't exist
     * applies styling for ux
     * resets box for next input
     */
    checkInput(): void {
      const word = this.guess.toUpperCase();
      // word exists in google common list
      if (this.validWords.includes(word) && !this.guessArray.includes(word)) {
        this.guessArray.push(word);
        this.inputColor = "#8f8";
        // handle success cases
        if (word === this.randomWord) {
          this.successRoutine(9);
        }
        // if (this.percentFound == 100) {
        //   this.successRoutine(100);
        // }
      }
      // word exists in scrabble dictionary
      else if (
        !this.otherArray.includes(word) &&
        !this.guessArray.includes(word) &&
        this.validDictionary.includes(word)
      ) {
        this.inputColor = "#ff8";
        this.otherArray.push(word);
      }
      // word doesn't exist
      else {
        this.inputColor = "#f88";
      }
      // reset input box for next attempt
      setTimeout(this.resetInput, 500);
    },
    /**
     * reset the input after a guess
     * TODO: how to appropriately display the guess validation for a period of time and still capture subsequent input for the subsequent guess
     */
    resetInput(): void {
      this.inputColor = "#efefef";
      this.guess = "";
      this.resetHighlight();
      this.returnFocus();
    },

    /**
     * guess validation
     * 4+ letters
     * must include middle letter
     * all letters in the random word
     * extra check for multiple letters
     */
    wordFilter(word: string): boolean {
      // remove short words
      if (word.length < 4) return false;
      // remove words that don't have the desired letter
      if (!word.includes(this.middleLetter)) return false;
      // check every letter in the word
      return allTheseAreInThat([...word], this.randomWordShuffled);
    },
  },
  watch: {
    guess(n, o): void {
      // check if letter was added or removed, and update highlight accordingly
      const lastLetter = (o.length > n.length ? o : n).toUpperCase().slice(-1);
      const highlightBool = n.length > o.length; // add highlight (else remove)
      this.highlightLetter(lastLetter, highlightBool);
    },
  },
  computed: {
    /**
     * returns uppercase of guess
     */
    guessUCS(): string[] {
      return this.guess.toUpperCase().split("");
    },
    /**
     * logic to determine if the current guess has an error, for styling
     */
    guessHasError(): boolean {
      return !allTheseAreInThat(this.guessUCS, this.randomWordShuffled);
    },
    /**
     * percent of the solution space that has been guessed
     */
    percentFound(): number {
      const percent =
        100 *
        (this.guessArray.filter((word) => !!word).length /
          this.validWords.length);
      return +percent.toFixed(2);
    },
    /**
     * the target word
     */
    randomWord(): string {
      const randIndex = Math.floor(this.random * this.listNine.length);
      return this.listNine[randIndex];
    },
    /**
     * the word as an array of letters in shuffled order
     */
    randomWordShuffled(): string[] {
      return [...this.randomWord].sort(() => (Math.random() > 0.5 ? 1 : -1));
    },
    /**
     * the middle letter, required in all valid words
     */
    middleLetter(): string {
      return this.randomWordShuffled[4];
    },
    /**
     * apply the word filter to the wordlist
     */
    validWords(): string[] {
      return this.wordlist.filter(this.wordFilter);
    },
    /**
     * apply the word filter to the dictionary
     */
    validDictionary(): string[] {
      return this.dictionary.filter(this.wordFilter);
    },
    /**
     * sort the valid words for display
     */
    validWordsSorted(): string[] {
      return [...this.validWords].sort(sortLengthAlpha);
    },
    /** sort the guesses for display */
    guessArraySorted(): string[] {
      return [...this.guessArray].sort(sortLengthAlpha);
    },
    /** sort the other words for display */
    otherArraySorted(): string[] {
      return [...this.otherArray].sort(sortLengthAlpha);
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
.container {
  max-width: 320px;
  margin: auto;
  text-align: center;
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.nav {
  color: grey;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  flex: 0 0 initial;
}
.nav > button {
  border: 0;
}
.main {
  flex: 1 1 100%;
}
.footer {
  flex: 0 0 initial;
}
.about {
  font-size: small;
}
.text {
  text-align: left;
  margin-bottom: 1rem;
}
button {
  border: 1px grey solid;
  padding: 0.5rem 1rem;
  /* margin-bottom: 0.25rem; */
  margin-left: 0.25rem;
}
.input {
  display: flex;
  flex-direction: row;
  flex-wrap: none;
  align-items: center;
  justify-content: space-between;
}
#theInput {
  flex: 1 1 initial;
  border: 2px grey solid;
  padding: 0.5rem 1rem;
  margin: auto;
  width: clamp(180px, 90%, 320px);
}
.square {
  width: 180px;
  height: 180px;
  border: 2px grey solid;
  margin: 1rem auto;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
}
.button {
  flex: 0 0 intiial;
  height: 100%;
  width: 3rem;
  font-size: normal;
  border: 2px grey solid;
  border-radius: 3px;
}
.tick {
  background: #d3f8d3;
  border-color: green;
}
.cross {
  background: #ffd9c3;
  border-color: red;
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
  cursor: pointer;
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
  font-weight: 700;
}
.letter-box:nth-of-type(4) + .used > .letter {
  color: yellow;
}
.letter-box:nth-of-type(4) + .used {
  background: rgba(64, 64, 0, 1);
}
.solutions {
  padding: 1rem;
}
.modal {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
}
</style>
