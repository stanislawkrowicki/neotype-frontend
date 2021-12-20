<template>
  <div class="container"><Login id="login" /></div>
</template>

<script>
export default {};
</script>

<style scoped lang="scss">
.container {
  position: absolute;
  top: 10vh;
  height: 90vh;
  width: 100vw;
  background-color: $background-color !important;

  #login {
    position: relative;
    margin: auto;
    margin-top: 5em;
  }
}
</style>