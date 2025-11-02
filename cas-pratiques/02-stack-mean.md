# Stack MEAN avec Docker (MongoDB + Express + Angular + Node.js)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous guide dans la création d'une **stack MEAN complète** avec Docker. MEAN est l'acronyme de **MongoDB + Express + Angular + Node.js**, une stack JavaScript moderne qui permet de développer des applications web full-stack en utilisant JavaScript de bout en bout.

**Ce que vous allez apprendre :**
- Comprendre l'architecture d'une stack MEAN
- Déployer MongoDB, Express, Angular et Node.js avec Docker Compose
- Créer une API REST avec Express et Node.js
- Développer un frontend avec Angular
- Faire communiquer le frontend et le backend
- Gérer les données avec MongoDB

**Durée estimée :** 45-60 minutes

---

## 🎯 Qu'est-ce qu'une Stack MEAN ?

### Définition

Une **stack MEAN** est un ensemble de technologies JavaScript/TypeScript permettant de créer des applications web full-stack modernes. L'avantage principal : **un seul langage (JavaScript) du frontend au backend**.

### Les 4 composants

| Composant | Rôle | Technologie |
|-----------|------|-------------|
| **M**ongoDB | Base de données | NoSQL orientée documents (JSON) |
| **E**xpress | Framework backend | Framework web minimaliste pour Node.js |
| **A**ngular | Framework frontend | Framework JavaScript/TypeScript de Google |
| **N**ode.js | Runtime serveur | Exécution de JavaScript côté serveur |

### Schéma de fonctionnement

```
┌────────────────────────────────────────────────────────┐
│                    NAVIGATEUR                          │
│              (http://localhost:4200)                   │
│                                                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │           ANGULAR (Frontend)                    │   │
│  │  - Interface utilisateur                        │   │
│  │  - Requêtes HTTP vers l'API                     │   │
│  └───────────────────┬─────────────────────────────┘   │
└────────────────────────┼───────────────────────────────┘
                         │
                         │ HTTP (REST API)
                         │
┌────────────────────────▼───────────────────────────────┐
│         CONTENEUR NODE.JS + EXPRESS (Backend)          │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Express écoute sur le port 3000                 │   │
│  │ - Routes API (/api/users, /api/tasks...)        │   │
│  │ - Logique métier                                │   │
│  │ - Connexion à MongoDB                           │   │
│  └───────────────────┬─────────────────────────────┘   │
└────────────────────────┼───────────────────────────────┘
                         │
                         │ Requêtes MongoDB
                         │
┌────────────────────────▼───────────────────────────────┐
│              CONTENEUR MONGODB                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │ MongoDB écoute sur le port 27017                │   │
│  │ - Stockage des documents JSON                   │   │
│  │ - Collections (users, tasks...)                 │   │
│  └─────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

### Pourquoi choisir la stack MEAN ?

| Avantage | Explication |
|----------|-------------|
| **JavaScript partout** | Un seul langage pour frontend, backend et base de données (JSON) |
| **Moderne et performant** | Technologies récentes et optimisées |
| **Écosystème riche** | npm (Node Package Manager) avec des millions de packages |
| **Temps réel facile** | WebSockets natifs avec Node.js |
| **Scalabilité** | MongoDB et Node.js sont conçus pour scaler horizontalement |
| **Communauté active** | Support important, tutoriels nombreux |

---

## 🔧 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ Docker installé (version 20.10+)
- ✅ Docker Compose installé (version 2.0+)
- ✅ Node.js installé localement (version 18+) pour développer Angular
- ✅ Un éditeur de texte (VS Code recommandé)
- ✅ Connaissances de base en JavaScript (recommandé)

**Vérification rapide :**

```bash
docker --version
docker-compose --version
node --version
npm --version
```

**💡 Note :** Nous aurons besoin de Node.js **localement** pour développer le frontend Angular, même si le backend tourne dans Docker.

---

## 📁 Étape 1 : Structure du projet

### 1.1 Créer l'arborescence

Créez un nouveau dossier pour votre projet MEAN :

```bash
# Créer le dossier principal
mkdir mean-stack
cd mean-stack

# Créer les sous-dossiers
mkdir -p backend/src
mkdir -p frontend
mkdir -p mongodb/data
mkdir -p mongodb/init
```

**Votre structure sera :**

```
mean-stack/
├── docker-compose.yml        # Configuration Docker
├── backend/                   # API Node.js + Express
│   ├── Dockerfile            # Image Docker personnalisée
│   ├── package.json          # Dépendances Node.js
│   ├── .dockerignore         # Fichiers à ignorer
│   └── src/
│       └── server.js         # Point d'entrée de l'API
├── frontend/                  # Application Angular
│   ├── Dockerfile            # Image Docker pour Angular
│   └── (fichiers Angular)    # Générés par Angular CLI
└── mongodb/
    ├── data/                 # Données persistantes
    └── init/                 # Scripts d'initialisation
        └── init-db.js        # Script MongoDB
```

---

## 🗄️ Étape 2 : Configuration de MongoDB

### 2.1 Script d'initialisation

Créez le fichier `mongodb/init/init-db.js` :

```javascript
// ==========================================
// Script d'initialisation MongoDB
// Exécuté automatiquement au 1er démarrage
// ==========================================

// Connexion à la base de données
db = db.getSiblingDB('mean_db');

// Créer une collection "users" avec validation
db.createCollection('users', {
    validator: {
        $jsonSchema: {
            bsonType: 'object',
            required: ['name', 'email'],
            properties: {
                name: {
                    bsonType: 'string',
                    description: 'Nom de l\'utilisateur (requis)'
                },
                email: {
                    bsonType: 'string',
                    pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$',
                    description: 'Email valide (requis)'
                },
                age: {
                    bsonType: 'int',
                    minimum: 0,
                    maximum: 150,
                    description: 'Âge (optionnel)'
                },
                createdAt: {
                    bsonType: 'date',
                    description: 'Date de création'
                }
            }
        }
    }
});

