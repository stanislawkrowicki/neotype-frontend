<template>
  <div class="main-window" @click="startTest">
    <transition name="fade">
      <div v-show="!inProgress" class="not-focused-message">
        <p>Click anywhere to focus...</p>
      </div>
    </transition>

    <div
      class="words-container"
      :class="inProgress ? '' : 'blur'"
      ref="words-container"
      @click="startTest"
      v-if="!testFinished"
    >
      <div class="timer">
        <div
          v-for="time in timerOptions"
          :key="time"
          class="timer-option-wrapper"
        >
          <input
            type="radio"
            name="time"
            v-bind:id="`time-${time}`"
            class="timer-option"
            :value="time"
            ref="timer-option"
            :checked="time == timerOptions[defaultTimerSelectionIndex]"
            @click="restartTest"
          />
          <label v-bind:for="`time-${time}`">{{ time }}</label>
        </div>
      </div>
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
      <div v-for="word in words" :key="`${tryNumber}_${word.id}`" class="word">
        <div
          v-for="(letter, index) in word.word"
          :key="`${tryNumber}_${word.id}_${index}`"
          class="letter"
        >
          <span>{{ letter }}</span>
        </div>
        <div class="fake-letter"></div>
      </div>
    </div>

    <div v-else class="results" @click="playAgain">
      <p id="heading">You finished! Your score:</p>
      <p id="result-wpm">{{ resultWPM }} WPM</p>
      <p id="play-again">Press anywhere to play again.</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [],
      timerOptions: [15, 30, 60],
      defaultTimerSelectionIndex: 1, // defaults to the second timerOption
      selectedTime: 30,
      currentWordIndex: 0,
      currentLetterIndex: 0,
      correctLetters: 0,
      wordsFromNewlineCounter: 0,
      tryNumber: 0,
      displayCaret: true,
      inProgress: false,
      testFinished: false,
      resultWPM: 0,
    };
  },

  methods: {
    startTest() {
      this.$refs["typing-input"].focus();
      if (this.inProgress) return;

      this.testFinished = false;
      this.tryNumber++;
      if (this.words.length == 0) this.loadWords();
      this.$nextTick(() => {
        this.moveCaret();
        this.inProgress = true;
        this.selectedTime = this.$refs["timer-option"].find(
          (input) => input.checked
        ).value;
        if (this.endTimeout) clearTimeout(this.endTimeout);
        this.endTimeout = setTimeout(this.endTest, this.selectedTime * 1000);
      });
    },

    endTest() {
      const LETTERS_IN_WORD = 5;
      let words = this.correctLetters / LETTERS_IN_WORD;
      let wpm = words / (this.selectedTime / 60);

      this.resultWPM = Math.floor(wpm);

      if (localStorage.getItem("token") !== null) {
        let auth = "Bearer " + localStorage.getItem("token");
        let config = {
          headers: {
            Authorization: auth,
          },
        };

        this.$axios
          .post(
            "/result",
            { wpm: this.resultWPM, time: parseInt(this.selectedTime) },
            config
          )
          .catch(() => {});
      }

      this.testFinished = true;
    },

    playAgain() {
      this.testFinished = false;
      this.$nextTick(() => {
        this.restartTest();
      });
    },

    async loadWords() {
      const WORDS_COUNT = 50;
      // TODO: handle request error
      let request = await this.$axios.get("/words/" + WORDS_COUNT);
      let wordsToLoad = request.data;
      let currentWordID = 0;

      this.words = [];

      wordsToLoad.forEach((word) => {
        this.words.push({ word: word, id: currentWordID });
        currentWordID++;
      });
    },

    async loadMoreWords() {
      const WORDS_COUNT = 40;
      // TODO: handle request error
      let request = await this.$axios.get("/words/" + WORDS_COUNT);
      let wordsToLoad = request.data;

      let lastWord = this.words[this.words.length - 1];
      let currentWordID = 0;
      if (lastWord.id) currentWordID = lastWord.id + 1;

      wordsToLoad.forEach((word) => {
        this.words.push({ word: word, id: currentWordID });
        currentWordID++;
      });
    },

    loadWordsIfNeeded() {
      const THRESHOLD = 50; // words left
      if (this.words.length - this.currentWordIndex <= THRESHOLD)
        this.loadMoreWords();
    },

    restartTest() {
      if (this.endTimeout) clearTimeout(this.endTimeout);
      this.currentWordIndex = 0;
      this.currentLetterIndex = 0;
      this.correctLetters = 0;
      this.wordsFromNewlineCounter = 0;
      this.inProgress = false;
      this.startTest();
    },

    updateTest() {
      if (this.moveCaret()) {
        this.words.splice(0, this.wordsFromNewlineCounter);
        this.currentWordIndex -= this.wordsFromNewlineCounter;
        this.wordsFromNewlineCounter = 0;
      }

      this.loadWordsIfNeeded();
    },

    /**
     * Returns bool: did caret move to a new line
     */
    moveCaret() {
      let caret = this.$refs.caret;

      if (!caret) return;

      const caretHeight = caret.getBoundingClientRect().top;

      let caretParent = caret.parentNode;
      if (!caretParent) return;

      let currentWord =
        this.$refs["words-container"].querySelectorAll(".word")[
          this.currentWordIndex
        ];

      if (!currentWord) return;

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

      return caret.getBoundingClientRect().top != caretHeight;
    },

    keyPressed(e) {
      if (e.code == "Space") {
        let currentWord =
          this.$refs["words-container"].querySelectorAll(".word")[
            this.currentWordIndex
          ];
        currentWord.querySelectorAll(".letter").forEach((letter) => {
          if (
            letter.classList.contains("correct") ||
            letter.classList.contains("incorrect")
          )
            return;
          letter.classList.add("incorrect");
        });
        this.currentWordIndex++;
        this.currentLetterIndex = 0;
        this.wordsFromNewlineCounter++;
        this.updateTest();
        return;
      }

      let word =
        this.$refs["words-container"].querySelectorAll(".word")[
          this.currentWordIndex
        ];

      if (word === undefined) return;

      let letter = word.querySelectorAll(".letter")[this.currentLetterIndex];
      if (letter === undefined) return;

      let expectedKey = letter.querySelector("span").innerHTML;

      if (e.key == expectedKey) {
        this.correctLetters++;
        letter.classList.add("correct");
      } else letter.classList.add("incorrect");

      this.currentLetterIndex++;
      this.updateTest();
    },
  },

  mounted() {
    this.loadWords();
  },
};
</script>

