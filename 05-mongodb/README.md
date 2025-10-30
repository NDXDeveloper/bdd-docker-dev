# 5. MongoDB - Guide complet avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📖 Introduction à MongoDB

**MongoDB** est une base de données NoSQL orientée documents, open source et très populaire. Contrairement aux bases de données relationnelles traditionnelles (SQL), MongoDB stocke les données sous forme de **documents JSON flexibles**, ce qui facilite le développement d'applications modernes.

### 🤔 Pourquoi MongoDB ?

MongoDB est particulièrement adapté pour :

- ✅ **Applications web modernes** avec structure de données évolutive
- ✅ **Prototypage rapide** grâce à un schéma flexible
- ✅ **Big Data** et analytics avec de gros volumes
- ✅ **Applications temps réel** (chat, notifications, IoT)
- ✅ **Gestion de contenu** (CMS, blogs, e-commerce)
- ✅ **Stockage de logs** et événements
- ✅ **Applications mobiles** avec synchronisation

---

## 🆚 MongoDB vs Bases de données relationnelles

### Différences fondamentales

| Concept | SQL (MySQL, PostgreSQL) | MongoDB |
|---------|-------------------------|---------|
| **Structure** | Tables avec lignes et colonnes | Collections avec documents JSON |
| **Schéma** | Rigide, défini à l'avance | Flexible, évolutif |
| **Relations** | Clés étrangères, JOIN | Documents imbriqués ou références |
| **Scalabilité** | Verticale (serveur plus puissant) | Horizontale (plus de serveurs) |
| **Transactions** | ACID natif | ACID depuis la version 4.0 |
| **Langage** | SQL (SELECT, INSERT...) | JavaScript-like (find, insertOne...) |

### Exemple de comparaison

**SQL (Table "users") :**
```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  nom VARCHAR(50),
  prenom VARCHAR(50),
  email VARCHAR(100),
  age INT
);

INSERT INTO users VALUES (1, 'Dupont', 'Jean', 'jean@example.com', 30);
```

**MongoDB (Collection "users") :**
```javascript
// Pas besoin de créer la structure à l'avance
db.users.insertOne({
  nom: "Dupont",
  prenom: "Jean",
  email: "jean@example.com",
  age: 30,
  // On peut ajouter d'autres champs librement
  adresses: [
    { type: "domicile", ville: "Paris" },
    { type: "travail", ville: "Lyon" }
  ]
})
```

**Avantage MongoDB :** Structure imbriquée naturelle sans JOIN !

---

## 🏗️ Architecture de MongoDB

### Structure hiérarchique

```
MongoDB Server
│
├── Base de données 1 (ex: "ecommerce")
│   ├── Collection 1 (ex: "produits")
│   │   ├── Document 1 { _id: 1, nom: "Laptop", prix: 999 }
│   │   ├── Document 2 { _id: 2, nom: "Souris", prix: 29 }
│   │   └── Document 3 { _id: 3, nom: "Clavier", prix: 79 }
│   │
│   └── Collection 2 (ex: "clients")
│       ├── Document 1 { _id: 1, nom: "Alice", email: "alice@..." }
│       └── Document 2 { _id: 2, nom: "Bob", email: "bob@..." }
│
└── Base de données 2 (ex: "blog")
    ├── Collection 1 (ex: "articles")
    └── Collection 2 (ex: "commentaires")
```

### Vocabulaire MongoDB

| Terme SQL | Équivalent MongoDB | Description |
|-----------|-------------------|-------------|
| **Database** | **Database** | Conteneur de collections |
| **Table** | **Collection** | Groupe de documents similaires |
| **Row** | **Document** | Enregistrement JSON/BSON |
| **Column** | **Field** | Champ d'un document |
| **Primary Key** | **_id** | Identifiant unique automatique |
| **Index** | **Index** | Améliore les performances de recherche |
| **JOIN** | **Embedded/Reference** | Documents imbriqués ou liés |

---

## 📄 Format des documents MongoDB

MongoDB utilise le format **BSON** (Binary JSON) en interne, mais vous travaillez avec du **JSON** standard.

### Structure d'un document

