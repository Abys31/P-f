<template>
  <div class="card">
    <h2>Enregistrer une participation</h2>

    <form class="participation-form" @submit.prevent="Participation">
      <!-- Personne -->
      <label for="personne" class="label">Personne</label>
      <select
        id="personne"
        v-model="selectedPersonne"
        required
        class="select"
      >
        <option value="" disabled>-- Choisir une personne --</option>
        <option
          v-for="personne in personnes"
          :key="personne.matricule"
          :value="personne.matricule"
        >
          {{ personne.nom }} {{ personne.prenom }}
        </option>
      </select>

      <!-- Projet -->
      <label for="projet" class="label">Projet</label>
      <select
        id="projet"
        v-model="selectedProjet"
        required
        class="select"
      >
        <option value="" disabled>-- Choisir un projet --</option>
        <option
          v-for="projet in projets"
          :key="projet.id"
          :value="projet.id"
        >
          {{ projet.nom }}
        </option>
      </select>

      <!-- Rôle -->
      <label for="role" class="label">Rôle</label>
      <input
        id="role"
        type="text"
        v-model="role"
        placeholder="Ex: développeur"
        required
        class="input"
      />

      <!-- Pourcentage (Slider) -->
      <label for="pourcentage" class="label">Pourcentage</label>
      <div class="slider-container">
        <input
          id="pourcentage"
          type="range"
          v-model.number="pourcentage"
          step="0.1"
          min="0"
          max="1"
          class="slider"
        />
        <span class="slider-value">{{ (pourcentage * 100).toFixed(0) }}%</span>
      </div>

      <button type="submit" class="btn-submit">Enregistrer</button>
    </form>

    <!-- Messages de feedback -->
    <div v-if="errorMsg" class="error">{{ errorMsg }}</div>
    <div v-if="successMsg" class="success">{{ successMsg }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios';

// Données réactives
const personnes = ref([])
const projets = ref([])
const selectedPersonne = ref('')
const selectedProjet = ref('')
const role = ref('')
const pourcentage = ref(0.1) // Valeur par défaut : 0.1 = 10%
const errorMsg = ref('')
const successMsg = ref('')

// Charger la liste des personnes
const chargerPersonnes = async () => {
  try {
    const resp = await axios.get('/api/personnes')
    personnes.value = resp.data._embedded
      ? resp.data._embedded.personnes
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement personnes : ' + err
  }
}

// Charger la liste des projets
const chargerProjets = async () => {
  try {
    const resp = await axios.get('/api/projets')
    projets.value = resp.data._embedded
      ? resp.data._embedded.projets
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement projets : ' + err
  }
}

// Enregistrer la participation
const Participation = async () => {
  const payload = {
    matricule: selectedPersonne.value,
    codeProjet: selectedProjet.value,
    role: role.value,
    pourcentage: pourcentage.value,
  }
  try {
    await axios.post('/api/gestion/participation', payload)
    successMsg.value = 'Participation enregistrée avec succès.'
    errorMsg.value = ''
    // Réinitialisation
    selectedPersonne.value = ''
    selectedProjet.value = ''
    role.value = ''
    pourcentage.value = 0.1
  } catch (err) {
    successMsg.value = ''
    if (err.response && err.response.data && err.response.data.message) {
      errorMsg.value = err.response.data.message
    } else {
      errorMsg.value = 'Erreur lors de l’enregistrement.'
    }
  }
}

onMounted(() => {
  chargerPersonnes()
  chargerProjets()
})
</script>

<style scoped>
/* --- Mise en forme générale --- */
/* --- Mise en forme générale --- */
.card {
  max-width: 500px;
  margin: 3rem auto;
  padding: 2.5rem;
  background-color: #f0f4f8; /* Fond clair et doux */
  border-radius: 16px; /* Coins arrondis plus grands */
  box-shadow: 0 6px 20px rgba(0,0,0,0.15); /* Ombre plus marquée */
}

h2 {
  margin-bottom: 2rem;
  text-align: center;
  font-size: 1.5rem;
  color: #333; /* Texte plus sombre */
  font-family: 'Arial', sans-serif; /* Typo différente */
}

/* --- Formulaire --- */
.participation-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem; /* Espacement plus large entre les champs */
}

.label {
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: #2c3e50; /* Couleur plus foncée pour les labels */
}

.select, .input {
  padding: 0.8rem;
  border: 2px solid #3498db; /* Bordure bleue */
  border-radius: 8px; /* Coins arrondis plus marqués */
  font-size: 1rem;
  background-color: #ecf0f1; /* Fond léger */
  color: #2c3e50; /* Texte plus sombre */
}

/* Le slider (input range) */
.slider {
  flex: 1;
  -webkit-appearance: none;
  height: 8px; /* Slider plus épais */
  background: #bdc3c7;
  border-radius: 5px;
  outline: none;
  cursor: pointer;
}
.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #2980b9; /* Couleur bleue plus intense */
  cursor: pointer;
  border: 3px solid #fff;
  box-shadow: 0 0 4px rgba(0,0,0,0.3);
}
.slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #2980b9;
  cursor: pointer;
  border: 3px solid #fff;
  box-shadow: 0 0 4px rgba(0,0,0,0.3);
}

/* Valeur du slider */
.slider-value {
  width: 50px;
  text-align: center;
  font-weight: bold;
  color: #2980b9; /* Texte bleu pour la valeur */
}

/* --- Bouton --- */
.btn-submit {
  padding: 0.75rem 1.5rem;
  background-color: #2980b9; /* Nouveau bleu plus profond */
  color: #fff;
  font-weight: 700;
  border: none;
  border-radius: 8px; /* Coins arrondis plus marqués */
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  font-size: 1rem;
}
.btn-submit:hover {
  background-color: #3498db; /* Couleur au survol */
  transform: translateY(-3px); /* Légère élévation au survol */
}

/* --- Feedback --- */
.error {
  margin-top: 1rem;
  color: #e74c3c; /* Couleur d'erreur plus vive */
}
.success {
  margin-top: 1rem;
  color: #27ae60; /* Couleur de succès verte */
}

</style>
