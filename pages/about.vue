<template>
  <div class="container">
    <header>Neotype</header>
    <p>Neotype is a minimalistic website for testing your typing speed.</p>
    <p>
      The project is heavily inspired by
      <a href="https://monkeytype.com/">Monkeytype</a> and
      <a href="https://play.typeracer.com/">typeracer</a>
    </p>
    <p>
      It was made solely for purpose of learning. If you came here by accident,
      I heavily suggest you to visit the websites mentioned above.
    </p>
    <p id="find">You can find me here:</p>
    <div class="images">
      <a href="https://discordapp.com/users/396680392134098954"
        ><img src="~/assets/images/discord.png" alt="Discord"
      /></a>
      <a href="https://github.com/stanislawkrowicki"
        ><img src="~/assets/images/github.png" alt="Github"
      /></a>
    </div>
  </div>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
.container {
  height: 90vh;
  width: 100vw;
  background-color: $background-color;
}

header,
p {
  font-family: $font-family;
  text-align: center;
}

header {
  color: $secondary-color;
  font-size: 6em;
  margin-bottom: 1em;
}

p {
  color: $primary-color;
  font-size: 2em;
}

a {
  color: $primary-color;
}

.images {
  display: flex;
  flex-direction: row;
  justify-content: center;

  * {
    margin-right: 1em;
    width: 3em;
    height: 3em;
  }
}
</style>