```javascript
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),  // ID unique auto-généré
  "nom": "Dupont",
  "prenom": "Jean",
  "age": 30,
  "email": "jean.dupont@example.com",
  "actif": true,
  "dateInscription": ISODate("2024-01-15T10:30:00Z"),
  "tags": ["développeur", "fullstack", "mongodb"],
  "adresse": {
    "rue": "123 rue de la Paix",
    "ville": "Paris",
    "codePostal": "75001"
  },
  "competences": [
    { "nom": "JavaScript", "niveau": 5 },
    { "nom": "Python", "niveau": 4 },
    { "nom": "Docker", "niveau": 3 }
  ]
}
```

**Caractéristiques :**
- ✅ **Schéma flexible** : chaque document peut avoir des champs différents
- ✅ **Types riches** : nombres, strings, dates, tableaux, objets imbriqués
- ✅ **_id automatique** : clé primaire générée si non fournie
- ✅ **Taille limite** : 16 Mo par document (largement suffisant)

---

## 🔑 Concepts clés à connaître

### 1. Collections

Les **collections** regroupent des documents similaires (comme les tables SQL).

```javascript
// Créer une collection (implicite lors de l'insertion)
db.createCollection("users")

// Lister les collections
show collections

// Supprimer une collection
db.users.drop()
```

### 2. Documents

Les **documents** sont les enregistrements individuels en JSON.

```javascript
// Insérer un document
db.users.insertOne({ nom: "Martin", age: 25 })

// Insérer plusieurs documents
db.users.insertMany([
  { nom: "Sophie", age: 28 },
  { nom: "Luc", age: 35 }
])
```

### 3. Requêtes

Rechercher des documents avec des **critères** flexibles.

```javascript
// Trouver tous les documents
db.users.find()

// Trouver avec critère
db.users.find({ age: { $gt: 25 } })  // age > 25

// Trouver un seul document
db.users.findOne({ nom: "Martin" })

// Compter
db.users.countDocuments({ age: { $gt: 25 } })
```

### 4. Opérateurs de requête

| Opérateur | Signification | Exemple |
|-----------|---------------|---------|
| `$eq` | Égal | `{ age: { $eq: 30 } }` |
| `$gt` | Supérieur à | `{ age: { $gt: 25 } }` |
| `$gte` | Supérieur ou égal | `{ age: { $gte: 25 } }` |
| `$lt` | Inférieur à | `{ age: { $lt: 40 } }` |
| `$lte` | Inférieur ou égal | `{ age: { $lte: 40 } }` |
| `$ne` | Différent de | `{ age: { $ne: 30 } }` |
| `$in` | Dans la liste | `{ age: { $in: [25, 30, 35] } }` |
| `$nin` | Pas dans la liste | `{ age: { $nin: [25, 30] } }` |
| `$and` | ET logique | `{ $and: [{ age: { $gt: 25 } }, { nom: "Martin" }] }` |
| `$or` | OU logique | `{ $or: [{ age: { $lt: 20 } }, { age: { $gt: 60 } }] }` |
| `$regex` | Expression régulière | `{ email: { $regex: "@gmail.com$" } }` |

### 5. Mise à jour

Modifier des documents existants.

```javascript
// Mettre à jour un document
db.users.updateOne(
  { nom: "Martin" },           // Critère de sélection
  { $set: { age: 26 } }        // Modification
)

// Mettre à jour plusieurs documents
db.users.updateMany(
  { age: { $lt: 30 } },
  { $set: { categorie: "junior" } }
)

// Remplacer un document entier
db.users.replaceOne(
  { nom: "Martin" },
  { nom: "Martin", prenom: "Paul", age: 26 }
)
```

### 6. Suppression

Supprimer des documents.

```javascript
// Supprimer un document
db.users.deleteOne({ nom: "Martin" })

// Supprimer plusieurs documents
db.users.deleteMany({ age: { $lt: 18 } })

// Supprimer tous les documents d'une collection
db.users.deleteMany({})
```

---

## ⚡ Avantages de MongoDB

### 1. Flexibilité du schéma

Pas besoin de définir la structure à l'avance. Idéal pour :
- Prototypage rapide
- Évolution des besoins métier
- Données hétérogènes

### 2. Performance

- **Index** puissants pour accélérer les recherches
- **Agrégation** pour traiter de gros volumes
- **Réplication** pour distribuer la charge en lecture

### 3. Scalabilité horizontale

- **Sharding** : Répartir les données sur plusieurs serveurs
- Gérer des pétaoctets de données
- Croissance linéaire des performances

