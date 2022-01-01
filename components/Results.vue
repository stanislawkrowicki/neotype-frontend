<template>
  <table v-if="results.length > 0">
    <tr>
      <th>Date</th>
      <th>Result</th>
      <th>Accuracy</th>
      <th>Time</th>
    </tr>
    <tr v-for="result in results.slice(0, 10)" :key="result.id">
      <th scope="row">{{ formatDate(result.date) }}</th>
      <td>{{ result.wpm }}</td>
      <td>{{ result.accuracy }}%</td>
      <td>{{ result.time }}</td>
    </tr>
  </table>
</template>

<script>
export default {
  data() {
    return {
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
    this.loadResults();
  },
};
</script>

<style lang="scss" scoped>
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