// Créer une collection "tasks" (tâches)
db.createCollection('tasks', {
    validator: {
        $jsonSchema: {
            bsonType: 'object',
            required: ['title', 'completed'],
            properties: {
                title: {
                    bsonType: 'string',
                    description: 'Titre de la tâche (requis)'
                },
                description: {
                    bsonType: 'string',
                    description: 'Description (optionnel)'
                },
                completed: {
                    bsonType: 'bool',
                    description: 'Statut de complétion (requis)'
                },
                userId: {
                    bsonType: 'objectId',
                    description: 'ID de l\'utilisateur'
                },
                createdAt: {
                    bsonType: 'date',
                    description: 'Date de création'
                }
            }
        }
    }
});

// Créer des index pour optimiser les recherches
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ name: 1 });
db.tasks.createIndex({ userId: 1 });
db.tasks.createIndex({ completed: 1 });

// Insérer des données de test
print('Insertion des utilisateurs de test...');

db.users.insertMany([
    {
        name: 'Alice Dupont',
        email: 'alice@example.com',
        age: 28,
        createdAt: new Date()
    },
    {
        name: 'Bob Martin',
        email: 'bob@example.com',
        age: 34,
        createdAt: new Date()
    },
    {
        name: 'Charlie Durand',
        email: 'charlie@example.com',
        age: 25,
        createdAt: new Date()
    }
]);

// Récupérer les IDs des utilisateurs créés
const alice = db.users.findOne({ email: 'alice@example.com' });
const bob = db.users.findOne({ email: 'bob@example.com' });

print('Insertion des tâches de test...');

db.tasks.insertMany([
    {
        title: 'Apprendre Docker',
        description: 'Comprendre les concepts de base de la conteneurisation',
        completed: true,
        userId: alice._id,
        createdAt: new Date()
    },
    {
        title: 'Maîtriser MongoDB',
        description: 'Apprendre les requêtes et l\'agrégation',
        completed: false,
        userId: alice._id,
        createdAt: new Date()
    },
    {
        title: 'Créer une API REST',
        description: 'Développer une API avec Express et Node.js',
        completed: false,
        userId: bob._id,
        createdAt: new Date()
    },
    {
        title: 'Développer le frontend Angular',
        description: 'Créer l\'interface utilisateur',
        completed: false,
        userId: bob._id,
        createdAt: new Date()
    }
]);

// Afficher les statistiques
print('===================================');
print('Base de données initialisée avec succès !');
print('Utilisateurs créés : ' + db.users.count());
print('Tâches créées : ' + db.tasks.count());
print('===================================');
```

### 2.2 Comprendre le script

| Élément | Explication |
|---------|-------------|
| `db.getSiblingDB('mean_db')` | Sélectionne (ou crée) la base de données `mean_db` |
| `createCollection` avec `validator` | Crée une collection avec validation de schéma (équivalent des tables SQL) |
| `$jsonSchema` | Définit la structure attendue des documents |
| `createIndex` | Crée des index pour accélérer les recherches |
| `{ unique: true }` | Garantit l'unicité d'une valeur (comme email) |
| `ObjectId` | Type MongoDB pour les identifiants uniques |
| `insertMany` | Insère plusieurs documents en une fois |

---

## 🔙 Étape 3 : Backend (Node.js + Express)

### 3.1 Créer le package.json

Créez le fichier `backend/package.json` :

```json
{
  "name": "mean-backend",
  "version": "1.0.0",
  "description": "API REST pour stack MEAN",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  },
  "keywords": ["mean", "express", "mongodb", "api"],
  "author": "Votre Nom",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^6.3.0",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  }
}
```

### 3.2 Créer le serveur Express

Créez le fichier `backend/src/server.js` :

```javascript
// ==========================================
// SERVEUR API REST - STACK MEAN
// Node.js + Express + MongoDB
// ==========================================

const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const cors = require('cors');

// ==========================================
// CONFIGURATION
// ==========================================

const app = express();
const PORT = process.env.PORT || 3000;

// URL de connexion à MongoDB (nom du service Docker)
const MONGO_URL = process.env.MONGO_URL || 'mongodb://mongodb:27017';
const DB_NAME = 'mean_db';

// Client MongoDB
let db;
let usersCollection;
let tasksCollection;

// ==========================================
// MIDDLEWARES
// ==========================================

// Parser JSON dans le body des requêtes
app.use(express.json());

