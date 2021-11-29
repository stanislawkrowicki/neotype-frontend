<template>
  <nav>
    <div class="links">
      <NuxtLink to="/">Home</NuxtLink>
      <NuxtLink to="/">Leaderboards</NuxtLink>
      <NuxtLink to="/">About</NuxtLink>
      <NuxtLink to="/">Account</NuxtLink>
      <NuxtLink to="/">Settings</NuxtLink>
    </div>
  </nav>
</template>

<script>
export default {};
</script>

<style lang="scss" scoped>
nav {
  position: relative;
  width: 100%;
  height: 10vh;
  background-color: $background-color;

  .links {
    margin: 0;
    position: absolute;
    top: 50%;
    right: 0%;
    transform: translateY(-50%);
    cursor: pointer;
    * {
      text-decoration: none;
      color: $primary-color;
      font-family: "Shippori Antique", sans-serif;
      margin: 2em;
    }
  }
}
</style>