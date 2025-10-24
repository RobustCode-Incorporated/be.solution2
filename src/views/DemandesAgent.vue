<template>
  <div class="page-demandes-agent">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo BE Solution" class="logo" />
        <h1>Demandes à traiter</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/dashboard" class="nav-btn">Tableau de bord</router-link>
        <button @click="logout" class="logout-btn">Déconnexion</button>
      </div>
    </header>

    <!-- Filtres -->
    <section class="controls">
      <input v-model="q" @input="applyFilters" placeholder="Rechercher par citoyen, type..." />
      <select v-model="filterStatut" @change="applyFilters">
        <option value="">Tous les statuts</option>
        <option v-for="s in statuts" :key="s.id" :value="s.id">{{ s.nom }}</option>
      </select>
      <button @click="fetchDemandes" class="refresh-btn">Rafraîchir</button>
    </section>

    <section v-if="loading" class="loading">Chargement...</section>

    <!-- Table des demandes -->
    <section v-else>
      <table class="table-demandes">
        <thead>
          <tr>
            <th>Type</th>
            <th>Citoyen</th>
            <th>Statut</th>
            <th>Date</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="d in pagedDemandes" :key="d.id">
            <td>{{ typeLabel(d.typeDemande) }}</td>
            <td>{{ fullname(d.citoyen || {}) }}</td>
            <td>{{ d.statut && d.statut.nom ? d.statut.nom : '—' }}</td>
            <td>{{ formatDate(d.createdAt) }}</td>
            <td>
              <button @click="openDetails(d.id)">Voir / Traiter</button>
            </td>
          </tr>
          <tr v-if="!demandes.length">
            <td colspan="5" class="empty">Aucune demande trouvée</td>
          </tr>
        </tbody>
      </table>

      <!-- Pagination -->
      <div class="pagination" v-if="pages > 1">
        <button :disabled="page === 1" @click="page-- && applyFilters()">←</button>
        <span>Page {{ page }} / {{ pages }}</span>
        <button :disabled="page === pages" @click="page++ && applyFilters()">→</button>
      </div>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DemandesAgent",
  data() {
    return {
      loading: true,
      demandes: [],
      statuts: [],
      q: "",
      filterStatut: "",
      page: 1,
      perPage: 10,
    };
  },
  computed: {
    filteredDemandes() {
      let arr = this.demandes.slice();

      if (this.filterStatut) {
        arr = arr.filter(d => String(d.statutId) === String(this.filterStatut));
      }

      if (this.q && this.q.trim()) {
        const q = this.q.toLowerCase();
        arr = arr.filter(d =>
          (d.citoyen && (
            (d.citoyen.nom || "").toLowerCase().includes(q) ||
            (d.citoyen.prenom || "").toLowerCase().includes(q)
          )) ||
          (d.typeDemande || "").toLowerCase().includes(q) ||
          (d.commentaires || "").toLowerCase().includes(q)
        );
      }

      return arr;
    },
    pages() {
      return Math.max(1, Math.ceil(this.filteredDemandes.length / this.perPage));
    },
    pagedDemandes() {
      const start = (this.page - 1) * this.perPage;
      return this.filteredDemandes.slice(start, start + this.perPage);
    }
  },
  methods: {
    formatDate(d) {
      return d ? new Date(d).toLocaleString("fr-FR") : "-";
    },
    fullname(person) {
      if (!person) return "-";
      return [person.nom || "", person.prenom || "", person.postnom || ""].filter(Boolean).join(" ");
    },
    typeLabel(t) {
      const map = {
        carte_identite: "Carte d'identité",
        acte_naissance: "Acte de naissance",
        acte_mariage: "Acte de mariage",
        acte_residence: "Acte de résidence"
      };
      return map[t] || t;
    },
    async fetchStatuts() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get("https://be-solution-backend.onrender.com/api/statuts", {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.statuts = res.data;
      } catch (err) {
        console.error("Erreur statuts:", err);
      }
    },
    async fetchDemandes() {
      this.loading = true;
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get("https://be-solution-backend.onrender.com/api/agents/assigned-demandes", {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.demandes = res.data.filter(d => d && d.id);
        this.page = 1;
      } catch (err) {
        console.error("Erreur chargement demandes:", err);
        alert("Erreur chargement des demandes.");
      } finally {
        this.loading = false;
      }
    },
    applyFilters() {
      if (this.page > this.pages) this.page = this.pages;
    },
    openDetails(id) {
      if (!id) {
        alert("Impossible d’ouvrir le détail : ID invalide.");
        return;
      }
      this.$router.push({ name: "DemandeDetailsAgent", params: { id } });
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push({ name: "LoginAgent" });
    }
  },
  async mounted() {
    await Promise.all([this.fetchStatuts(), this.fetchDemandes()]);
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

.page-demandes-agent {
  font-family: "Inter", sans-serif;
  padding: 24px;
  background: #f9f9f9;
}

/* NAVBAR */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #0E2C5A;
  padding: 14px 24px;
  color: white;
  font-family: 'Ysabeau Office', sans-serif;
  border-radius: 10px;
}
.navbar-left {
  display: flex;
  align-items: center;
  gap: 10px;
}
.logo {
  height: 40px;
  display: block;
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
  transition: background 0.3s;
}
.nav-btn:hover {
  background: #f1f1f1;
}
.logout-btn {
  background: #d9534f;
  border: none;
  color: #fff;
  padding: 8px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: background 0.3s ease;
}
.logout-btn:hover {
  background: #b52b27;
}

/* CONTROLS */
.controls {
  margin: 20px 0;
  display: flex;
  gap: 12px;
  align-items: center;
  flex-wrap: wrap;
}
input,
select {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-family: "ABeeZee", sans-serif;
}
.refresh-btn {
  background: #104B71;
  color: white;
  border: none;
  padding: 8px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
}
.refresh-btn:hover {
  background: #0E2C5A;
}

/* TABLE */
.table-demandes {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}
.table-demandes th,
.table-demandes td {
  border: 1px solid #e3e3e3;
  padding: 10px;
  text-align: left;
}
.table-demandes th {
  background: #0E2C5A;
  color: white;
}
button {
  background: #104B71;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}
button:hover {
  background: #0E2C5A;
}
.empty {
  text-align: center;
  color: #666;
  font-style: italic;
}

/* PAGINATION */
.pagination {
  margin-top: 16px;
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: center;
}
.pagination button {
  background: #104B71;
  color: white;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
}
.pagination button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>