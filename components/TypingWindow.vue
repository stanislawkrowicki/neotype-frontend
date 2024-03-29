<template>
  <div class="main-window" @click="startTest">
    <div v-if="words.length == 0" class="loading">
      <p>Loading...</p>
    </div>
    <div v-else>
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
              :checked="time == timerOptions[selectedTime]"
              @click="changeTime"
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
        <div
          v-for="word in words"
          :key="`${tryNumber}_${word.id}`"
          class="word"
        >
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
        <div v-if="antiAFK" id="anti-afk">
          <p id="heading">It looks like you were AFK.</p>
          <p id="description">Your result has not been saved.</p>
        </div>
        <div v-else id="result">
          <p id="heading">You finished! Your score:</p>
          <p id="result-wpm">{{ resultWPM }} WPM</p>
          <p id="result-accuracy">{{ resultAccuracy }}% accuracy</p>
        </div>
        <p id="play-again">Press anywhere to play again.</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [],
      timerOptions: [15, 30, 60],
      selectedTime: localStorage.getItem("time") || 1,
      currentWordIndex: 0,
      currentLetterIndex: 0,
      correctLetters: 0,
      totalLetters: 0,
      wordsFromNewlineCounter: 0,
      tryNumber: 0,
      displayCaret: true,
      inProgress: false,
      testFinished: false,
      resultWPM: 0,
      resultAccuracy: 0,
      antiAFK: false,
      lastTime: null,
    };
  },

  methods: {
    startTest() {
      if (this.words.length == 0) return;
      this.$refs["typing-input"].focus();
      if (this.inProgress) return;

      this.testFinished = false;
      this.tryNumber++;
      if (this.words.length == 0) this.loadWords();
      this.$nextTick(() => {
        this.moveCaret();
        this.inProgress = true;
        if (this.endTimeout) {
          clearTimeout(this.endTimeout);
          this.endTimeout = undefined;
        }
        this.lastTime = new Date().getTime();
      });
    },

    endTest() {
      if (this.antiAFKInterval) {
        clearInterval(this.antiAFKInterval);
        this.antiAFKInterval = undefined;
      }
      const LETTERS_IN_WORD = 5;
      let words = this.correctLetters / LETTERS_IN_WORD;
      let wpm = words / (this.timerOptions[this.selectedTime] / 60);

      this.resultWPM = Math.floor(wpm);
      if (this.totalLetters > 0)
        this.resultAccuracy = (
          (this.correctLetters / this.totalLetters) *
          100
        ).toFixed(2);
      else this.resultAccuracy = 0;

      const resultWPM = this.resultWPM;
      const resultAccuracy = this.resultAccuracy;
      const selectedTime = this.selectedTime;

      if (wpm < 10) {
        this.antiAFK = true;
        this.testFinished = true;
        return;
      }

      this.testFinished = true;

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
            {
              wpm: resultWPM,
              time: this.timerOptions[selectedTime],
              accuracy: parseFloat(resultAccuracy),
            },
            config
          )
          .catch(() => {});
      }
    },

    playAgain() {
      this.testFinished = false;
      this.$nextTick(() => {
        this.restartTest();
      });
    },

    changeTime(event) {
      let index = this.timerOptions.indexOf(parseInt(event.target.value));
      if (this.selectedTime == index) return;
      this.selectedTime = index;
      localStorage.setItem("time", this.selectedTime);
      this.restartTest();
    },

    checkAntiAfk() {
      const ANTI_AFK_THRESHOLD = 10; // seconds

      if (this.lastTime !== null) {
        if (new Date().getTime() - this.lastTime >= ANTI_AFK_THRESHOLD * 1000) {
          this.antiAFK = true;
          this.endTest();
          return;
        }
      }
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
      if (this.antiAFKInterval) clearInterval(this.antiAFKInterval);
      this.currentWordIndex = 0;
      this.currentLetterIndex = 0;
      this.correctLetters = 0;
      this.totalLetters = 0;
      this.wordsFromNewlineCounter = 0;
      this.inProgress = false;
      this.antiAFK = false;
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
      if (!this.endTimeout)
        this.endTimeout = setTimeout(
          this.endTest,
          this.timerOptions[this.selectedTime] * 1000
        );

      if (!this.antiAFKInterval)
        this.antiAFKInterval = setInterval(this.checkAntiAfk, 1000);

      this.lastTime = new Date().getTime();

      // This is a total mess. There must be a better way, but I can not think of it.
      let word =
        this.$refs["words-container"].querySelectorAll(".word")[
          this.currentWordIndex
        ];
      let letters = word.querySelectorAll(".letter");

      if (e.code == "Space") {
        let isWordCorrect = true;
        letters.forEach((letter) => {
          if (letter.classList.contains("incorrect")) isWordCorrect = false;
          if (
            letter.classList.contains("correct") ||
            letter.classList.contains("incorrect")
          )
            return;
          letter.classList.add("incorrect");
          isWordCorrect = false;
        });
        // AFAIK space is considered a character
        if (isWordCorrect) {
          this.correctLetters++;
          this.totalLetters++;
        }
        this.currentWordIndex++;
        this.currentLetterIndex = 0;
        this.wordsFromNewlineCounter++;
        this.updateTest();
        return;
      } else if (e.code == "Backspace") {
        if (this.currentLetterIndex == 0) {
          let precedingWord =
            this.$refs["words-container"].querySelectorAll(".word")[
              this.currentWordIndex - 1
            ];
          if (precedingWord === undefined) return;
          this.currentWordIndex--;
          this.currentLetterIndex =
            precedingWord.querySelectorAll(".letter").length;
          this.wordsFromNewlineCounter--;
          this.updateTest();
          return;
        }
        let lastLetter = letters[this.currentLetterIndex - 1];
        if (lastLetter.classList.contains("correct")) this.correctLetters--;
        this.totalLetters--;
        this.currentLetterIndex--;
        lastLetter.classList.remove("correct");
        lastLetter.classList.remove("incorrect");
        this.updateTest();
        return;
      }

      if (word === undefined) return;

      let letter = word.querySelectorAll(".letter")[this.currentLetterIndex];
      if (letter === undefined) return;

      let expectedKey = letter.querySelector("span").innerHTML;

      if (e.key == expectedKey) {
        this.correctLetters++;
        letter.classList.add("correct");
      } else letter.classList.add("incorrect");

      this.totalLetters++;
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
  background-color: var(--background-color);
  text-align: justify;
  padding: 0.4em;
}