<style lang="scss" scoped>
$font-size: 3em;

@media only screen and (min-width: 850px) {
  .main-window {
    font-size: 150%;
  }
}

.main-window {
  width: 70vw;
  height: 50vh;
  overflow: hidden;
  background-color: $background-color;
  text-align: justify;
  padding: 0.4em;
}

.words-container {
  &.blur {
    filter: blur(0.3em);
  }
  transition: blur 0.3s ease;
}

.not-focused-message {
  position: absolute;
  display: table;
  height: 50vh;
  width: 70vw;
  p {
    display: table-cell;
    text-align: center;
    width: 100%;
    vertical-align: middle;
    color: $secondary-color;
    font-size: 2 * $font-size;
    opacity: 0.8;
  }
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
    font-family: $font-family;
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
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.timer {
  float: right;
}

.timer-option-wrapper {
  display: inline-block;
  label {
    font-family: $font-family;
    color: $primary-color;
    opacity: 0.8;
  }
  input.timer-option {
    appearance: none;
  }
  input.timer-option:checked + label {
    opacity: 1;
    font-weight: bold;
  }
}

.results {
  font-family: $font-family;
  color: white;
  text-align: center;
  #heading {
    font-size: 2.5rem;
  }
  #result-wpm {
    font-size: 4rem;
  }
  #play-again {
    opacity: 0.7;
  }
}

@media only screen and (min-width: 850px) {
  .results {
    #heading {
      font-size: 3rem;
    }
    #result-wpm {
      font-size: 5rem;
    }
  }
}
</style>
