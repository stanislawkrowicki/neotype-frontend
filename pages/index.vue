<template>
  <div class="main-container">
    <TypingWindow id="typing-window" />
    <Leaderboard id="leaderboard" />
  </div>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
.main-container {
  position: absolute;
  min-height: 90vh; // 10vh for navbar
  min-width: 100%;
  background-color: var(--background-color);
}

#typing-window {
  display: block;
  margin: 7.5em auto;
  border-style: ridge;
  border-color: var(--primary-color);
}

#leaderboard {
  margin-top: 30vh;
}
</style>