// CORS : Autoriser les requêtes depuis Angular (port 4200)
app.use(cors({
    origin: ['http://localhost:4200', 'http://localhost:80'],
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// Logger les requêtes (pour le développement)
app.use((req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
    next();
});

// ==========================================
// CONNEXION À MONGODB
// ==========================================

async function connectToMongoDB() {
    try {
        console.log('Connexion à MongoDB...');
        const client = await MongoClient.connect(MONGO_URL, {
            useNewUrlParser: true,
            useUnifiedTopology: true
        });

        db = client.db(DB_NAME);
        usersCollection = db.collection('users');
        tasksCollection = db.collection('tasks');

        console.log('✅ Connecté à MongoDB !');
    } catch (error) {
        console.error('❌ Erreur de connexion à MongoDB:', error);
        process.exit(1);
    }
}

// ==========================================
// ROUTES - PAGE D'ACCUEIL
// ==========================================

app.get('/', (req, res) => {
    res.json({
        message: '🚀 API REST MEAN - Bienvenue !',
        version: '1.0.0',
        endpoints: {
            users: '/api/users',
            tasks: '/api/tasks',
            health: '/api/health'
        }
    });
});

// ==========================================
// ROUTES - HEALTH CHECK
// ==========================================

app.get('/api/health', async (req, res) => {
    try {
        // Vérifier la connexion MongoDB
        await db.admin().ping();

        res.json({
            status: 'OK',
            database: 'Connected',
            timestamp: new Date().toISOString()
        });
    } catch (error) {
        res.status(500).json({
            status: 'ERROR',
            database: 'Disconnected',
            error: error.message
        });
    }
});

// ==========================================
// ROUTES - UTILISATEURS (USERS)
// ==========================================

// GET /api/users - Récupérer tous les utilisateurs
app.get('/api/users', async (req, res) => {
    try {
        const users = await usersCollection.find({}).toArray();
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// GET /api/users/:id - Récupérer un utilisateur par ID
app.get('/api/users/:id', async (req, res) => {
    try {
        const user = await usersCollection.findOne({
            _id: new ObjectId(req.params.id)
        });

        if (!user) {
            return res.status(404).json({ error: 'Utilisateur non trouvé' });
        }

        res.json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// POST /api/users - Créer un nouvel utilisateur
app.post('/api/users', async (req, res) => {
    try {
        const { name, email, age } = req.body;

        // Validation simple
        if (!name || !email) {
            return res.status(400).json({
                error: 'Les champs name et email sont requis'
            });
        }

        // Vérifier que l'email n'existe pas déjà
        const existingUser = await usersCollection.findOne({ email });
        if (existingUser) {
            return res.status(409).json({
                error: 'Un utilisateur avec cet email existe déjà'
            });
        }

        // Créer l'utilisateur
        const newUser = {
            name,
            email,
            age: age || null,
            createdAt: new Date()
        };

        const result = await usersCollection.insertOne(newUser);

        res.status(201).json({
            message: 'Utilisateur créé avec succès',
            userId: result.insertedId,
            user: { _id: result.insertedId, ...newUser }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// PUT /api/users/:id - Mettre à jour un utilisateur
app.put('/api/users/:id', async (req, res) => {
    try {
        const { name, email, age } = req.body;

        const updateDoc = {};
        if (name) updateDoc.name = name;
        if (email) updateDoc.email = email;
        if (age !== undefined) updateDoc.age = age;

        const result = await usersCollection.updateOne(
            { _id: new ObjectId(req.params.id) },
            { $set: updateDoc }
        );

        if (result.matchedCount === 0) {
            return res.status(404).json({ error: 'Utilisateur non trouvé' });
        }

        res.json({ message: 'Utilisateur mis à jour', modifiedCount: result.modifiedCount });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// DELETE /api/users/:id - Supprimer un utilisateur
app.delete('/api/users/:id', async (req, res) => {
    try {
        const result = await usersCollection.deleteOne({
            _id: new ObjectId(req.params.id)
        });

        if (result.deletedCount === 0) {
            return res.status(404).json({ error: 'Utilisateur non trouvé' });
        }

        res.json({ message: 'Utilisateur supprimé' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ==========================================
// ROUTES - TÂCHES (TASKS)
// ==========================================

// GET /api/tasks - Récupérer toutes les tâches
app.get('/api/tasks', async (req, res) => {
    try {
        const { userId, completed } = req.query;

        // Filtres optionnels
        const filter = {};
        if (userId) filter.userId = new ObjectId(userId);
        if (completed !== undefined) filter.completed = completed === 'true';

        const tasks = await tasksCollection.find(filter).toArray();
        res.json(tasks);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// GET /api/tasks/:id - Récupérer une tâche par ID
app.get('/api/tasks/:id', async (req, res) => {
    try {
        const task = await tasksCollection.findOne({
            _id: new ObjectId(req.params.id)
        });

        if (!task) {
            return res.status(404).json({ error: 'Tâche non trouvée' });
        }

        res.json(task);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// POST /api/tasks - Créer une nouvelle tâche
app.post('/api/tasks', async (req, res) => {
    try {
        const { title, description, userId } = req.body;

        if (!title) {
            return res.status(400).json({ error: 'Le champ title est requis' });
        }

        const newTask = {
            title,
            description: description || '',
            completed: false,
            userId: userId ? new ObjectId(userId) : null,
            createdAt: new Date()
        };

        const result = await tasksCollection.insertOne(newTask);

        res.status(201).json({
            message: 'Tâche créée avec succès',
            taskId: result.insertedId,
            task: { _id: result.insertedId, ...newTask }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// PATCH /api/tasks/:id - Marquer une tâche comme complétée/non complétée
app.patch('/api/tasks/:id', async (req, res) => {
    try {
        const { completed } = req.body;

        if (completed === undefined) {
            return res.status(400).json({ error: 'Le champ completed est requis' });
        }

        const result = await tasksCollection.updateOne(
            { _id: new ObjectId(req.params.id) },
            { $set: { completed } }
        );

        if (result.matchedCount === 0) {
            return res.status(404).json({ error: 'Tâche non trouvée' });
        }

        res.json({ message: 'Tâche mise à jour', modifiedCount: result.modifiedCount });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// DELETE /api/tasks/:id - Supprimer une tâche
app.delete('/api/tasks/:id', async (req, res) => {
    try {
        const result = await tasksCollection.deleteOne({
            _id: new ObjectId(req.params.id)
        });

        if (result.deletedCount === 0) {
            return res.status(404).json({ error: 'Tâche non trouvée' });
        }

        res.json({ message: 'Tâche supprimée' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ==========================================
// ROUTE 404 - Route non trouvée
// ==========================================

app.use((req, res) => {
    res.status(404).json({
        error: 'Route non trouvée',
        method: req.method,
        url: req.url
    });
});

// ==========================================
// DÉMARRAGE DU SERVEUR
// ==========================================

async function startServer() {
    // Attendre la connexion à MongoDB
    await connectToMongoDB();

    // Démarrer le serveur Express
    app.listen(PORT, () => {
        console.log(`🚀 Serveur API démarré sur le port ${PORT}`);
        console.log(`📡 Endpoints disponibles :`);
        console.log(`   - GET  /api/users`);
        console.log(`   - POST /api/users`);
        console.log(`   - GET  /api/tasks`);
        console.log(`   - POST /api/tasks`);
        console.log(`   - GET  /api/health`);
    });
}

// Démarrer l'application
startServer().catch(console.error);
```

### 3.3 Créer le Dockerfile backend

Créez le fichier `backend/Dockerfile` :

```dockerfile
# Image Node.js officielle
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm install --production

# Copier le code source
COPY . .

# Exposer le port 3000
EXPOSE 3000

# Commande de démarrage
CMD ["npm", "start"]
```

### 3.4 Créer le .dockerignore

Créez le fichier `backend/.dockerignore` :

```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
```

---

## 🎨 Étape 4 : Frontend (Angular)

### 4.1 Installer Angular CLI

Si vous n'avez pas Angular CLI installé :

```bash
# Installer Angular CLI globalement
npm install -g @angular/cli

# Vérifier l'installation
ng version
```

### 4.2 Créer l'application Angular

Depuis le dossier `mean-stack` :

```bash
# Créer une nouvelle application Angular dans le dossier frontend
ng new frontend --routing --style=css --skip-git

# Répondre aux questions :
# ? Would you like to add Angular routing? Yes
# ? Which stylesheet format would you like to use? CSS
```

**💡 Note :** Angular CLI va créer tous les fichiers nécessaires dans `frontend/`.

### 4.3 Créer le service API

Créez le fichier `frontend/src/app/api.service.ts` :

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

// Interface pour un utilisateur
export interface User {
  _id?: string;
  name: string;
  email: string;
  age?: number;
  createdAt?: Date;
}

// Interface pour une tâche
export interface Task {
  _id?: string;
  title: string;
  description?: string;
  completed: boolean;
  userId?: string;
  createdAt?: Date;
}

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  // URL de l'API backend
  private apiUrl = 'http://localhost:3000/api';

  constructor(private http: HttpClient) { }

  // ==========================================
  // MÉTHODES UTILISATEURS
  // ==========================================

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`);
  }

  getUser(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`);
  }

  createUser(user: User): Observable<any> {
    return this.http.post(`${this.apiUrl}/users`, user);
  }

  updateUser(id: string, user: Partial<User>): Observable<any> {
    return this.http.put(`${this.apiUrl}/users/${id}`, user);
  }

  deleteUser(id: string): Observable<any> {
    return this.http.delete(`${this.apiUrl}/users/${id}`);
  }

  // ==========================================
  // MÉTHODES TÂCHES
  // ==========================================

  getTasks(): Observable<Task[]> {
    return this.http.get<Task[]>(`${this.apiUrl}/tasks`);
  }

  getTask(id: string): Observable<Task> {
    return this.http.get<Task>(`${this.apiUrl}/tasks/${id}`);
  }

  createTask(task: Task): Observable<any> {
    return this.http.post(`${this.apiUrl}/tasks`, task);
  }

  toggleTaskCompleted(id: string, completed: boolean): Observable<any> {
    return this.http.patch(`${this.apiUrl}/tasks/${id}`, { completed });
  }

  deleteTask(id: string): Observable<any> {
    return this.http.delete(`${this.apiUrl}/tasks/${id}`);
  }

  // ==========================================
  // MÉTHODE HEALTH CHECK
  // ==========================================

  checkHealth(): Observable<any> {
    return this.http.get(`${this.apiUrl}/health`);
  }
}
```

### 4.4 Créer le composant principal

Modifiez le fichier `frontend/src/app/app.component.ts` :

```typescript
import { Component, OnInit } from '@angular/core';
import { ApiService, User, Task } from './api.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  title = 'Stack MEAN avec Docker';

  // Données
  users: User[] = [];
  tasks: Task[] = [];

  // État de chargement
  loading = true;
  error: string | null = null;

  // Formulaire nouveau utilisateur
  newUser: User = { name: '', email: '' };

  // Formulaire nouvelle tâche
  newTask: Task = { title: '', description: '', completed: false };

  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.loadData();
  }

  // Charger toutes les données
  loadData(): void {
    this.loading = true;
    this.error = null;

    // Charger les utilisateurs
    this.apiService.getUsers().subscribe({
      next: (users) => {
        this.users = users;
        this.loading = false;
      },
      error: (err) => {
        this.error = 'Erreur de connexion à l\'API backend';
        this.loading = false;
        console.error(err);
      }
    });

    // Charger les tâches
    this.apiService.getTasks().subscribe({
      next: (tasks) => {
        this.tasks = tasks;
      },
      error: (err) => {
        console.error(err);
      }
    });
  }

  // Ajouter un utilisateur
  addUser(): void {
    if (this.newUser.name && this.newUser.email) {
      this.apiService.createUser(this.newUser).subscribe({
        next: () => {
          this.loadData(); // Recharger les données
          this.newUser = { name: '', email: '' }; // Réinitialiser le formulaire
        },
        error: (err) => {
          alert('Erreur lors de la création de l\'utilisateur');
          console.error(err);
        }
      });
    }
  }

  // Supprimer un utilisateur
  deleteUser(id: string): void {
    if (confirm('Voulez-vous vraiment supprimer cet utilisateur ?')) {
      this.apiService.deleteUser(id).subscribe({
        next: () => {
          this.loadData();
        },
        error: (err) => {
          alert('Erreur lors de la suppression');
          console.error(err);
        }
      });
    }
  }

  // Ajouter une tâche
  addTask(): void {
    if (this.newTask.title) {
      this.apiService.createTask(this.newTask).subscribe({
        next: () => {
          this.loadData();
          this.newTask = { title: '', description: '', completed: false };
        },
        error: (err) => {
          alert('Erreur lors de la création de la tâche');
          console.error(err);
        }
      });
    }
  }

  // Basculer l'état d'une tâche
  toggleTask(task: Task): void {
    if (task._id) {
      this.apiService.toggleTaskCompleted(task._id, !task.completed).subscribe({
        next: () => {
          this.loadData();
        },
        error: (err) => {
          console.error(err);
        }
      });
    }
  }

  // Supprimer une tâche
  deleteTask(id: string): void {
    if (confirm('Voulez-vous vraiment supprimer cette tâche ?')) {
      this.apiService.deleteTask(id).subscribe({
        next: () => {
          this.loadData();
        },
        error: (err) => {
          alert('Erreur lors de la suppression');
          console.error(err);
        }
      });
    }
  }

  // Récupérer le nom d'un utilisateur par son ID
  getUserName(userId: string): string {
    const user = this.users.find(u => u._id === userId);
    return user ? user.name : 'Inconnu';
  }
}
```

### 4.5 Créer le template HTML

Modifiez le fichier `frontend/src/app/app.component.html` :

```html
<div class="container">
  <header>
    <h1>🚀 {{ title }}</h1>
    <p class="subtitle">MongoDB + Express + Angular + Node.js</p>
  </header>

  <!-- Message d'erreur -->
  <div *ngIf="error" class="alert alert-error">
    ❌ {{ error }}
  </div>

  <!-- Chargement -->
  <div *ngIf="loading" class="loading">
    ⏳ Chargement des données...
  </div>

  <!-- Contenu principal -->
  <div *ngIf="!loading && !error">

    <!-- Section Utilisateurs -->
    <section class="card">
      <h2>👥 Utilisateurs ({{ users.length }})</h2>

      <!-- Formulaire ajout utilisateur -->
      <div class="form-group">
        <h3>➕ Ajouter un utilisateur</h3>
        <input
          type="text"
          [(ngModel)]="newUser.name"
          placeholder="Nom complet"
          class="input">
        <input
          type="email"
          [(ngModel)]="newUser.email"
          placeholder="Email"
          class="input">
        <input
          type="number"
          [(ngModel)]="newUser.age"
          placeholder="Âge (optionnel)"
          class="input">
        <button (click)="addUser()" class="btn btn-primary">
          Créer l'utilisateur
        </button>
      </div>

      <!-- Liste des utilisateurs -->
      <div class="user-list">
        <div *ngFor="let user of users" class="user-item">
          <div class="user-info">
            <strong>{{ user.name }}</strong>
            <span class="email">{{ user.email }}</span>
            <span *ngIf="user.age" class="age">{{ user.age }} ans</span>
          </div>
          <button (click)="deleteUser(user._id!)" class="btn btn-danger btn-sm">
            🗑️ Supprimer
          </button>
        </div>
      </div>
    </section>

    <!-- Section Tâches -->
    <section class="card">
      <h2>📝 Tâches ({{ tasks.length }})</h2>

      <!-- Formulaire ajout tâche -->
      <div class="form-group">
        <h3>➕ Ajouter une tâche</h3>
        <input
          type="text"
          [(ngModel)]="newTask.title"
          placeholder="Titre de la tâche"
          class="input">
        <textarea
          [(ngModel)]="newTask.description"
          placeholder="Description (optionnel)"
          class="input"
          rows="3"></textarea>
        <button (click)="addTask()" class="btn btn-primary">
          Créer la tâche
        </button>
      </div>

      <!-- Liste des tâches -->
      <div class="task-list">
        <div
          *ngFor="let task of tasks"
          class="task-item"
          [class.completed]="task.completed">

          <input
            type="checkbox"
            [checked]="task.completed"
            (change)="toggleTask(task)">

          <div class="task-info">
            <strong>{{ task.title }}</strong>
            <p *ngIf="task.description">{{ task.description }}</p>
            <span class="task-user" *ngIf="task.userId">
              👤 {{ getUserName(task.userId) }}
            </span>
          </div>

          <button (click)="deleteTask(task._id!)" class="btn btn-danger btn-sm">
            🗑️
          </button>
        </div>
      </div>

      <!-- Statistiques -->
      <div class="stats">
        <span class="badge">
          ✅ {{ (tasks | filter:'completed':true).length }} complétées
        </span>
        <span class="badge">
          ⏳ {{ (tasks | filter:'completed':false).length }} en cours
        </span>
      </div>
    </section>

  </div>

  <footer>
    <p>🐳 Stack MEAN avec Docker - Guide pour développeurs</p>
  </footer>
</div>
```

### 4.6 Ajouter les styles CSS

Modifiez le fichier `frontend/src/app/app.component.css` :

```css
/* Variables CSS */
:root {
  --primary-color: #667eea;
  --secondary-color: #764ba2;
  --success-color: #48bb78;
  --danger-color: #f56565;
  --gray-100: #f7fafc;
  --gray-200: #edf2f7;
  --gray-700: #4a5568;
  --gray-900: #1a202c;
}

/* Reset et base */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
  background: linear-gradient(135deg, var(--primary-color) 0%, var(--secondary-color) 100%);
  color: var(--gray-900);
  min-height: 100vh;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* Header */
header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
}

header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.subtitle {
  font-size: 1.2rem;
  opacity: 0.9;
}

/* Cartes */
.card {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
}

.card h2 {
  color: var(--primary-color);
  margin-bottom: 20px;
  font-size: 1.8rem;
}

.card h3 {
  color: var(--gray-700);
  margin-bottom: 15px;
  font-size: 1.2rem;
}

/* Formulaires */
.form-group {
  background: var(--gray-100);
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.input {
  width: 100%;
  padding: 12px;
  margin-bottom: 10px;
  border: 2px solid var(--gray-200);
  border-radius: 6px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.input:focus {
  outline: none;
  border-color: var(--primary-color);
}

/* Boutons */
.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background: var(--secondary-color);
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-danger {
  background: var(--danger-color);
  color: white;
}

.btn-danger:hover {
  background: #e53e3e;
}

.btn-sm {
  padding: 8px 16px;
  font-size: 0.9rem;
}

/* Liste utilisateurs */
.user-list {
  display: grid;
  gap: 15px;
}

.user-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: var(--gray-100);
  border-radius: 8px;
  transition: transform 0.2s;
}

.user-item:hover {
  transform: translateX(5px);
}

.user-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.user-info strong {
  color: var(--gray-900);
  font-size: 1.1rem;
}

.email {
  color: var(--gray-700);
  font-size: 0.9rem;
}

.age {
  color: var(--primary-color);
  font-size: 0.85rem;
  font-weight: 600;
}

/* Liste tâches */
.task-list {
  display: grid;
  gap: 15px;
}

.task-item {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  padding: 15px;
  background: var(--gray-100);
  border-radius: 8px;
  transition: all 0.3s;
}

.task-item:hover {
  background: var(--gray-200);
}

.task-item.completed {
  opacity: 0.6;
}

.task-item.completed .task-info strong {
  text-decoration: line-through;
}

.task-item input[type="checkbox"] {
  width: 20px;
  height: 20px;
  cursor: pointer;
  flex-shrink: 0;
  margin-top: 2px;
}

.task-info {
  flex: 1;
}

.task-info strong {
  display: block;
  color: var(--gray-900);
  font-size: 1.1rem;
  margin-bottom: 5px;
}

.task-info p {
  color: var(--gray-700);
  font-size: 0.9rem;
  margin-bottom: 8px;
}

.task-user {
  display: inline-block;
  color: var(--primary-color);
  font-size: 0.85rem;
  font-weight: 600;
}

/* Statistiques */
.stats {
  display: flex;
  gap: 15px;
  margin-top: 20px;
  padding-top: 20px;
  border-top: 2px solid var(--gray-200);
}

.badge {
  padding: 8px 16px;
  background: var(--primary-color);
  color: white;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 600;
}

/* Alertes */
.alert {
  padding: 15px 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  font-weight: 500;
}

.alert-error {
  background: #fed7d7;
  color: #c53030;
  border: 2px solid #fc8181;
}

/* Chargement */
.loading {
  text-align: center;
  padding: 40px;
  font-size: 1.2rem;
  color: white;
}

/* Footer */
footer {
  text-align: center;
  color: white;
  padding: 20px 0;
  margin-top: 40px;
  opacity: 0.9;
}

/* Responsive */
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }

  header h1 {
    font-size: 2rem;
  }

  .card {
    padding: 20px;
  }

  .user-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }

  .stats {
    flex-direction: column;
  }
}
```

### 4.7 Configurer app.module.ts

Modifiez le fichier `frontend/src/app/app.module.ts` :

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { ApiService } from './api.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule,  // Pour les requêtes HTTP
    FormsModule        // Pour ngModel (two-way binding)
  ],
  providers: [ApiService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 4.8 Créer le Dockerfile frontend

Créez le fichier `frontend/Dockerfile` :

```dockerfile
# Stage 1: Build Angular
FROM node:18-alpine AS build

WORKDIR /app

# Copier package.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le code source
COPY . .

# Build production
RUN npm run build -- --configuration production

# Stage 2: Servir avec Nginx
FROM nginx:alpine

# Copier les fichiers buildés
COPY --from=build /app/dist/frontend /usr/share/nginx/html

# Copier la configuration Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 4.9 Configuration Nginx

Créez le fichier `frontend/nginx.conf` :

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gestion du routing Angular
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache des assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 🐳 Étape 5 : Configuration Docker Compose

### 5.1 Créer le docker-compose.yml

Créez le fichier `docker-compose.yml` à la racine du projet :

```yaml
version: '3.8'

services:
  # ==========================================
  # SERVICE 1 : BASE DE DONNÉES MONGODB
  # ==========================================
  mongodb:
    image: mongo:7.0
    container_name: mean_mongodb
    restart: unless-stopped

    environment:
      # Pas d'authentification pour le développement
      # En production, configurez MONGO_INITDB_ROOT_USERNAME et MONGO_INITDB_ROOT_PASSWORD
      MONGO_INITDB_DATABASE: mean_db

    ports:
      # Accès direct à MongoDB depuis l'hôte
      - "27017:27017"

    volumes:
      # Données persistantes
      - ./mongodb/data:/data/db

      # Script d'initialisation
      - ./mongodb/init/init-db.js:/docker-entrypoint-initdb.d/init-db.js:ro

    networks:
      - mean_network

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 2 : BACKEND API (Node.js + Express)
  # ==========================================
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: mean_backend
    restart: unless-stopped

    # Attend que MongoDB soit prêt
    depends_on:
      mongodb:
        condition: service_healthy

    ports:
      # API accessible sur http://localhost:3000
      - "3000:3000"

    environment:
      # URL de connexion à MongoDB
      MONGO_URL: mongodb://mongodb:27017
      NODE_ENV: development

    volumes:
      # Hot reload : modifications du code prises en compte immédiatement
      - ./backend/src:/app/src

    networks:
      - mean_network

  # ==========================================
  # SERVICE 3 : FRONTEND (Angular)
  # ==========================================
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: mean_frontend
    restart: unless-stopped

    depends_on:
      - backend

    ports:
      # Application accessible sur http://localhost:4200 (ou 80)
      - "4200:80"

    networks:
      - mean_network

# ==========================================
# RÉSEAU DOCKER PARTAGÉ
# ==========================================
networks:
  mean_network:
    driver: bridge
```

---

## ▶️ Étape 6 : Démarrer la stack

### 6.1 Build et démarrage

Depuis le dossier `mean-stack` :

```bash
# Construire les images Docker
docker-compose build

# Démarrer tous les services
docker-compose up -d
```

**Ce qui se passe :**
1. Construction de l'image backend (Node.js)
2. Construction de l'image frontend (Angular)
3. Téléchargement de l'image MongoDB
4. Création du réseau `mean_network`
5. Démarrage de MongoDB
6. Exécution du script d'initialisation MongoDB
7. Démarrage du backend (attend MongoDB)
8. Démarrage du frontend

### 6.2 Vérifier que tout fonctionne

```bash
# Voir les conteneurs actifs
docker-compose ps

# Résultat attendu :
# NAME             STATE     PORTS
# mean_mongodb     Up        27017/tcp
# mean_backend     Up        0.0.0.0:3000->3000/tcp
# mean_frontend    Up        0.0.0.0:4200->80/tcp
```

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Logs d'un service spécifique
docker-compose logs backend
```

---

## 🌐 Étape 7 : Tester l'application

### 7.1 Accéder au frontend

Ouvrez votre navigateur et allez sur :

```
http://localhost:4200
```

**Vous devriez voir :**
- ✅ L'interface Angular avec le titre "Stack MEAN avec Docker"
- 👥 La liste des 3 utilisateurs de test
- 📝 La liste des 4 tâches de test
- Formulaires pour ajouter des utilisateurs et des tâches

### 7.2 Tester l'API backend directement

Vous pouvez tester l'API avec curl ou Postman :

```bash
# Health check
curl http://localhost:3000/api/health

# Liste des utilisateurs
curl http://localhost:3000/api/users

# Liste des tâches
curl http://localhost:3000/api/tasks

# Créer un nouvel utilisateur (POST)
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","age":30}'
```

### 7.3 Accéder à MongoDB directement

```bash
# Se connecter au shell MongoDB
docker exec -it mean_mongodb mongosh

# Une fois dans le shell MongoDB
use mean_db

# Voir les collections
show collections

# Afficher tous les utilisateurs
db.users.find().pretty()

# Afficher toutes les tâches
db.tasks.find().pretty()

# Quitter
exit
```

---

## 🛠️ Étape 8 : Développement

### 8.1 Modifier le backend en temps réel

**Avantage du volume mount :**
- Vous modifiez `backend/src/server.js` sur votre machine
- **Les changements sont immédiats** (si vous utilisez `nodemon`)

**Pour activer le hot reload du backend :**

Modifiez `backend/package.json` :

```json
"scripts": {
  "start": "nodemon src/server.js"
}
```

Installez nodemon dans le backend :

```bash
cd backend
npm install --save-dev nodemon
cd ..
```

Reconstruisez et redémarrez :

```bash
docker-compose build backend
docker-compose restart backend
```

### 8.2 Modifier le frontend

Pour développer Angular en local (avec hot reload) :

```bash
# Arrêter le conteneur frontend
docker-compose stop frontend

# Développer en local
cd frontend
ng serve

# L'application sera accessible sur http://localhost:4200
# Avec hot reload automatique
```

Pour remettre en production Docker :

```bash
docker-compose start frontend
```

### 8.3 Ajouter une nouvelle route API

**Exemple : Statistiques**

Dans `backend/src/server.js`, ajoutez :

```javascript
// GET /api/stats - Statistiques globales
app.get('/api/stats', async (req, res) => {
    try {
        const usersCount = await usersCollection.countDocuments();
        const tasksCount = await tasksCollection.countDocuments();
        const completedTasks = await tasksCollection.countDocuments({ completed: true });

        res.json({
            users: usersCount,
            tasks: {
                total: tasksCount,
                completed: completedTasks,
                pending: tasksCount - completedTasks
            }
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

Testez :

```bash
curl http://localhost:3000/api/stats
```

---

## 📊 Étape 9 : Monitoring

### 9.1 Commandes utiles

```bash
# Voir les conteneurs actifs
docker-compose ps

# Logs de tous les services
docker-compose logs

# Logs en temps réel d'un service
docker-compose logs -f backend

# Statistiques de ressources
docker stats mean_mongodb mean_backend mean_frontend

# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer tous les services
docker-compose start

# Redémarrer un service spécifique
docker-compose restart backend
```

### 9.2 Accéder aux conteneurs

```bash
# Shell dans le conteneur backend
docker exec -it mean_backend sh

# Une fois dedans :
ls -la /app/src/
cat /app/src/server.js
exit

# Shell dans le conteneur MongoDB
docker exec -it mean_mongodb mongosh

# Une fois dedans :
show dbs
use mean_db
show collections
exit
```

---

## 🛑 Étape 10 : Arrêt et nettoyage

### 10.1 Arrêter proprement

```bash
# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer plus tard
docker-compose start
```

### 10.2 Suppression complète

```bash
# Arrêter et supprimer les conteneurs
docker-compose down

# Supprimer également les volumes (⚠️ SUPPRIME LES DONNÉES)
docker-compose down -v

# Supprimer les images construites
docker rmi mean-stack-backend mean-stack-frontend

# Supprimer les données MongoDB manuellement
rm -rf mongodb/data/*
```

---

## 🐛 Dépannage

### Problème 1 : "Cannot connect to MongoDB"

**Symptôme :** Le backend ne peut pas se connecter à MongoDB.

**Solutions :**

1. **Vérifier que MongoDB est démarré :**
   ```bash
   docker-compose ps
   # mean_mongodb doit être "Up (healthy)"
   ```

2. **Vérifier les logs MongoDB :**
   ```bash
   docker-compose logs mongodb
   ```

3. **Attendre que MongoDB soit prêt :**
   - MongoDB peut mettre 10-20 secondes à être complètement prêt
   - Le healthcheck garantit que le backend attend

4. **Vérifier le réseau :**
   ```bash
   docker network inspect mean-stack_mean_network
   ```

### Problème 2 : "CORS policy error" dans le navigateur

**Symptôme :** Erreurs CORS dans la console du navigateur.

**Solutions :**

1. **Vérifier la configuration CORS dans server.js :**
   ```javascript
   app.use(cors({
       origin: ['http://localhost:4200', 'http://localhost:80'],
       methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH']
   }));
   ```

2. **Vérifier l'URL de l'API dans Angular :**
   - Dans `api.service.ts`, assurez-vous que `apiUrl` pointe vers `http://localhost:3000/api`

3. **Redémarrer le backend :**
   ```bash
   docker-compose restart backend
   ```

### Problème 3 : "Port 3000 already in use"

**Symptôme :** Le backend ne démarre pas car le port est utilisé.

**Solutions :**

1. **Changer le port dans docker-compose.yml :**
   ```yaml
   backend:
     ports:
       - "3001:3000"  # Utilisez le port 3001 au lieu de 3000
   ```

2. **Mettre à jour l'URL API dans Angular :**
   ```typescript
   private apiUrl = 'http://localhost:3001/api';
   ```

### Problème 4 : Frontend ne se build pas

**Symptôme :** Erreur lors du `docker-compose build`.

**Solutions :**

1. **Builder Angular localement d'abord :**
   ```bash
   cd frontend
   npm install
   ng build --configuration production
   cd ..
   docker-compose build frontend
   ```

2. **Vérifier que Node.js est installé :**
   ```bash
   node --version
   npm --version
   ```

3. **Augmenter la mémoire Node.js :**
   Dans `frontend/package.json` :
   ```json
   "scripts": {
     "build": "node --max_old_space_size=4096 node_modules/@angular/cli/bin/ng build"
   }
   ```

### Problème 5 : "Cannot GET /api/..."

**Symptôme :** 404 sur les routes API.

**Solutions :**

1. **Vérifier que le backend est démarré :**
   ```bash
   docker-compose logs backend
   ```

2. **Tester directement l'API :**
   ```bash
   curl http://localhost:3000/api/users
   ```

3. **Vérifier les routes dans server.js**

---

## ✅ Récapitulatif

### Ce que vous avez appris

- ✅ Comprendre l'architecture d'une stack MEAN
- ✅ Déployer MongoDB, Express, Angular et Node.js avec Docker Compose
- ✅ Créer une API REST complète avec Express
- ✅ Développer un frontend Angular moderne
- ✅ Faire communiquer frontend et backend
- ✅ Gérer des données NoSQL avec MongoDB
- ✅ Utiliser Docker pour un environnement full-stack
- ✅ Hot reload pour le développement

### Technologies maîtrisées

| Technologie | Rôle | Version |
|-------------|------|---------|
| MongoDB | Base de données NoSQL | 7.0 |
| Express | Framework backend | 4.18 |
| Angular | Framework frontend | 17+ |
| Node.js | Runtime JavaScript | 18 |
| Docker | Conteneurisation | 20.10+ |

### Fichiers créés

```
mean-stack/
├── docker-compose.yml                 # Orchestration Docker
├── backend/
│   ├── Dockerfile                     # Image backend
│   ├── package.json                   # Dépendances Node.js
│   ├── .dockerignore                  # Exclusions Docker
│   └── src/
│       └── server.js                  # API Express
├── frontend/
│   ├── Dockerfile                     # Image frontend
│   ├── nginx.conf                     # Configuration Nginx
│   └── src/
│       ├── app/
│       │   ├── api.service.ts         # Service API
│       │   ├── app.component.ts       # Composant principal
│       │   ├── app.component.html     # Template
│       │   ├── app.component.css      # Styles
│       │   └── app.module.ts          # Module Angular
│       └── ...
└── mongodb/
    ├── data/                          # Données persistantes
    └── init/
        └── init-db.js                 # Script d'init
```

### Commandes essentielles

```bash
# Construire et démarrer
docker-compose build
docker-compose up -d

# Voir les logs
docker-compose logs -f

# Arrêter (données conservées)
docker-compose stop

# Redémarrer
docker-compose start

# Supprimer tout
docker-compose down -v

# Accéder à MongoDB
docker exec -it mean_mongodb mongosh

# Développement Angular local
cd frontend && ng serve
```

---

## 🚀 Pour aller plus loin

### Extensions possibles

1. **Ajouter l'authentification JWT** (JSON Web Tokens)
2. **Implémenter le CRUD complet** (Create, Read, Update, Delete)
3. **Ajouter des tests** (Jest pour backend, Jasmine pour Angular)
4. **Mettre en place Mongo Express** (interface web MongoDB)
5. **Configurer HTTPS** avec Let's Encrypt
6. **Ajouter Redis** pour le cache
7. **Implémenter des WebSockets** pour le temps réel
8. **Déployer sur un cloud** (AWS, Azure, Google Cloud)

### Frameworks alternatifs

Cette stack peut utiliser :
- **React** au lieu d'Angular (MERN stack)
- **Vue.js** au lieu d'Angular (MEVN stack)
- **NestJS** au lieu d'Express (architecture plus structurée)
- **Fastify** au lieu d'Express (plus performant)

### Ressources complémentaires

- 📖 [Documentation MongoDB](https://www.mongodb.com/docs/)
- 📖 [Documentation Express](https://expressjs.com/)
- 📖 [Documentation Angular](https://angular.io/docs)
- 📖 [Documentation Node.js](https://nodejs.org/docs/)
- 🎓 [Tutoriel MEAN Stack](https://www.mongodb.com/languages/mean-stack-tutorial)

---

## 🎉 Félicitations !

Vous avez maintenant une **stack MEAN complète et fonctionnelle** ! Cette base vous permet de :

- 🌐 Développer des applications web modernes full-stack
- 🗄️ Gérer des données flexibles avec MongoDB
- 🚀 Utiliser JavaScript de bout en bout
- 🐳 Déployer rapidement avec Docker
- 💪 Créer des APIs REST performantes

**Prochain pas :** Explorez les autres stacks du guide !

➡️ [Stack LAMP (Apache + MariaDB + PHP)](01-stack-lamp.md)
➡️ [Stack ELK (Elasticsearch + Logstash + Kibana)](03-stack-elk.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