.words-container {
  &.blur {
    filter: blur(0.3em);
  }
  transition: blur 0.3s ease;
}

.not-focused-message,
.loading {
  position: absolute;
  display: table;
  height: 50vh;
  width: 70vw;
  p {
    display: table-cell;
    text-align: center;
    width: 100%;
    vertical-align: middle;
    color: var(--secondary-color);
    font-size: 2 * $font-size;
    opacity: 0.8;
  }
}

.loading {
  p {
    color: var(--secondary-color);
    animation: blinking 2s infinite;
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
    color: var(--primary-color);
    font-size: $font-size;
    font-family: var(--font-family);
    opacity: 0.4;
  }
  #caret {
    position: absolute;
    height: $font-size * 1.5;
    width: 0.2em;
    background-color: var(--secondary-color);
    opacity: 0.8;
    top: 0;
    animation: blinking 1s infinite;
  }
}

@keyframes blinking {
  50% {
    opacity: 0.2;
  }
}

.fake-letter {
  position: absolute;
}

.correct > span {
  opacity: 0.8;
  color: var(--primary-color);
}

.incorrect > span {
  opacity: 0.5;
  color: var(--error-color);
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
  text-align: right;
}

.timer-option-wrapper {
  display: inline-block;
  label {
    font-family: var(--font-family);
    color: var(--primary-color);
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
  font-family: var(--font-family);
  text-align: center;

  #anti-afk {
    #heading {
      color: var(--error-color);
      font-size: 2.25em;
    }
    #description {
      color: var(--primary-color);
      font-size: 1.25em;
      margin-bottom: 3em;
    }
  }

  #result {
    color: white;
    #heading {
      font-size: 2.5rem;
    }
    #result-wpm {
      font-size: 4rem;
      margin-bottom: 0em;
    }
    #result-accuracy {
      margin-top: 0em;
      margin-bottom: 3em;
    }
  }

  #play-again {
    color: white;
    opacity: 0.7;
  }
}

@media only screen and (min-width: 850px) {
  .results {
    #result {
      #heading {
        font-size: 3rem;
      }
      #result-wpm {
        font-size: 5rem;
      }
    }
  }
}

@media only screen and (min-width: 1300px) {
  .results {
    #anti-afk {
      #heading {
        color: var(--error-color);
        font-size: 3em;
      }
      #description {
        color: var(--primary-color);
        font-size: 2em;
        margin-bottom: 2em;
      }
    }
  }
}
</style>
