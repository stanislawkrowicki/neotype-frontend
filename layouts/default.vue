<template>
  <div v-if="render">
    <Navbar />
    <Nuxt />
  </div>
</template>

<script>
export default {
  data() {
    return {
      render: false,
    };
  },

  methods: {
    loadTheme() {
      const THEME_ID = "theme";
      const DEFAULT_THEME = "olivia";

      let theme = localStorage.getItem("theme") || DEFAULT_THEME;
      let parsedTheme = theme.replace(" ", "_");

      if (!document.getElementById(THEME_ID)) {
        let head = document.getElementsByTagName("head")[0];
        let link = document.createElement("link");
        link.id = THEME_ID;
        link.rel = "stylesheet";
        link.type = "text/css";
        link.href = `themes/${parsedTheme}.css`;
        link.media = "all";
        head.appendChild(link);
      }

      this.render = true;
    },
  },

  mounted() {
    this.loadTheme();
  },
};
</script>

<style lang="scss">
body {
  margin: 0 !important;
}

.container,
nav,
.nav-link,
header,
.main-container,
#typing-window,
.main-window {
  transition: color 750ms linear, background-color 750ms linear,
    border-color 750ms linear;
}
</style>