<template>
  <div class="container">
    <header>Leaderboard</header>
    <table v-if="leaders.length > 0">
      <tr>
        <th>User</th>
        <th>WPM</th>
        <th>Accuracy</th>
        <th>Time</th>
      </tr>
      <tr v-for="leader in leaders" :key="leader.username">
        <th>{{ leader.username }}</th>
        <th>{{ leader.wpm }}</th>
        <th>{{ parseFloat(leader.accuracy).toFixed(2) }}%</th>
        <th>{{ leader.time }}s</th>
      </tr>
    </table>

    <div v-else class="no-leaders">
      <p>Hmmm, it looks like there are no recent results yet. 🙁</p>
      <p>Why don't you start the party? 😀</p>
    </div>

    <footer>
      <p>Leaderboard is cleaned every 72 hours.</p>
    </footer>
  </div>
</template>

<script>
export default {
  data() {
    return {
      leaders: [],
    };
  },

  methods: {
    getLeaders() {
      const LEADERS_TO_FETCH = 10;

      this.$axios
        .get("/leaderboards/" + LEADERS_TO_FETCH)
        .catch(() => {})
        .then((response) => {
          if (response && response.data) {
            this.leaders = response.data;
          }
        });
    },
  },

  mounted() {
    this.getLeaders();
  },
};
</script>

<style lang="scss" scoped>
header {
  color: var(--primary-color);
  font-family: var(--font-family);
  text-align: center;
  font-size: 4em;
}

table {
  table-layout: fixed;
  width: 50%;
  border-collapse: collapse;
  border: 2px solid var(--secondary-color);
  margin: auto;
  margin-top: 2em;
  margin-bottom: 4em;
}

th,
td {
  padding: 1.5em;
  color: var(--primary-color);
  font-family: var(--font-family);
  text-align: center;
  white-space: nowrap;
}

.no-leaders {
  p {
    color: var(--primary-color);
    font-family: var(--font-family);
    text-align: center;
  }
}

footer {
  p {
    color: var(--primary-color);
    font-family: var(--font-family);
    text-align: center;
  }
}
</style>