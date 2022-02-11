<template>
  <div class="modal-background">
    <div class="modal-container">
      <div class="header">
        <h1>Success!</h1>
      </div>

      <div class="main">
        <p>
          You discovered the target word
          <span class="highlight">{{ word }}</span> in {{ time / 1000 }}s.
        </p>
        <br />
        <p>Your fastest discovery is {{ fastest / 1000 }}s.</p>
        <p>
          You have discovered {{ solved }} words out of {{ total }} attempts ({{
            Number.parseFloat((solved / total) * 100).toFixed(1)
          }}%)
        </p>
        <p>You're on a streak of {{ streak }} discoveries.</p>
      </div>

      <div class="footer">
        <button
          id="nextButton"
          class="button"
          @click="nextWord"
          @enter.stop="nextWord"
          @keypress.esc.prevent="sameWord"
          :autofocus="true"
        >
          Next Word
        </button>
        <button class="button" @click="sameWord">Continue</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
export default Vue.extend({
  name: "Success",
  props: {
    word: { required: true },
    time: { required: true },
    streak: { required: true },
    fastest: { required: true },
    total: { required: true },
    solved: { required: true },
  },
  mounted() {
    document.getElementById("nextButton")?.focus();
  },
  methods: {
    nextWord() {
      this.$emit("newWord");
    },
    sameWord() {
      this.$emit("sameWord");
    },
  },
});
</script>

<style scoped>
.modal-background {
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(5px);
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  position: absolute;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.modal-container {
  display: flex;
  flex-direction: column;
  background-image: linear-gradient(rgb(159, 253, 159), rgb(205, 255, 205));
  margin: auto;
  padding: 2rem;
  height: clamp(450px, 40vh, 640px);
  width: clamp(300px, 60vw, 480px);
  justify-content: space-around;
  border: 1px grey solid;
}
.footer {
  flex: 0 0 initial;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}
.header {
  flex: 0 0 initial;
}
h1 {
  font-size: xx-large;
}
.main {
  flex: 1 1 initial;
}
button {
  padding: 0.5rem 1rem;
  background: #ccc;
  border: 1px #888 solid;
}
.highlight {
  font-size: large;
}
</style>
