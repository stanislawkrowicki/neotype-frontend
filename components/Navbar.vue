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
        <li><NuxtLink to="/#leaderboard">Leaderboards</NuxtLink></li>
        <li><NuxtLink to="/about">About</NuxtLink></li>
        <li><NuxtLink to="/">Settings</NuxtLink></li>
        <li>
          <NuxtLink to="/account">{{ username }}</NuxtLink>
        </li>
      </ul>
    </div>

    <div class="links">
      <NuxtLink to="/">Home</NuxtLink>
      <NuxtLink to="/#leaderboard">Leaderboards</NuxtLink>
      <NuxtLink to="/about">About</NuxtLink>
      <NuxtLink to="/">Settings</NuxtLink>
      <NuxtLink to="/account">{{ username || "Guest" }}</NuxtLink>
    </div>
  </nav>
</template>

<script>
export default {
  data() {
    return {
      username: null,
    };
  },

  methods: {
    hamburgerMenu() {
      const menu = this.$refs["hamburger-menu-main"];
      if (menu.style.display == "none") menu.style.display = "block";
      else menu.style.display = "none";
    },

    getUsername() {
      if (localStorage.getItem("token") === null) {
        this.username = null;
        return;
      }

      let auth = "Bearer " + localStorage.getItem("token");
      let config = {
        headers: {
          Authorization: auth,
        },
      };

      this.$axios
        .$get("/username", config)
        .catch((error) => {
          if (error.response.status == 401)
            //Unauthorized
            localStorage.removeItem("token");
        })
        .then((response) => {
          if (response && response.username) this.username = response.username;
        });
    },
  },

  mounted() {
    this.getUsername();
  },

  watch: {
    $route() {
      this.getUsername();
    },
  },
};
</script>

<style lang="scss" scoped>
nav {
  position: relative;
  width: 100%;
  height: 9.75vh;
  background-color: var(--background-color);
  border-bottom: 0.25vh solid var(--secondary-color);

  .hamburger-menu {
    position: absolute;
    right: 2%;
    top: 50%;

    .hamburger-menu-btn {
      float: right;
      fill: var(--primary-color);
      cursor: pointer;
    }

    .menu {
      display: none;
      margin-top: 3em;
      text-align: right;
      list-style: none;

      li > * {
        text-decoration: none;
        color: var(--primary-color);
        font-family: var(--font-family);
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
        color: var(--primary-color);
        font-family: var(--font-family);
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