### 4. Haute disponibilité

- **Replica Sets** : Redondance automatique
- **Failover** : Basculement automatique en cas de panne
- Pas d'interruption de service

### 5. Écosystème riche

- **MongoDB Compass** : Interface graphique officielle
- **MongoDB Atlas** : Solution cloud managée
- **Drivers** : Support de tous les langages (Node.js, Python, Java, Go, C#...)
- **Community** : Large communauté et documentation

---

## ⚠️ Limitations à connaître

### 1. Pas de JOIN natif

Les relations entre collections nécessitent :
- Documents imbriqués (duplication de données)
- Références manuelles (plusieurs requêtes)
- Agrégation `$lookup` (plus lent qu'un JOIN SQL)

### 2. Taille des documents

Limite de **16 Mo par document**. Pour de gros fichiers, utiliser **GridFS**.

### 3. Transactions

Les transactions multi-documents sont disponibles depuis MongoDB 4.0, mais :
- Plus lentes que les opérations simples
- Moins matures que SQL

### 4. Consommation mémoire

MongoDB charge les index en RAM :
- Nécessite beaucoup de mémoire pour de gros datasets
- Optimisation du schéma importante

---

## 🎯 Quand utiliser MongoDB ?

### ✅ Cas d'usage idéaux

| Scénario | Pourquoi MongoDB ? |
|----------|-------------------|
| **Application web/mobile** | Schéma flexible, données JSON naturelles |
| **Catalogue produits** | Documents hétérogènes, recherche plein texte |
| **Gestion de contenu** | Structure évolutive, pas de migrations |
| **IoT / Logs** | Ingestion massive, séries temporelles |
| **Données géospatiales** | Index géospatiaux natifs |
| **Prototypage** | Démarrage rapide sans DDL |
| **Microservices** | Base par service, pas de relations complexes |

### ❌ Cas où SQL est préférable

| Scénario | Pourquoi SQL ? |
|----------|---------------|
| **Transactions complexes** | ACID plus mature (banque, comptabilité) |
| **Relations multiples** | JOIN performants et simples |
| **Rapports complexes** | SQL analytique puissant |
| **Données structurées stables** | Schéma rigide = garanties fortes |
| **Équipe SQL experte** | Courbe d'apprentissage MongoDB |

---

## 🐳 MongoDB et Docker : La combinaison parfaite

### Pourquoi Docker pour MongoDB ?

1. **Installation instantanée** : Pas de configuration système
2. **Isolation** : Plusieurs versions de MongoDB en parallèle
3. **Reproductibilité** : Même environnement pour toute l'équipe
4. **Nettoyage facile** : Suppression propre sans traces
5. **Tests** : Environnements éphémères pour les tests

### Ce que vous allez apprendre

Dans les fiches suivantes, vous découvrirez comment :

1. **Déployer MongoDB** avec `docker-compose` (Fiche 5.1)
2. **Sécuriser** avec authentification (Fiche 5.2)
3. **Configurer une IP fixe** pour architectures multi-conteneurs (Fiche 5.3)
4. **Utiliser Mongo Express** : interface web d'administration (Fiche 5.4)
5. **Créer un Replica Set** : haute disponibilité et réplication (Fiche 5.5)

---

## 📚 Ressources complémentaires

### Documentation officielle

- [MongoDB Manual](https://www.mongodb.com/docs/manual/) - Documentation complète
- [MongoDB University](https://university.mongodb.com/) - Cours gratuits
- [MongoDB Compass](https://www.mongodb.com/products/compass) - Client graphique officiel
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) - Solution cloud

### Outils

| Outil | Type | Description |
|-------|------|-------------|
| **mongosh** | CLI | Shell MongoDB moderne (remplace `mongo`) |
| **MongoDB Compass** | GUI | Interface graphique officielle |
| **Mongo Express** | Web | Interface web légère (pour dev) |
| **Robo 3T** | GUI | Client léger open-source |
| **NoSQLBooster** | GUI | Client avancé avec intellisense |

### Communauté

- [MongoDB Community Forum](https://www.mongodb.com/community/forums/)
- [Stack Overflow - MongoDB](https://stackoverflow.com/questions/tagged/mongodb)
- [MongoDB Blog](https://www.mongodb.com/blog)
- [MongoDB GitHub](https://github.com/mongodb/mongo)

---

## 🎓 Prérequis pour suivre ce guide

### Connaissances recommandées

- ✅ Bases de la ligne de commande (terminal)
- ✅ Notions de bases de données (tables, requêtes)
- ✅ JSON (structure de base)
- ✅ Docker et Docker Compose installés

### Connaissances optionnelles (mais utiles)

- JavaScript (pour les requêtes MongoDB)
- Node.js (pour les exemples d'intégration)
- Concepts réseau (IP, ports)

---

## 🗺️ Plan du guide MongoDB

Voici les fiches que vous allez découvrir :

### 📘 Fiche 5.1 - Configuration basique
- Déployer MongoDB avec Docker Compose
- Premiers pas avec le shell MongoDB
- Créer des bases, collections et documents
- Persistance des données avec volumes

### 🔐 Fiche 5.2 - Authentification
- Activer la sécurité MongoDB
- Créer des utilisateurs avec différents rôles
- Gérer les permissions
- Bonnes pratiques de sécurité

### 🌐 Fiche 5.3 - IP fixe
- Créer un réseau Docker personnalisé
- Assigner une IP statique à MongoDB
- Connecter plusieurs conteneurs
- Architecture multi-services

### 🖥️ Fiche 5.4 - Mongo Express
- Déployer une interface web d'administration
- Gérer MongoDB depuis le navigateur
- Import/Export de données
- Configuration sécurisée

### 🔄 Fiche 5.5 - Replica Set
- Comprendre la réplication MongoDB
- Créer un cluster à haute disponibilité
- Tester le basculement automatique (failover)
- Monitoring et maintenance

---

## 💡 Conseils avant de commencer

### 1. Environnement de travail

Préparez votre espace de travail :

```bash
# Créer un dossier pour tous vos projets MongoDB
mkdir -p ~/docker-mongodb
cd ~/docker-mongodb
```

### 2. Vérifier Docker

```bash
# Vérifier que Docker fonctionne
docker --version
docker-compose --version

# Tester avec un conteneur simple
docker run hello-world
```

### 3. Nettoyer l'ancien

Si vous avez déjà MongoDB installé localement :

```bash
# Arrêter MongoDB local (si installé)
sudo systemctl stop mongod  # Linux
brew services stop mongodb  # macOS
```

### 4. Prévoir du temps

- **Fiche 5.1** : ~30 minutes (démarrage rapide)
- **Fiche 5.2** : ~45 minutes (authentification)
- **Fiche 5.3** : ~30 minutes (réseau)
- **Fiche 5.4** : ~30 minutes (interface web)
- **Fiche 5.5** : ~60 minutes (replica set)

**Total recommandé** : 3-4 heures pour maîtriser MongoDB sur Docker.

---

## 🎯 Objectifs d'apprentissage

À la fin de ce guide complet, vous serez capable de :

- ✅ **Déployer** MongoDB avec Docker en quelques minutes
- ✅ **Sécuriser** vos bases avec authentification
- ✅ **Gérer** des architectures multi-conteneurs
- ✅ **Utiliser** les outils d'administration graphiques
- ✅ **Créer** des clusters répliqués pour la haute disponibilité
- ✅ **Comprendre** les concepts avancés (réplication, sharding)
- ✅ **Intégrer** MongoDB dans vos applications
- ✅ **Dépanner** les problèmes courants

---

## 🚀 Prêt à commencer ?

Vous avez maintenant toutes les bases pour démarrer avec MongoDB sur Docker !

**Prochaine étape :**
👉 [5.1 - Configuration basique avec docker-compose](01-config-basique-docker-compose.md)

---

## 📞 Besoin d'aide ?

Si vous rencontrez des difficultés :

1. 📖 Consultez les [annexes](/annexes/) du guide
2. 💬 Posez vos questions sur le [MongoDB Community Forum](https://www.mongodb.com/community/forums/)
3. 🔍 Recherchez sur [Stack Overflow - MongoDB](https://stackoverflow.com/questions/tagged/mongodb)
4. 📚 Suivez les cours gratuits sur [MongoDB University](https://university.mongodb.com/)

---

**Bon apprentissage avec MongoDB et Docker ! 🎉**

🔝 Retour au [Sommaire](/SOMMAIRE.md)
