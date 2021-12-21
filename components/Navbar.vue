<template>
  <nav>
    <div class="hamburger-menu" ref="hamburger-menu">
      <div
        class="hamburger-menu-btn"
        ref="hamburger-menu-btn"
        @click="hamburgerMenu"
      >
        <svg viewBox="0 0 100 80" width="40" height="40">
          <rect width="100" height="15"></rect>
          <rect y="30" width="100" height="15"></rect>
          <rect y="60" width="100" height="15"></rect>
        </svg>
      </div>

      <ul class="menu" ref="hamburger-menu-main">
        <li><NuxtLink to="/">Home</NuxtLink></li>
        <li><NuxtLink to="/">Leaderboards</NuxtLink></li>
        <li><NuxtLink to="/">About</NuxtLink></li>
        <li><NuxtLink to="/account">Account</NuxtLink></li>
        <li><NuxtLink to="/">Settings</NuxtLink></li>
      </ul>
    </div>

    <div class="links">
      <NuxtLink to="/">Home</NuxtLink>
      <NuxtLink to="/">Leaderboards</NuxtLink>
      <NuxtLink to="/">About</NuxtLink>
      <NuxtLink to="/account">Account</NuxtLink>
      <NuxtLink to="/">Settings</NuxtLink>
    </div>
  </nav>
</template>

<script>
export default {
  methods: {
    hamburgerMenu() {
      const menu = this.$refs["hamburger-menu-main"];
      if (menu.style.display == "none") menu.style.display = "block";
      else menu.style.display = "none";
    },
  },
};
</script>

<style lang="scss" scoped>
nav {
  position: relative;
  width: 100%;
  height: 9.75vh;
  background-color: $background-color;
  border-bottom: 0.25vh solid $secondary-color;

  .hamburger-menu {
    position: absolute;
    right: 2%;
    top: 50%;

    .hamburger-menu-btn {
      float: right;
      fill: $primary-color;
      cursor: pointer;
    }

    .menu {
      display: none;
      margin-top: 3em;
      text-align: right;
      list-style: none;

      li > * {
        text-decoration: none;
        color: $primary-color;
        font-family: $font-family;
        cursor: pointer;

        &:hover {
          font-weight: bold;
        }
      }
    }
  }

  .links {
    display: none;
  }

  @media only screen and (min-width: 680px) {
    .links {
      display: inline-block;
      margin: 0;
      position: absolute;
      top: 50%;
      right: 0%;
      transform: translateY(-50%);
      cursor: pointer;
      * {
        text-decoration: none;
        color: $primary-color;
        font-family: $font-family;
        margin: 2em;

        &:hover {
          font-weight: bold;
        }
      }
    }

    .hamburger-menu {
      display: none;
    }
  }
}
</style>