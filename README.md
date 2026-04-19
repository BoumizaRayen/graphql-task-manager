# Système de gestion de tâches avec GraphQL

---

## Objectifs

- Comprendre comment configurer et utiliser GraphQL avec Node.js et Express.
- Créer un schéma GraphQL et des résolveurs pour gérer les requêtes et mutations d'une API de gestion de tâches.

---

## Technologies utilisées

| Outil | Rôle |
|-------|------|
| **Node.js** | Environnement d'exécution JavaScript côté serveur |
| **Express** | Framework HTTP pour créer le serveur web |
| **GraphQL** | Langage de requête pour l'API |
| **Apollo Server** | Serveur GraphQL intégré à Express |
| **@as-integrations/express5** | Middleware pour connecter Apollo Server à Express 5 |
| **@graphql-tools/schema** | Utilitaire pour associer les résolveurs au schéma |

---

## Structure du projet

```
tp5-graphql/
├── index.js            # Point d'entrée : configuration Express + Apollo Server
├── taskSchema.gql      # Schéma GraphQL (types, queries, mutations)
├── taskSchema.js       # Lecture et compilation du fichier .gql
├── taskResolver.js     # Résolveurs : logique métier et données en mémoire
├── package.json        # Métadonnées et dépendances du projet
├── .gitignore          # Fichiers exclus du dépôt git
└── README.md           # Documentation du projet
```

---

## Installation et démarrage

### Prérequis

- [Node.js](https://nodejs.org/en/download) installé sur votre machine.

### Étapes

```bash
# 1. Cloner le dépôt
git clone <URL_DU_REPO>
cd tp5-graphql

# 2. Installer les dépendances
npm install

# 3. Démarrer le serveur
npm start
```

Le serveur démarre sur le port **5000** par défaut.  
Accéder à l'interface Apollo Sandbox : [http://localhost:5000/graphql](http://localhost:5000/graphql)

---

## Explication des fichiers

### `taskSchema.gql` — Schéma GraphQL

Ce fichier définit la **structure des données** et les **opérations disponibles** sur l'API.

```graphql
type Task {
  id: ID!
  title: String!
  description: String!
  completed: Boolean!
  duration: Int          # Durée estimée en jours (ajoutée à l'étape 11)
}

type Query {
  task(id: ID!): Task    # Récupérer une tâche par son ID
  tasks: [Task]          # Récupérer toutes les tâches
}

type Mutation {
  addTask(title: String!, description: String!, completed: Boolean!, duration: Int): Task
  completeTask(id: ID!): Task
  changeDescription(id: ID!, description: String!): Task
  deleteTask(id: ID!): Task
}
```

- `!` signifie que le champ est **obligatoire** (non nullable).
- `[Task]` signifie une **liste** de tâches.
- `Query` = opérations de **lecture**.
- `Mutation` = opérations d'**écriture** (ajout, modification, suppression).

---

### `taskSchema.js` — Chargement du schéma

Ce fichier lit le fichier `.gql` depuis le disque et le compile en un objet schéma GraphQL utilisable par Apollo Server.

```js
const { buildSchema } = require('graphql');
// Lit le fichier taskSchema.gql de façon asynchrone
// puis le compile avec buildSchema()
// Exporte une Promise qui résout vers le schéma compilé
module.exports = getTaskSchema();
```

> **Pourquoi séparer le schéma dans un `.gql` ?**  
> Cela permet de bénéficier de la coloration syntaxique dans les éditeurs et de garder une séparation claire entre la définition du schéma et le code JavaScript.

---

### `taskResolver.js` — Résolveurs

Les résolveurs sont les **fonctions qui répondent aux requêtes GraphQL**. Ils définissent comment récupérer ou modifier les données.

Les données sont stockées **en mémoire** dans un tableau `tasks` (pas de base de données pour ce TP).

#### Structure d'un résolveur

```js
const taskResolver = {
  Query: {
    // Résolveurs pour les lectures
  },
  Mutation: {
    // Résolveurs pour les écritures
  }
};
```

Chaque fonction résolveur reçoit deux paramètres principaux :
- `_` : le parent (ignoré ici, d'où le nom `_`)
- `args` : les arguments passés dans la requête GraphQL (ex: `{ id, title, ... }`)

#### Résolveurs implémentés

| Résolveur | Type | Description |
|-----------|------|-------------|
| `tasks` | Query | Retourne toutes les tâches |
| `task(id)` | Query | Retourne une tâche par son ID |
| `addTask(...)` | Mutation | Crée une nouvelle tâche |
| `completeTask(id)` | Mutation | Passe `completed` à `true` |
| `changeDescription(id, description)` | Mutation | Modifie la description d'une tâche |
| `deleteTask(id)` | Mutation | Supprime une tâche et la retourne |

---

### `index.js` — Serveur principal

Ce fichier configure et démarre le serveur en combinant Express et Apollo Server.

```
taskSchemaPromise  ──┐
                     ├──► addResolversToSchema ──► ApolloServer ──► expressMiddleware ──► /graphql
taskResolver       ──┘
```

**Étapes du démarrage :**
1. Attendre la résolution du schéma (Promise).
2. Associer les résolveurs au schéma avec `addResolversToSchema`.
3. Créer une instance `ApolloServer` avec le schéma complet.
4. Démarrer Apollo Server (`server.start()`).
5. Monter le middleware GraphQL sur la route `/graphql`.
6. Lancer Express sur le port 5000.

---

## Requêtes GraphQL — Exemples

Toutes ces requêtes peuvent être testées dans **Apollo Sandbox** à l'adresse `http://localhost:5000/graphql`.

### Récupérer toutes les tâches

```graphql
query {
  tasks {
    id
    title
    description
    completed
    duration
  }
}
```

### Récupérer une tâche par ID

```graphql
query {
  task(id: "1") {
    id
    title
    description
    completed
    duration
  }
}
```

### Ajouter une nouvelle tâche

```graphql
mutation {
  addTask(
    title: "Déploiement sur AWS"
    description: "Déployer l'application sur un serveur EC2 avec Docker."
    completed: false
    duration: 3
  ) {
    id
    title
    duration
  }
}
```

### Marquer une tâche comme terminée

```graphql
mutation {
  completeTask(id: "1") {
    id
    title
    completed
  }
}
```

### Changer la description d'une tâche

```graphql
mutation {
  changeDescription(id: "2", description: "Nouvelle description mise à jour.") {
    id
    title
    description
  }
}
```

### Supprimer une tâche

```graphql
mutation {
  deleteTask(id: "3") {
    id
    title
  }
}
```

---

## Concept clé — GraphQL vs REST

| Critère | REST | GraphQL |
|---------|------|---------|
| Endpoint | Un par ressource (`/tasks`, `/tasks/:id`) | Un seul (`/graphql`) |
| Données reçues | Toujours le même format fixe | Exactement ce que le client demande |
| Sur-fetching | Fréquent (données inutiles reçues) | Éliminé |
| Sous-fetching | Nécessite plusieurs appels | Une seule requête suffit |
| Documentation | Manuelle (Swagger...) | Introspection intégrée |

---

## Évolutions possibles

- Connecter une base de données (MongoDB, PostgreSQL) à la place du tableau en mémoire.
- Ajouter une mutation `updateTask` pour modifier tous les champs d'une tâche.
- Ajouter des **Subscriptions** GraphQL pour des mises à jour en temps réel.
- Ajouter une authentification JWT sur le contexte Apollo.
