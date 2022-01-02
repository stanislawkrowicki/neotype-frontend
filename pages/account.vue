<template>
  <main>
    <div v-if="shouldRender" class="info-container">
      <div class="user">
        <img class="user-img" src="~/assets/images/profile.png" />
        <p id="name">{{ name }}</p>
        <p @click="logout" id="logout">Log out</p>
      </div>
      <div class="stats">
        <p>Total tests: {{ tests }}</p>
        <p>All time average: {{ avg.toFixed(2) }} WPM</p>
        <p>Member since: {{ memberSince }}</p>
      </div>
    </div>
    <Results />
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
      shouldRender: false,
    };
  },

  methods: {
    formatDate(date) {
      let dateObj = new Date(date);
      return `${dateObj.getFullYear()}-${dateObj.getMonth()}-${dateObj.getDate()}`;
    },

    logout() {
      localStorage.removeItem("token");
      this.$router.push("/");
    },
  },

  mounted() {
    if (localStorage.getItem("token") === null) {
      this.$router.push("/login");
      return;
    }
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
        this.memberSince = this.formatDate(response.createdAt);
        this.shouldRender = true;
      });
  },
};
</script>

<style scoped lang="scss">
main {
  position: absolute;
  min-height: 90vh;
  min-width: 100%;
  background-color: var(--background-color);
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
    margin-top: 0.9em;
  }
  #name {
    font-size: 3em;
    font-family: var(--font-family);
    color: var(--primary-color);
    margin-bottom: 0em;
    width: 14rem;
  }
  #logout {
    font-size: 2em;
    font-family: var(--font-family);
    color: var(--secondary-color);
    cursor: pointer;

    &:hover {
      font-weight: bold;
    }
  }
}

.stats {
  margin-left: 2em;
  padding-right: 5vw;
  font-size: 2em;
  color: var(--primary-color);
  font-family: var(--font-family);

  p {
    margin-top: 0;
    margin-bottom: 2em;
  }
}
</style>