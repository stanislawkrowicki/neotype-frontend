<template>
  <div class="main-container">
    <TypingWindow id="typing-window" />
  </div>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
.main-container {
  height: 90vh; // 10vh is taken by the navbar
  background-color: $background-color;
  overflow: hidden;
}

#typing-window {
  display: block;
  margin: 10em auto;
  border-style: ridge;
  border-color: $primary-color;
}
</style>