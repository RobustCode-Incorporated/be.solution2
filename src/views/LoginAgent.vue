<template>
  <div class="login-container">
    <!-- Bloc gauche -->
    <div class="left-panel">
      <img src="../assets/RobustCodelogowhite.png" alt="Logo Robust Code" class="logo" />
      <h1 class="title">ROBUST ADMINISTRATIVE SYSTEM</h1>
    </div>

    <!-- Bloc droit -->
    <div class="right-panel">
      <h1 class="form-title">Connexion Agent</h1>
      <form class="login-form" @submit.prevent="login">
        <label for="username">Nom d'utilisateur</label>
        <input
          type="text"
          id="username"
          v-model="username"
          placeholder="Nom d'utilisateur"
          required
        />

        <label for="password">Mot de passe</label>
        <input
          type="password"
          id="password"
          v-model="password"
          placeholder="Mot de passe"
          required
        />

        <button type="submit" :disabled="loading">
          {{ loading ? "Connexion..." : "SE CONNECTER" }}
        </button>
      </form>

      <p v-if="error" class="error">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "LoginAgent",
  data() {
    return {
      username: "",
      password: "",
      error: "",
      loading: false,
    };
  },
  methods: {
    async login() {
      this.loading = true;
      this.error = "";
      try {
        const res = await axios.post("http://localhost:4001/api/agents/login", {
          username: this.username,
          password: this.password,
        });

        // Sauvegarde du token
        localStorage.setItem("token", res.data.token);

        // Redirection vers dashboard
        this.$router.push("/dashboard");
      } catch (err) {
        this.error = err.response?.data?.message || "Erreur de connexion";
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

.login-container {
  display: flex;
  height: 100vh;
  font-family: 'ABeeZee', sans-serif;
}

/* Bloc gauche (branding) */
.left-panel {
  width: 704px;
  height: 100vh;
  background-color: #0E2C5A;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.logo {
  width: 320px;
  margin-bottom: 30px;
}

.title {
  font-family: 'Ysabeau Office', sans-serif;
  font-size: 1.5rem;
  letter-spacing: 1px;
}

/* Bloc droit (formulaire) */
.right-panel {
  flex: 1;
  background: #fff;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 0 80px;
}

.form-title {
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 24px;
  color: #0E2C5A;
}

.login-form label {
  font-weight: 600;
  margin-bottom: 5px;
  display: block;
}

.login-form input {
  width: 437px;
  height: 50px;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-family: 'ABeeZee', sans-serif;
}

.login-form button {
  background-color: #104B71;
  color: white;
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  border: none;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  width: 437px;
  height: 70px;
  font-weight: bold;
  letter-spacing: 1px;
}

.login-form button:disabled {
  background-color: gray;
  cursor: not-allowed;
}

.login-form button:hover:not(:disabled) {
  background-color: #0d3c5a;
}

.error {
  color: red;
  margin-top: 15px;
  text-align: center;
  font-weight: bold;
}
</style>