<template>
  <div class="register">
    <p class="heading">Sign up</p>
    <div class="inputs">
      <input
        type="text"
        class="login-input"
        placeholder="Login"
        v-model="promptedLogin"
        @keyup.enter="register"
        required
      />
      <input
        type="password"
        class="password-input"
        placeholder="Password"
        v-model="promptedPassword"
        @keyup.enter="register"
        required
      />
      <input
        type="password"
        class="password-input-repeat"
        placeholder="Repeat password"
        v-model="promptedPasswordRepeat"
        @keyup.enter="register"
        required
      />
    </div>
    <span class="message">{{ message }}</span>
    <button class="submit-btn" @click="register">Sign in</button>
    <span id="redirect"
      >Already have an account? <NuxtLink to="/login">Sign in</NuxtLink></span
    >
  </div>
</template>

<script>
export default {
  data() {
    return {
      promptedLogin: "",
      promptedPassword: "",
      promptedPasswordRepeat: "",
      message: "",
    };
  },

  methods: {
    async register() {
      if (this.promptedPassword != this.promptedPasswordRepeat) {
        // this should be done server side as well
        this.message = "Passwords do not match";
        return;
      }
      this.$axios
        .$post("/register", {
          login: this.promptedLogin,
          password: this.promptedPassword,
        })
        .catch((error) => {
          this.message = error.response.data.message;
        })
        .then((response) => {
          if (response) this.$router.push("/login");
        });
    },
  },
};
</script>

<style scoped lang="scss">
.register {
  display: flex;
  background-color: var(--background-color);
  width: 60vh;
  margin-left: auto;
  margin-right: auto;
  gap: 1em;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  flex-flow: column wrap;
}

.heading {
  color: var(--primary-color);
  opacity: 0.8;
  font-family: var(--font-family);
  font-size: 4em;
  margin-bottom: 0em;
  margin-top: 0em;
}

.inputs {
  margin-top: 2em;
}

input {
  display: block;
  margin-bottom: 3em;
  padding: 1em;
  background-color: var(--background-color);
  box-sizing: border-box;
  border: none;
  border-bottom: 2px solid var(--secondary-color);
  color: var(--primary-color);
  outline: none;
}

input:focus {
  border-bottom: 2px solid var(--primary-color);
}

.message {
  font-family: var(--font-family);
  color: var(--primary-color);
  margin-bottom: 1em;
}

.submit-btn {
  margin-bottom: 3em;
  width: 6em;
  height: 4em;
  background-color: var(--background-color);
  border: 1px solid var(--primary-color);
  color: var(--primary-color);
  transition-duration: 0.3s;
  border-radius: 0.5em;
  cursor: pointer;
}

.submit-btn:hover {
  border-color: var(--background-color);
  background-color: var(--primary-color);
  color: var(--background-color);
}

#redirect {
  color: var(--primary-color);
  font-family: var(--font-family);

  * {
    text-decoration: none;
    color: var(--secondary-color);
    font-weight: bold;
  }
}
</style>