<template>
  <div class="reminder-list-container">
    <!-- Formulaire de création des rappels -->
    <div class="create-reminder">
      <div class="search-bar">
        <h2>🔎 Barre de recherche</h2>
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Rechercher un rappel"
          class="search-input"
        />
      </div>
      <h2>✏️ Ajouter un rappel</h2>
      <input
        v-model="newReminderName"
        type="text"
        placeholder="Entrez un nom de rappel"
        class="reminder-input"
      />
      <button @click="addReminder" class="add-button">Ajouter</button>
      <button @click="updateLocalDatabase" class="update-button">
        Mettre à jour
      </button>
      <!-- <button @click="listIndexes">Afficher les index</button> -->
    </div>

    <!-- Liste des rappels -->
    <div class="reminders-display">
      <h2>📋 Mes rappels</h2>
      <ul>
        <li
          v-for="reminder in filteredReminders"
          :key="reminder._id"
          class="reminder-item"
        >
          {{ reminder.name }}
          <button @click="deleteReminder(reminder._id)" class="delete-button">
            Supprimer
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import PouchDB from "pouchdb";
import { result } from "lodash";
import pouchdbFind from "pouchdb-find";

PouchDB.plugin(pouchdbFind);

export default {
  data() {
    return {
      newReminderName: "", // Nom du nouveau rappel
      reminders: [], // Liste des rappels
      searchQuery: "", // Requête de recherche
      db: null, // Instance de la base de données PouchDB
      remoteDB: null, // Base de données distante
    };
  },
  mounted() {
    this.initDatabase(); // Initialiser la base de données lors du montage du composant
    this.fetchReminders(); // Charger les rappels existants
  },

  computed: {
    // Filtrer les rappels en fonction de la barre de recherche
    filteredReminders() {
      if (!this.searchQuery) {
        return this.reminders; // Si la barre de recherche est vide, afficher tous les rappels
      }
      const lowerCaseQuery = this.searchQuery.toLowerCase();
      return this.reminders.filter((reminder) =>
        reminder.name.toLowerCase().includes(lowerCaseQuery),
      );
    },
  },

  methods: {
    // Initialisation de la base de données PouchDB
    async initDatabase() {
      this.db = new PouchDB("http://admin:admin@localhost:5984/reminders");
      this.remoteDB = new PouchDB(
        "http://admin:admin@localhost:5984/reminders_remote",
      );
      await this.createIndex();
    },

    // Créer un index sur le champ 'name'
    async createIndex() {
      try {
        const result = await this.db.createIndex({
          index: {
            fields: ["name"], // Champ sur lequel l'index sera créé
          },
        });
        console.log("Index créé avec succès", result);
      } catch (err) {
        console.error("Erreur lors de la création de l'index", err);
      }
    },

    // Ajouter un rappel
    async addReminder() {
      if (this.newReminderName.trim() === "") {
        alert("Veuillez entrer un nom de rappel.");
        return;
      }

      const newReminder = {
        _id: new Date().toISOString(), // Utilisation de l'ISO string comme identifiant
        name: this.newReminderName,
      };

      try {
        await this.db.put(newReminder); // Ajouter le rappel à la base de données
        this.newReminderName = ""; // Réinitialiser le champ de saisie
        this.fetchReminders(); // Recharger la liste des rappels
      } catch (error) {
        console.error("Erreur lors de l'ajout du rappel", error);
      }
    },

    // Supprimer un rappel
    async deleteReminder(id) {
      try {
        const doc = await this.db.get(id); // Récupérer le document à partir de son ID
        await this.db.remove(doc._id, doc._rev); // Supprimer le document
        this.fetchReminders(); // Recharger la liste des rappels après la suppression
      } catch (error) {
        console.error("Erreur lors de la suppression du rappel", error);
      }
    },

    // Charger les rappels à partir de la base de données
    async fetchReminders() {
      try {
        const result = await this.db.allDocs({
          include_docs: true,
        });
        this.reminders = result.rows
          .map((row) => row.doc)
          .filter((reminder) => reminder.name && reminder.name.trim() !== "");
      } catch (error) {
        console.error("Erreur lors du chargement des rappels", error);
      }
    },

    // Mettre à jour la base de données locale en la répliquant depuis la base distante
    async updateLocalDatabase() {
      try {
        console.log(
          "Démarrage de la réplication de la base locale vers la base distante...",
        );
        const result = await this.db.replicate.to(this.remoteDB);
        console.log("Réplication vers la base distante réussie", result);

        // Optionnellement, on peut aussi vérifier l'état de la réplication inverse
        console.log("Démarrage de la réplication depuis la base distante...");
        await this.db.replicate.from(this.remoteDB);
        console.log("Réplication depuis la base distante réussie");

        this.fetchReminders(); // Recharger les rappels après mise à jour
        console.log("Base de données locale mise à jour avec succès !");
      } catch (error) {
        console.error(
          "Erreur lors de la mise à jour de la base de données locale",
          error,
        );
      }
    },
  },
};
</script>
