<template>
  <div class="container">
    <header>Theme</header>
    <ThemeSelector id="selector" />
  </div>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
.container {
  position: absolute;
  min-height: 90vh;
  min-width: 100%;
  background-color: var(--background-color);
}

#selector {
  margin: auto;
  width: 40vw;
  height: 30vh;
}

header {
  text-align: center;
  font-family: var(--font-family);
  color: var(--primary-color);
  font-size: 4rem;
  margin-bottom: 2rem;
}
</style>