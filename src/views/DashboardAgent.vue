<template>
  <div class="dashboard">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo Robust Code" class="logo" />
        <h1>Tableau de bord Agent</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/demandes" class="nav-btn">Toutes les demandes</router-link>
        <button @click="logout" class="logout-btn">Déconnexion</button>
      </div>
    </header>

    <!-- Statistiques -->
    <section class="stats">
      <div class="stat-card">
        <h2>{{ stats.demandesSoumises }}</h2>
        <p>Demandes à traiter</p>
      </div>
      <div class="stat-card">
        <h2>{{ stats.demandesEnTraitement }}</h2>
        <p>Demandes en traitement</p>
      </div>
      <div class="stat-card">
        <h2>{{ stats.demandesValidees }}</h2>
        <p>Demandes validées</p>
      </div>
    </section>

    <!-- Liste des dernières demandes -->
    <section class="demandes">
      <h3>Dernières demandes</h3>
      <table>
        <thead>
          <tr>
            <th>Type</th>
            <th>Citoyen</th>
            <th>Statut</th>
            <th>Date</th>
            <th>Détail</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="demande in demandes" :key="demande.id">
            <td>{{ getTypeDemandeLabel(demande.typeDemande) }}</td>
            <td>{{ formatNomComplet(demande.citoyen) }}</td>
            <td>{{ getStatutLabel(demande.statut?.nom) }}</td>
            <td>{{ formatDate(demande.createdAt) }}</td>
            <td>
              <button @click="goToDetail(demande.id)" class="detail-btn">Voir</button>
            </td>
          </tr>
          <tr v-if="demandes.length === 0">
            <td colspan="5" class="empty">Aucune demande disponible</td>
          </tr>
        </tbody>
      </table>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DashboardAgent",
  data() {
    return {
      stats: {
        demandesSoumises: 0,
        demandesEnTraitement: 0,
        demandesValidees: 0,
      },
      demandes: [],
      selectedDemandeId: null,
    };
  },
  methods: {
    async fetchStats() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "https://be-solution-backend.onrender.com/api/agents/dashboard",
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        this.stats = res.data;
      } catch (err) {
        console.error("Erreur chargement stats agent", err);
      }
    },
    async fetchDemandes() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "https://be-solution-backend.onrender.com/api/agents/assigned-demandes",
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        this.demandes = res.data;
      } catch (err) {
        console.error("Erreur chargement demandes", err);
      }
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push("/login");
    },
    formatNomComplet(person) {
      if (!person) return "-";
      return [person.nom, person.prenom, person.postnom].filter(Boolean).join(" ");
    },
    getTypeDemandeLabel(type) {
      const mapping = {
        carte_identite: "Carte d'identité",
        acte_naissance: "Acte de naissance",
        acte_mariage: "Acte de mariage",
        acte_residence: "Acte de résidence",
      };
      return mapping[type] || type;
    },
    getStatutLabel(statut) {
      const mapping = {
        soumise: "Soumise",
        en_traitement: "En traitement",
        validee: "Validée",
      };
      return mapping[statut] || statut;
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString("fr-FR");
    },
    goToDetail(demandeId) {
      this.selectedDemandeId = demandeId;
      this.$router.push(`/demandes/${demandeId}`);
    },
  },
  async mounted() {
    await this.fetchStats();
    await this.fetchDemandes();
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

/* NAVBAR */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #0E2C5A;
  padding: 14px 24px;
  color: white;
  font-family: 'Ysabeau Office', sans-serif;
}
.navbar-left {
  display: flex;
  align-items: center;
  gap: 10px;
}
.logo {
  height: 42px;
}
.navbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
}
.nav-btn {
  background: white;
  color: #0E2C5A;
  padding: 8px 14px;
  border-radius: 8px;
  font-weight: bold;
  text-decoration: none;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: background 0.3s;
}
.nav-btn:hover {
  background: #f1f1f1;
}
.logout-btn {
  background: white;
  color: #0E2C5A;
  padding: 8px 14px;
  border-radius: 8px;
  font-weight: bold;
  border: none;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
}
.logout-btn:hover {
  background: #f1f1f1;
}

/* STATS */
.stats {
  display: flex;
  gap: 20px;
  margin: 30px 20px;
  flex-wrap: wrap;
}
.stat-card {
  background: white;
  padding: 24px;
  border-radius: 14px;
  flex: 1;
  text-align: center;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.08);
}
.stat-card h2 {
  font-size: 30px;
  color: #0E2C5A;
  font-family: 'Ysabeau Office', sans-serif;
}
.stat-card p {
  font-size: 15px;
  color: #666;
  font-family: 'ABeeZee', sans-serif;
}

/* TABLE */
.demandes {
  margin: 30px 20px;
}
.demandes h3 {
  color: #0E2C5A;
  font-family: 'Ysabeau Office', sans-serif;
  margin-bottom: 14px;
}
table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}
th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #eee;
  font-family: 'ABeeZee', sans-serif;
}
th {
  background: #0E2C5A;
  color: white;
  font-family: 'Inter', sans-serif;
}
tr:nth-child(even) {
  background: #f9f9f9;
}

/* BOUTONS */
.detail-btn {
  background: #104B71;
  color: white;
  padding: 6px 12px;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  font-weight: bold;
  transition: background 0.3s;
}
.detail-btn:hover {
  background: #0E2C5A;
}

/* MESSAGE */
.empty {
  text-align: center;
  padding: 20px;
  font-style: italic;
  color: #666;
}
</style>