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

      let elems = this.querySelectorAll(
        ".container, nav, .nav-link, header, .main-container"
      );
      elems.forEach((elem) => {
        elem.style.transition = "none";
      });

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

      elems.forEach((elem) => {
        elem.style.transition = "all 750ms linear";
      });
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
.main-container {
  transition: all 750ms linear;
}
</style>