<template>
  <div class="page-demande-details">
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo BE Solution" class="logo" />
        <h1>Détail de la demande</h1>
      </div>
      <div class="navbar-right">
        <router-link :to="{ name: 'DemandesAgent' }" class="nav-btn">← Retour</router-link>
      </div>
    </header>

    <section v-if="loading" class="loading">Chargement...</section>

    <section v-else class="card">
      <h2>{{ typeLabel(demande.typeDemande) }}</h2>

      <div class="row"><strong>Citoyen :</strong> {{ fullname(demande.citoyen) }}</div>
      <div class="row"><strong>Date :</strong> {{ formatDate(demande.createdAt) }}</div>
      <div class="row"><strong>Statut actuel :</strong> {{ getStatutNom(demande.statutId) }}</div>

      <div v-if="demande.documentPath" class="document-link-container">
        <a 
          :href="`https://be-solution-backend.onrender.com/documents/${demande.documentPath}`"
          target="_blank"
          class="document-link">
          📥 Voir le document généré
        </a>
      </div>

      <h3>Données fournies</h3>
      <pre class="donnees">{{ prettyJson(demande.donneesJson) }}</pre>

      <h3>Commentaires</h3>
      <p v-if="demande.commentaires">{{ demande.commentaires }}</p>
      <p v-else class="muted">Aucun commentaire</p>

      <hr />

      <div class="form-row">
        <label>Changer le statut</label>
        <select v-model="selectedStatutId">
          <option value="">-- Choisir --</option>
          <option v-for="s in statuts" :key="s.id" :value="s.id">{{ s.nom }}</option>
        </select>
      </div>

      <div class="form-row">
        <label>Ajouter un commentaire</label>
        <textarea v-model="comment" rows="3" placeholder="Votre commentaire..."></textarea>
      </div>

      <div class="actions">
        <button @click="updateDemande" :disabled="updating">Enregistrer</button>
        <button @click="refresh" class="ghost">Rafraîchir</button>
      </div>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DemandeDetailsAgent",
  data() {
    return {
      loading: true,
      updating: false,
      demande: null,
      statuts: [],
      selectedStatutId: "",
      comment: ""
    };
  },
  methods: {
    formatDate(d) {
      return d ? new Date(d).toLocaleString("fr-FR") : "-";
    },
    fullname(person) {
      if (!person) return "-";
      return [person.nom, person.prenom, person.postnom].filter(Boolean).join(" ");
    },
    prettyJson(raw) {
      if (!raw) return "{}";
      try {
        const parsed = typeof raw === "string" ? JSON.parse(raw) : raw;
        return JSON.stringify(parsed, null, 2);
      } catch {
        return String(raw);
      }
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
    getStatutNom(statutId) {
      const statut = this.statuts.find(s => s.id === statutId);
      return statut ? statut.nom : "—";
    },
    async fetchStatuts() {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get("https://be-solution-backend.onrender.com/api/statuts", {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.statuts = res.data;
      } catch (err) {
        console.error("Erreur chargement statuts:", err);
      }
    },
    async fetchDemande() {
      this.loading = true;
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(`https://be-solution-backend.onrender.com/api/demandes/${this.$route.params.id}`, {
          headers: { Authorization: `Bearer ${token}` }
        });
        this.demande = res.data;
        this.selectedStatutId = this.demande.statutId || "";
        this.comment = this.demande.commentaires || "";
      } catch (err) {
        console.error("Erreur chargement demande:", err);
        alert("Impossible de charger la demande.");
      } finally {
        this.loading = false;
      }
    },
    async updateDemande() {
      if (!this.selectedStatutId && !this.comment) {
        alert("Rien à mettre à jour.");
        return;
      }

      this.updating = true;
      try {
        const token = localStorage.getItem("token");
        const payload = {};
        if (this.selectedStatutId) payload.statutId = this.selectedStatutId;
        if (this.comment !== undefined) payload.commentaires = this.comment;

        // Mise à jour du statut et du commentaire
        await axios.put(
          `https://be-solution-backend.onrender.com/api/demandes/${this.demande.id}`,
          payload,
          { headers: { Authorization: `Bearer ${token}` } }
        );

        // Actualisation locale
        this.demande.commentaires = this.comment;
        this.demande.statutId = this.selectedStatutId;

        alert("Mise à jour enregistrée.");

        // Si statut = "en traitement", générer le document
        const statutToTraitement = this.statuts.find(s => s.nom === "en traitement");
        if (statutToTraitement && parseInt(this.selectedStatutId, 10) === statutToTraitement.id) {
          const genRes = await axios.put(
            `https://be-solution-backend.onrender.com/api/demandes/${this.demande.id}/generate-document`,
            {},
            { headers: { Authorization: `Bearer ${token}` } }
          );
          await this.fetchDemande();
          alert(genRes.data.message);
        }
      } catch (err) {
        console.error("Erreur mise à jour demande:", err.response?.data || err.message);
        alert("Erreur lors de la mise à jour.");
      } finally {
        this.updating = false;
      }
    },
    refresh() {
      this.fetchDemande();
    }
  },
  async mounted() {
    await Promise.all([this.fetchStatuts(), this.fetchDemande()]);
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

.page-demande-details {
  padding: 24px;
  font-family: "ABeeZee", sans-serif;
  background: #f9f9f9;
}

/* NAVBAR */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #0E2C5A;
  color: #fff;
  padding: 12px 20px;
  border-radius: 10px;
}
.logo {
  height: 42px;
}
.nav-btn {
  background: white;
  color: #0E2C5A;
  padding: 8px 14px;
  border-radius: 10px;
  font-weight: bold;
  text-decoration: none;
  transition: background 0.3s;
}
.nav-btn:hover {
  background: #f1f1f1;
}

/* CARD */
.card {
  background: #fff;
  padding: 24px;
  border-radius: 12px;
  margin-top: 24px;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.1);
}
.row {
  margin: 8px 0;
}
h2, h3 {
  color: #0E2C5A;
  font-family: "Ysabeau Office", sans-serif;
}
.donnees {
  background: #f3f3f3;
  padding: 12px;
  border-radius: 8px;
  max-height: 300px;
  overflow: auto;
  font-family: monospace;
}
.form-row {
  margin: 14px 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
textarea, select {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-family: "ABeeZee", sans-serif;
}
textarea:focus, select:focus {
  outline: none;
  border-color: #104B71;
}
.actions {
  display: flex;
  gap: 12px;
  margin-top: 20px;
}
button {
  background: #104B71;
  color: #fff;
  padding: 10px 16px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  font-weight: bold;
  font-family: "Inter", sans-serif;
}
button:hover {
  background: #0E2C5A;
}
button.ghost {
  background: #f1f1f1;
  color: #333;
}

/* LIEN DOCUMENT */
.document-link-container {
  margin-top: 15px;
}
.document-link {
  display: inline-block;
  padding: 10px 14px;
  background-color: #104B71;
  color: white;
  border-radius: 8px;
  text-decoration: none;
  font-weight: bold;
  transition: background 0.3s;
}
.document-link:hover {
  background-color: #0E2C5A;
}

/* ÉTATS */
.loading {
  color: #0E2C5A;
  font-weight: bold;
  font-family: "Inter", sans-serif;
}
.muted {
  color: #888;
  font-style: italic;
}
</style>