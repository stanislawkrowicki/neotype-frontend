<template>
  <div class="selectors">
    <button
      v-for="theme in themes"
      :key="theme[0]"
      @click="selectTheme(theme[0])"
      :style="`color:${theme[2]};background-color:${theme[1]};border:1px solid ${theme[2]}`"
      @mouseover="onThemeHover($event, theme)"
      @mouseleave="onThemeLeave($event, theme)"
    >
      {{ theme[0] }}
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      themes: [
        ["blue light", "hsl(47, 39%, 86%)", "hsl(201, 34%, 70%)"],
        ["blueberry", "hsl(222, 33%, 19%)", "hsl(209, 100%, 84%)"],
        ["dev", "hsl(217, 19%, 13%)", "hsl(195, 72%, 49%)"],
        ["gruvbox dark", "hsl(0, 0%, 16%)", "hsl(40, 73%, 49%)"],
        ["gruvbox light", "hsl(48, 87%, 88%)", "hsl(122, 21%, 51%)"],
        ["lemon", "hsl(213, 26%, 29%)", "hsl(66, 62%, 51%)"],
        ["mint", "hsl(220, 3%, 18%)", "hsl(174, 58%, 68%)"],
        ["olivia", "hsla(0, 0%, 14%, 1)", "hsla(15, 67%, 84%, 1)"],
        ["one dark", "hsl(220, 13%, 18%)", "hsl(286, 60%, 67%)"],
        ["pinky", "hsl(236, 33%, 16%)", "hsl(330, 93%, 82%)"],
        ["purple haze", "hsl(249, 60%, 24%)", "hsl(41, 81%, 73%)"],
        ["solarized", "hsl(184, 89%, 25%)", "hsl(183, 58%, 70%)"],
      ],
    };
  },

  methods: {
    loadTheme(theme) {
      const THEME_ID = "theme";

      let parsedTheme = theme.replace(" ", "_");
      let oldThemeLink = document.getElementById(THEME_ID);

      let head = document.getElementsByTagName("head")[0];
      let link = document.createElement("link");
      link.id = THEME_ID;
      link.rel = "stylesheet";
      link.type = "text/css";
      link.href = `themes/${parsedTheme}.css`;
      link.media = "all";
      if (oldThemeLink) oldThemeLink.replaceWith(link);
      else head.appendChild(link);
    },

    selectTheme(theme) {
      this.loadTheme(theme);
      localStorage.setItem("theme", theme);
    },

    testTheme(theme) {
      this.loadTheme(theme);
    },

    disableTesting() {
      this.loadTheme(localStorage.getItem("theme") || "olivia");
    },

    onThemeHover(event, theme) {
      this.testTheme(theme[0]);

      let btn = event.target;
      btn.style.backgroundColor = theme[2];
      btn.style.color = theme[1];
    },

    onThemeLeave(event, theme) {
      this.disableTesting();

      let btn = event.target;
      btn.style.backgroundColor = theme[1];
      btn.style.color = theme[2];
    },
  },
};
</script>

<style lang="scss" scoped>
.selectors {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  box-sizing: border-box;
  flex-wrap: wrap;
}

button {
  margin: 1em;
  width: 9rem;
  height: 3rem;
  border-radius: 1rem;
  background-color: var(--background-color);
  font-family: var(--font-family);
  cursor: pointer;
  transition-duration: 0.3s;
}
</style>