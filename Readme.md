# 🌿 EcoTrack & Tech Dashboard

> Application de monitoring mixte : indicateurs écologiques globaux + gestion d'inventaire informatique local, avec alertes vers des applications tierces.

![Vue.js](https://img.shields.io/badge/Vue.js-3.x-42b883?logo=vue.js&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Express-339933?logo=node.js&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-3-003B57?logo=sqlite&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

---

## Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Architecture](#architecture)
- [Fonctionnalités](#fonctionnalités)
- [Stack technique](#stack-technique)
- [Structure du projet](#structure-du-projet)
- [Installation](#installation)
- [Configuration](#configuration)
- [API Backend](#api-backend)
- [Intégrations API](#intégrations-api)
- [Défis techniques](#défis-techniques)
- [Feuille de route](#feuille-de-route)
- [Contribuer](#contribuer)

---

## Vue d'ensemble

**EcoTrack & Tech Dashboard** est une application web fullstack conçue comme projet d'apprentissage progressif de Vue.js 3. Elle combine trois flux de données distincts :

- **Données publiques en temps réel** — empreinte carbone de l'électricité, météo, via des API ouvertes
- **Inventaire local** — gestion CRUD d'équipements informatiques stockés dans SQLite
- **Alertes externes** — notifications vers Discord, Slack ou synchronisation avec GLPI/Trello en cas d'incident

Ce projet couvre tous les cas d'usage API essentiels d'une application Vue.js de production.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Vue.js (Frontend)                      │
│          Dashboard · Inventaire · Alertes                │
└───────────────────┬─────────────────────────────────────┘
                    │
        ┌───────────┼───────────────┐
        │           │               │
        ▼           ▼               ▼
  (1) API Publiques  (2) Backend    (3) App Tierce
  Open-Meteo         Node/Express   Discord Webhook
  CO2 Signal         └─► SQLite     Slack Webhook
  electricity maps                  GLPI API
                                     Trello API
```

### Les trois types de communication API

| # | Type | Direction | Technologie | Objectif pédagogique |
|---|------|-----------|-------------|----------------------|
| 1 | **API publique** | Vue → Internet | Axios / Fetch | `onMounted`, async/await, formatage pour graphiques |
| 2 | **API CRUD locale** | Vue ↔ Backend ↔ SQLite | REST + Express | Pinia, formulaires, réactivité, validation |
| 3 | **App tierce** | Backend → Service externe | Webhooks / REST | Gestion d'événements, payload JSON, authentification API |

---

## Fonctionnalités

### Frontend Vue.js

#### Dashboard écologique
- Graphiques temps réel (Chart.js ou ApexCharts) alimentés par Open-Meteo et CO2 Signal
- Sélection de la région pour afficher l'intensité carbone du réseau électrique local
- Historique des mesures avec courbes de tendance

#### Gestion d'inventaire (CRUD complet)
- Liste des équipements avec recherche et filtres en temps réel (computed properties)
- Formulaire d'ajout et d'édition avec validation côté client
- Suppression avec confirmation
- Indicateurs de statut visuels : `Opérationnel` · `Dégradé` · `En panne`

#### Système d'alertes
- Déclenchement manuel ou automatique lors du passage en statut `En panne`
- Notification immédiate vers Discord ou Slack via webhook
- Historique des alertes envoyées
- Option avancée : création de ticket dans GLPI ou Trello

#### UX & qualité
- Mode sombre / clair (persisté via Pinia + localStorage)
- Spinners de chargement pendant les appels API
- Gestion des erreurs réseau avec messages utilisateur clairs
- Design responsive (mobile-first)

---

### Backend Node.js / Express

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/api/equipements` | `GET` | Récupérer tous les équipements |
| `/api/equipements/:id` | `GET` | Détail d'un équipement |
| `/api/equipements` | `POST` | Créer un équipement |
| `/api/equipements/:id` | `PUT` | Modifier un équipement |
| `/api/equipements/:id` | `DELETE` | Supprimer un équipement |
| `/api/alertes` | `POST` | Envoyer une alerte vers l'app tierce |
| `/api/proxy/carbon` | `GET` | Proxy vers CO2 Signal (gestion de la clé API) |

---

## Stack technique

### Frontend
- **Vue.js 3** — Composition API, `<script setup>`
- **Vite** — bundler et serveur de développement
- **Vue Router 4** — navigation SPA avec guards d'authentification
- **Pinia** — gestion d'état global (inventaire, préférences, alertes)
- **Axios** — client HTTP avec intercepteurs
- **Chart.js** ou **ApexCharts** — visualisation des données
- **Tailwind CSS** — styles utilitaires

### Backend
- **Node.js + Express** — serveur REST léger
- **better-sqlite3** — interface SQLite synchrone et performante
- **cors** — gestion des origines cross-domain en développement
- **dotenv** — variables d'environnement

### Infrastructure & outils
- **SQLite** — base de données locale, zéro configuration
- **Postman / Insomnia** — test des endpoints
- **Discord Webhooks** ou **Slack Incoming Webhooks** — notifications
- **GLPI API** / **Trello API** — option avancée pour la gestion des incidents

---

## Structure du projet

```
ecotrack-dashboard/
├── frontend/                   # Application Vue.js
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── common/         # Composants réutilisables
│   │   │   │   ├── StatCard.vue
│   │   │   │   ├── AlertButton.vue
│   │   │   │   ├── DataTable.vue
│   │   │   │   └── LoadingSpinner.vue
│   │   │   ├── dashboard/
│   │   │   │   ├── CarbonChart.vue
│   │   │   │   └── WeatherWidget.vue
│   │   │   └── inventaire/
│   │   │       ├── EquipementForm.vue
│   │   │       └── EquipementList.vue
│   │   ├── composables/        # Logique réutilisable (hooks)
│   │   │   ├── useApi.js
│   │   │   ├── useCarbonData.js
│   │   │   └── useAlertes.js
│   │   ├── router/
│   │   │   └── index.js
│   │   ├── stores/             # Stores Pinia
│   │   │   ├── inventaire.js
│   │   │   ├── alertes.js
│   │   │   └── preferences.js
│   │   ├── views/
│   │   │   ├── DashboardView.vue
│   │   │   ├── InventaireView.vue
│   │   │   └── AlertesView.vue
│   │   ├── App.vue
│   │   └── main.js
│   ├── .env.example
│   ├── package.json
│   └── vite.config.js
│
├── backend/                    # Serveur Express
│   ├── src/
│   │   ├── controllers/
│   │   │   ├── equipements.js
│   │   │   └── alertes.js
│   │   ├── db/
│   │   │   ├── init.js         # Création des tables SQLite
│   │   │   └── database.js     # Instance better-sqlite3
│   │   ├── middleware/
│   │   │   └── errorHandler.js
│   │   ├── routes/
│   │   │   ├── equipements.js
│   │   │   └── alertes.js
│   │   └── services/
│   │       ├── discordService.js
│   │       └── glpiService.js
│   ├── data/
│   │   └── ecotrack.db         # Fichier SQLite (gitignored)
│   ├── .env.example
│   ├── package.json
│   └── server.js
│
└── README.md
```

---

## Installation

### Prérequis

- Node.js >= 18.x
- npm >= 9.x

### 1. Cloner le dépôt

```bash
git clone https://github.com/ton-user/ecotrack-dashboard.git
cd ecotrack-dashboard
```

### 2. Installer et démarrer le backend

```bash
cd backend
npm install

# Copier et remplir le fichier d'environnement
cp .env.example .env

# Initialiser la base de données SQLite
node src/db/init.js

# Démarrer le serveur (port 3000 par défaut)
npm run dev
```

### 3. Installer et démarrer le frontend

```bash
cd ../frontend
npm install

cp .env.example .env

# Démarrer Vite (port 5173 par défaut)
npm run dev
```

### 4. Accéder à l'application

Ouvrir `http://localhost:5173` dans le navigateur.

---

## Configuration

### Backend — `backend/.env`

```env
# Serveur
PORT=3000
NODE_ENV=development

# Base de données
DB_PATH=./data/ecotrack.db

# Discord (option simple)
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxxx/yyyy

# Slack (option simple)
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxxx/yyyy/zzzz

# GLPI (option avancée)
GLPI_URL=https://mon-instance-glpi.exemple.com
GLPI_APP_TOKEN=mon_app_token
GLPI_USER_TOKEN=mon_user_token
```

### Frontend — `frontend/.env`

```env
# URL du backend local
VITE_API_BASE_URL=http://localhost:3000/api

# CO2 Signal (clé gratuite sur electricitymaps.com)
VITE_CO2_SIGNAL_API_KEY=ta_cle_api

# Open-Meteo (aucune clé requise)
VITE_OPEN_METEO_URL=https://api.open-meteo.com/v1
```

> **Important :** Ne jamais committer de fichiers `.env` contenant de vraies clés. Le fichier `.gitignore` doit lister `*.env` et `*.db`.

### Proxy Vite (développement)

Dans `frontend/vite.config.js`, le proxy évite les erreurs CORS en développement :

```javascript
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  }
})
```

---

## API Backend

### Schéma de la table `equipements`

```sql
CREATE TABLE equipements (
  id               INTEGER PRIMARY KEY AUTOINCREMENT,
  nom              TEXT    NOT NULL,
  type             TEXT    NOT NULL,         -- 'serveur', 'switch', 'poste', etc.
  statut           TEXT    NOT NULL DEFAULT 'Opérationnel',
  consommation_w   INTEGER,                  -- Consommation en watts
  emplacement      TEXT,
  ip               TEXT,
  notes            TEXT,
  created_at       TEXT    DEFAULT (datetime('now')),
  updated_at       TEXT    DEFAULT (datetime('now'))
);
```

### Exemple de requête — POST `/api/equipements`

```json
{
  "nom": "SRV-PROD-01",
  "type": "serveur",
  "statut": "Opérationnel",
  "consommation_w": 350,
  "emplacement": "Baie A - Rack 3",
  "ip": "192.168.1.10"
}
```

### Exemple de requête — POST `/api/alertes`

```json
{
  "equipement_id": 5,
  "message": "SRV-PROD-01 est passé en statut En panne",
  "destination": "discord"
}
```

---

## Intégrations API

### A. Open-Meteo (météo, aucune clé requise)

Documentation : [open-meteo.com/en/docs](https://open-meteo.com/en/docs)

```javascript
// composables/useWeather.js
const { data } = await axios.get('https://api.open-meteo.com/v1/forecast', {
  params: {
    latitude: -18.91,
    longitude: 47.54,
    current: 'temperature_2m,wind_speed_10m'
  }
})
```

### B. CO2 Signal / Electricity Maps (clé gratuite)

Documentation : [static.electricitymaps.com/api/docs](https://static.electricitymaps.com/api/docs/index.html)

```javascript
// composables/useCarbonData.js
const { data } = await axios.get('https://api.electricitymap.org/v3/carbon-intensity/latest', {
  params: { zone: 'FR' },
  headers: { 'auth-token': import.meta.env.VITE_CO2_SIGNAL_API_KEY }
})
```

### C. Discord Webhook (notification d'alerte)

```javascript
// backend/src/services/discordService.js
const sendDiscordAlert = async (message) => {
  await axios.post(process.env.DISCORD_WEBHOOK_URL, {
    embeds: [{
      title: '🔴 Alerte EcoTrack',
      description: message,
      color: 0xff0000,
      timestamp: new Date().toISOString()
    }]
  })
}
```

### D. GLPI (option avancée — création de ticket)

```javascript
// backend/src/services/glpiService.js
// Étape 1 : Ouvrir une session
const { data: session } = await axios.get(`${GLPI_URL}/apirest.php/initSession`, {
  headers: {
    'App-Token': process.env.GLPI_APP_TOKEN,
    'Authorization': `user_token ${process.env.GLPI_USER_TOKEN}`
  }
})

// Étape 2 : Créer le ticket
await axios.post(`${GLPI_URL}/apirest.php/Ticket`, {
  input: {
    name: 'Panne détectée : SRV-PROD-01',
    content: 'Équipement passé en statut En panne via EcoTrack.',
    urgency: 5,
    itilcategories_id: 1
  }
}, {
  headers: { 'Session-Token': session.session_token }
})
```

---

## Défis techniques

Ces challenges sont intégrés au projet pour garantir une progression réelle sur Vue.js :

### 1. Composants réutilisables

Créer des composants génériques paramétrables via des props :

```vue
<!-- StatCard.vue -->
<StatCard
  title="Intensité carbone"
  :value="carbonIntensity"
  unit="gCO₂/kWh"
  :trend="trend"
  color="green"
/>
```

### 2. Gestion d'état avec Pinia

Centraliser l'inventaire dans un store dédié — aucune donnée métier ne vit dans les composants :

```javascript
// stores/inventaire.js
export const useInventaireStore = defineStore('inventaire', () => {
  const equipements = ref([])
  const filtre = ref('')
  const statutFiltre = ref('tous')

  const equipementsFiltres = computed(() =>
    equipements.value.filter(e =>
      e.nom.toLowerCase().includes(filtre.value.toLowerCase()) &&
      (statutFiltre.value === 'tous' || e.statut === statutFiltre.value)
    )
  )

  const charger = async () => { /* appel API */ }
  const ajouter = async (data) => { /* POST + push local */ }
  const supprimer = async (id) => { /* DELETE + splice local */ }

  return { equipements, filtre, statutFiltre, equipementsFiltres, charger, ajouter, supprimer }
})
```

### 3. Filtrage et recherche en temps réel

Les `computed` de Pinia recalculent automatiquement la liste à chaque frappe, sans appel réseau supplémentaire.

### 4. Gestion des erreurs et états de chargement

Encapsuler chaque appel dans un composable `useApi` qui expose `{ data, loading, error }` :

```javascript
// composables/useApi.js
export const useApi = (fn) => {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)

  const execute = async (...args) => {
    loading.value = true
    error.value = null
    try {
      data.value = await fn(...args)
    } catch (e) {
      error.value = e.message || 'Erreur réseau'
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, execute }
}
```

---

## Feuille de route

- [x] Architecture de base Vue.js + Express + SQLite
- [x] CRUD équipements complet
- [x] Intégration Open-Meteo (météo temps réel)
- [ ] Intégration CO2 Signal (empreinte carbone)
- [ ] Graphiques Chart.js sur le dashboard
- [ ] Webhook Discord pour les alertes
- [ ] Filtres avancés et recherche temps réel
- [ ] Mode sombre / clair avec Pinia
- [ ] Intégration GLPI (tickets d'incidents)
- [ ] Tests Vitest sur les composables
- [ ] Déploiement Railway (backend) + Vercel (frontend)

---

## Contribuer

Les contributions sont les bienvenues. Pour proposer une amélioration :

1. Forker le dépôt
2. Créer une branche (`git checkout -b feature/ma-fonctionnalite`)
3. Committer les changements (`git commit -m 'feat: ajouter ma fonctionnalité'`)
4. Pousser la branche (`git push origin feature/ma-fonctionnalite`)
5. Ouvrir une Pull Request

---

## Licence

Distribué sous licence MIT. Voir `LICENSE` pour plus de détails.

---

*Projet conçu comme terrain d'apprentissage Vue.js 3 fullstack — API publiques · Backend REST · SQLite · Intégrations tierces.*