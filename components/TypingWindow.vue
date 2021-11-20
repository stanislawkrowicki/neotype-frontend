<template>
  <div class="main-window" @click="startTest">
    <div class="words-container" ref="words-container">
      <input
        type="text"
        id="words-input"
        ref="typing-input"
        v-on:keydown="keyPressed"
        autocomplete="off"
        autocapitalize="off"
        autocorrect="off"
      />
      <div v-if="displayCaret" ref="caret" id="caret"></div>
      <div v-for="word in words" :key="word" ref="words" class="word">
        <div v-for="letter in word" :key="letter" ref="letters" class="letter">
          <span>{{ letter }}</span>
        </div>
        <div class="fake-letter"></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [
        "age",
        "eyes",
        "wildblue",
        "jingle",
        "blues",
        "californication",
        "wet",
        "sand",
        "space",
        "font",
        "height",
        "dependency",
      ],
      currentWordIndex: 0,
      currentLetterIndex: 0,
      correctLetters: 0,
      displayCaret: true,
    };
  },

  methods: {
    startTest() {
      this.$refs["typing-input"].focus();
      this.moveCaret();
    },

    moveCaret() {
      let caret = this.$refs.caret;

      if (!caret) return;

      let caretParent = caret.parentNode;
      if (!caretParent) return;

      let currentWord = this.$refs.words[this.currentWordIndex];
      let currentLetter =
        currentWord.querySelectorAll(".letter")[this.currentLetterIndex];

      if (!currentLetter) {
        let fakeLetter = currentWord.querySelector(".fake-letter");
        caretParent.removeChild(caret);
        fakeLetter.appendChild(caret);
        return;
      }

      caretParent.removeChild(caret);
      currentLetter.appendChild(caret);
      this.displayCaret = true;
    },

    keyPressed(e) {
      if (e.code == "Space") {
        this.currentWordIndex++;
        this.currentLetterIndex = 0;
        this.moveCaret();
        return;
      }

      let word = this.$refs.words[this.currentWordIndex];
      if (word === undefined) return;

      let letter = word.querySelectorAll(".letter")[this.currentLetterIndex];
      if (letter === undefined) return;

      let expectedKey = letter.querySelector("span").innerHTML;

      if (e.key == expectedKey) {
        this.correctLetters++;
        letter.className = `${letter.className} correct`;
      } else letter.className = `${letter.className} incorrect`;

      this.currentLetterIndex++;
      this.moveCaret();
    },
  },
};
</script>

<style lang="scss" scoped>
$font-size: 4em;

.main-window {
  width: 80em;
  height: 30em;
  background-color: $background-color;
  text-align: justify;
  padding: 0.4em;
}

.word {
  display: inline-block;
  margin-right: 1em;
}

.letter,
.fake-letter {
  display: inline-block;
  position: relative;
  span {
    color: $primary-color;
    font-size: $font-size;
    font-family: "Shippori Antique", sans-serif;
    opacity: 0.4;
  }
  #caret {
    position: absolute;
    height: $font-size * 1.5;
    width: 0.5em;
    background-color: $secondary-color;
    opacity: 0.8;
    top: 0;
  }
}

.fake-letter {
  position: absolute;
}

.correct > span {
  opacity: 0.8;
  color: $primary-color;
}

.incorrect > span {
  opacity: 0.5;
  color: $error-color;
}

#words-input {
  z-index: -1;
  padding: 0;
  margin: 0;
  border: none;
  outline: none;
  display: block;
  position: fixed;
  cursor: default;
  pointer-events: none;
}
</style>
