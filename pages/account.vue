<template>
  <main>
    <div class="info-container">
      <div class="user">
        <div class="user-img"></div>
        <p id="name">{{ name }}</p>
      </div>
      <div class="stats">
        <p>Total tests: {{ tests }}</p>
        <p>All time average: {{ avg.toFixed(2) }} WPM</p>
        <p>Member since: {{ memberSince }}</p>
      </div>
    </div>

    <table v-if="results.length > 0">
      <tr>
        <th>Date</th>
        <th>Result</th>
        <th>Time</th>
      </tr>
      <tr v-for="result in results.slice(0, 10)" :key="result.id">
        <th scope="row">{{ formatDate(result.date) }}</th>
        <td>{{ result.wpm }}</td>
        <td>{{ result.time }}</td>
      </tr>
    </table>
  </main>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      tests: 0,
      avg: 0,
      memberSince: "",
      results: [],
    };
  },

  methods: {
    loadResults() {
      const RESULTS_COUNT = 10;

      let auth = "Bearer " + localStorage.getItem("token");
      let config = {
        headers: {
          Authorization: auth,
        },
      };

      this.$axios
        .$get("/results/" + RESULTS_COUNT, config)
        .then((response) => {
          this.results = response;
        })
        .catch(() => {});
    },

    formatDate(date) {
      let dateObj = new Date(date);
      return `${dateObj.getFullYear()}-${dateObj.getMonth()}-${dateObj.getDate()}`;
    },
  },

  mounted() {
    if (localStorage.getItem("token") === null) this.$router.push("/login");

    let auth = "Bearer " + localStorage.getItem("token");
    let config = {
      headers: {
        Authorization: auth,
      },
    };

    this.$axios
      .$get("/data", config)
      .catch((error) => {
        if (!error.response) this.$router.push("/");
        if (error.response.status == 401) {
          // Unauthorized
          localStorage.removeItem("token");
          this.$router.push("/login");
        } else this.$router.push("/");
      })
      .then((response) => {
        if (response === undefined) this.$router.push("/");
        this.name = response.login;
        this.tests = response.tests;
        this.avg = response.avg;
        this.memberSince = "todo";
      });

    this.loadResults();
  },
};
</script>

<style scoped lang="scss">
main {
  position: absolute;
  min-height: 90vh;
  min-width: 100%;
  background-color: $background-color;
}

.info-container {
  margin-top: 5em;
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-left: 5vw;

  @media only screen and (min-width: 680px) {
    align-items: flex-start;
    margin-left: 10vw;
  }
}

.user {
  .user-img {
    height: 14em;
    width: 14em;
    background-color: #f0f;
    margin-top: 0.9em;
  }
  #name {
    font-size: 3em;
    font-family: $font-family;
    color: $primary-color;
  }
}

.stats {
  margin-left: 2em;
  padding-right: 5vw;
  font-size: 2em;
  color: $primary-color;
  font-family: $font-family;

  p {
    margin-top: 0;
    margin-bottom: 2em;
  }
}

table {
  table-layout: fixed;
  width: 50%;
  border-collapse: collapse;
  border: 2px solid $secondary-color;
  margin: auto;
  margin-top: 2em;
  margin-bottom: 4em;
}

th,
td {
  padding: 1.5em;
  color: $primary-color;
  font-family: $font-family;
  text-align: center;
  white-space: nowrap;
}
</style>