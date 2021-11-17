<template>
  <div class="main-window" @click="startTest">
    <div class="words-container">
      <input type="text" id="words-input" ref="typing-input" v-on:keydown="keyPressed" autocomplete="off" autocapitalize="off" autocorrect="off">
      <div v-for="word in words" :key="word" ref="words" class="word">
        <div v-for="letter in word" :key="letter" ref="letters" class="letter">{{ letter }}</div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        words: ['age', 'eyes', 'wildblue', 'jingle', 'blues', 'californication', 'wet', 'sand', 'space', 'font', 'height', 'dependency'],
        currentWordIndex: 0,
        currentLetterIndex: 0,
        correctLetters: 0
      }
    },

    methods: {
      startTest() {
        this.$refs['typing-input'].focus()
      },

      keyPressed(e) {
        if (e.code == 'Space') {
          this.currentWordIndex++
          this.currentLetterIndex = 0
          return
        }

        let word = this.$refs.words[this.currentWordIndex]
        if (word === undefined)
          return

        let letter = word.children[this.currentLetterIndex]   
        if (letter === undefined)
          return

        let expectedKey = letter.innerHTML

        if (e.key == expectedKey) {
          this.correctLetters++
          letter.className = `${letter.className} correct`
        } else 
          letter.className = `${letter.className} incorrect`

        this.currentLetterIndex++;
      }
    },

  }
</script>

<style lang="scss" scoped>
  .main-window {
      width: 80em;
      height: 30em;
      background-color: $background-color;
      text-align: justify;
      padding: .4em;
  }

  .word {
    display: inline-block;
    margin-right: 1em;
  }

  .letter {
      display: inline-block;
      color: $primary-color;
      font-size: 4em;
      font-family: 'Shippori Antique', sans-serif;
      opacity: 0.4;
  }

  .correct {
    opacity: 0.8;
    color: $primary-color;
  }

  .incorrect {
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
