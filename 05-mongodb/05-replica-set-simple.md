# 5.5 - Replica Set MongoDB simple

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Jusqu'à présent, nous avons travaillé avec une **instance unique** de MongoDB. C'est parfait pour le développement, mais cela présente un problème majeur : **si MongoDB tombe en panne, vos données deviennent inaccessibles**.

Dans cette fiche, nous allons mettre en place un **Replica Set** (ensemble de réplication), une fonctionnalité avancée de MongoDB qui permet :
- 🔄 **Réplication automatique** des données sur plusieurs serveurs
- 🛡️ **Haute disponibilité** : si un serveur tombe, les autres prennent le relais
- 📊 **Répartition de la charge** en lecture

### 🤔 Qu'est-ce qu'un Replica Set ?

Un **Replica Set** est un **groupe de serveurs MongoDB** qui maintiennent la même copie des données. Il se compose de :

1. **Un nœud PRIMARY (principal)** : Reçoit toutes les écritures
2. **Un ou plusieurs nœuds SECONDARY (secondaires)** : Répliquent les données du PRIMARY
3. **Optionnel : Un nœud ARBITER** : Ne stocke pas de données, mais participe aux élections

**Analogie simple :**
Imaginez un professeur (PRIMARY) qui écrit au tableau. Les élèves (SECONDARY) recopient ce qu'il écrit dans leurs cahiers. Si le professeur part, un élève peut devenir le nouveau professeur et continuer le cours.

### 📊 Architecture d'un Replica Set à 3 nœuds

```
┌─────────────────────────────────────────────────────────┐
│                    Votre Application                    │
└────────────┬────────────────────────────────────────────┘
             │
             │ Écritures et Lectures
             ▼
┌─────────────────────────────────────────────────────────┐
│                   PRIMARY (Port 27017)                  │
│              ✍️  Reçoit toutes les écritures            │
│              📖 Lecture possible                        │
└────────────┬─────────────────┬──────────────────────────┘
             │                 │
             │ Réplication     │ Réplication
             ▼                 ▼
┌──────────────────────┐  ┌──────────────────────┐
│  SECONDARY #1        │  │  SECONDARY #2        │
│  (Port 27018)        │  │  (Port 27019)        │
│  📖 Lecture seule*   │  │  📖 Lecture seule*   │
│  🔄 Copie du PRIMARY │  │  🔄 Copie du PRIMARY │
└──────────────────────┘  └──────────────────────┘

* Par défaut, mais peut être configuré
```

**Fonctionnement :**
- Toutes les **écritures** vont sur le **PRIMARY**
- Le PRIMARY **réplique** automatiquement vers les SECONDARY
- Si le PRIMARY tombe, un SECONDARY devient automatiquement le nouveau PRIMARY

---

## 🎯 Objectifs de cette fiche

À la fin de ce tutoriel, vous saurez :

- ✅ Comprendre ce qu'est un Replica Set
- ✅ Déployer un Replica Set à 3 nœuds avec Docker
- ✅ Initialiser et configurer le Replica Set
- ✅ Tester le basculement automatique (failover)
- ✅ Vérifier l'état et la santé du cluster
- ✅ Comprendre les concepts de PRIMARY et SECONDARY

---

## 🛠️ Prérequis

Avant de commencer :

- **Docker** et **Docker Compose** installés
- Avoir suivi les fiches précédentes (fortement recommandé)
- Comprendre les bases de MongoDB
- 8 Go de RAM minimum recommandés (3 instances MongoDB)

---

## ⚠️ Important : Contexte d'apprentissage

**Ce tutoriel est destiné à l'apprentissage et aux tests en local.**

Pour la production :
- ❌ Ne pas utiliser Docker Compose pour un vrai cluster
- ❌ Un Replica Set nécessite au minimum 3 serveurs **physiques** séparés
- ✅ Utiliser Kubernetes, Docker Swarm ou des machines virtuelles dédiées
- ✅ Consulter la documentation MongoDB officielle pour la production

---

## 📁 Étape 1 : Préparation de l'environnement

### Créer la structure du projet

```bash
# Créer le dossier principal
mkdir mongodb_replica_set

# Se déplacer dedans
cd mongodb_replica_set

# Créer les dossiers pour les données de chaque nœud
mkdir -p data/mongo1 data/mongo2 data/mongo3
```

Votre structure :

```
mongodb_replica_set/
├── data/
│   ├── mongo1/    (données du nœud PRIMARY initial)
│   ├── mongo2/    (données du SECONDARY #1)
│   └── mongo3/    (données du SECONDARY #2)
└── docker-compose.yml (à créer)
```

---

## 📄 Étape 2 : Configuration du `docker-compose.yml`

### Version sans authentification (apprentissage)

Créez un fichier **`docker-compose.yml`** :

```yaml
version: '3.8'

services:
  # ===== Nœud MongoDB 1 (PRIMARY initial) =====
  mongo1:
    image: mongo:7.0
    container_name: mongo1
    restart: unless-stopped
    ports:
      - "27017:27017"
    volumes:
      - ./data/mongo1:/data/db
    command: mongod --replSet rs0 --bind_ip_all
    networks:
      - mongo-cluster

  # ===== Nœud MongoDB 2 (SECONDARY) =====
  mongo2:
    image: mongo:7.0
    container_name: mongo2
    restart: unless-stopped
    ports:
      - "27018:27017"
    volumes:
      - ./data/mongo2:/data/db
    command: mongod --replSet rs0 --bind_ip_all
    networks:
      - mongo-cluster

  # ===== Nœud MongoDB 3 (SECONDARY) =====
  mongo3:
    image: mongo:7.0
    container_name: mongo3
    restart: unless-stopped
    ports:
      - "27019:27017"
    volumes:
      - ./data/mongo3:/data/db
    command: mongod --replSet rs0 --bind_ip_all
    networks:
      - mongo-cluster

# Réseau partagé pour la communication entre nœuds
networks:
  mongo-cluster:
    driver: bridge
```

### 📖 Explication des paramètres

**Paramètres communs à tous les nœuds :**

| Paramètre | Explication |
|-----------|-------------|
| `image: mongo:7.0` | Même version pour tous les nœuds (important !) |
| `command: mongod --replSet rs0` | Lance MongoDB en mode Replica Set nommé "rs0" |
| `--bind_ip_all` | Permet les connexions depuis tous les réseaux |
| `networks: mongo-cluster` | Tous les nœuds sur le même réseau Docker |

**Particularités par nœud :**

| Nœud | Port hôte | Port conteneur | Volume |
|------|-----------|----------------|--------|
| mongo1 | 27017 | 27017 | ./data/mongo1 |
| mongo2 | 27018 | 27017 | ./data/mongo2 |
| mongo3 | 27019 | 27017 | ./data/mongo3 |

**💡 Important :** Chaque nœud écoute sur le port 27017 **à l'intérieur** du conteneur, mais est exposé sur des ports différents **sur votre machine** (27017, 27018, 27019).

---

## ▶️ Étape 3 : Démarrage des nœuds MongoDB

### Lancer les trois nœuds

```bash
docker-compose up -d
```

**Sortie attendue :**

```
Creating network "mongodb_replica_set_mongo-cluster" with driver "bridge"
Creating mongo1 ... done
Creating mongo2 ... done
Creating mongo3 ... done
```

### ✅ Vérification du démarrage

```bash
docker-compose ps
```

**Résultat attendu :**

```
  Name                Command             State            Ports
---------------------------------------------------------------------------
mongo1    mongod --replSet rs0 ...    Up      0.0.0.0:27017->27017/tcp
mongo2    mongod --replSet rs0 ...    Up      0.0.0.0:27018->27017/tcp
mongo3    mongod --replSet rs0 ...    Up      0.0.0.0:27019->27017/tcp
```

Les trois conteneurs doivent être en état **"Up"**.

---

## 🔧 Étape 4 : Initialisation du Replica Set

Les nœuds sont démarrés, mais ils **ne se connaissent pas encore**. Nous devons les configurer pour former un Replica Set.

### A. Se connecter au premier nœud

```bash
docker exec -it mongo1 mongosh
```

Vous êtes maintenant dans le shell MongoDB du nœud `mongo1`.

### B. Initialiser le Replica Set

Dans le shell MongoDB, exécutez :

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
```

**Explication :**

| Élément | Signification |
|---------|---------------|
| `_id: "rs0"` | Nom du Replica Set (doit correspondre au paramètre `--replSet`) |
| `members` | Liste des nœuds membres |
| `{ _id: 0, host: "mongo1:27017" }` | Premier nœud (devient PRIMARY) |
| `host: "mongo1:27017"` | Nom du conteneur (résolution DNS Docker) |

**Résultat attendu :**

```javascript
{
  ok: 1,
  '$clusterTime': { ... },
  operationTime: Timestamp({ ... })
}
```

Si vous voyez `ok: 1`, l'initialisation a réussi ! 🎉

### C. Observer le changement de prompt

Après quelques secondes, votre prompt MongoDB change :

```
test>                     (avant)
rs0 [direct: primary]>    (après)
```

Le `[direct: primary]` confirme que ce nœud est devenu le PRIMARY.

---

## 🔍 Étape 5 : Vérification du Replica Set

### A. Vérifier l'état du Replica Set

Dans le shell MongoDB (toujours connecté à mongo1) :

```javascript
rs.status()
```

Cette commande affiche des informations détaillées. Cherchez :

**Pour chaque membre :**

```javascript
{
  _id: 0,
  name: "mongo1:27017",
  stateStr: "PRIMARY",    // ← Rôle du nœud
  health: 1,              // ← 1 = OK, 0 = problème
  // ...
}
```

**Résumé attendu :**

| Nœud | État attendu | Santé |
|------|--------------|-------|
| mongo1:27017 | **PRIMARY** | 1 (OK) |
| mongo2:27017 | **SECONDARY** | 1 (OK) |
| mongo3:27017 | **SECONDARY** | 1 (OK) |

### B. Vérifier la configuration

```javascript
rs.conf()
```

Affiche la configuration complète du Replica Set.

### C. Version simplifiée de l'état

```javascript
rs.isMaster()
```

Affiche des informations essentielles :

```javascript
{
  ismaster: true,           // Ce nœud est le PRIMARY
  hosts: [
    "mongo1:27017",
    "mongo2:27017",
    "mongo3:27017"
  ],
  primary: "mongo1:27017",  // Adresse du PRIMARY actuel
  me: "mongo1:27017",       // Adresse de ce nœud
  // ...
}
```

---

## 📝 Étape 6 : Tester la réplication

### A. Insérer des données sur le PRIMARY

Depuis le shell MongoDB connecté à `mongo1` (PRIMARY) :

```javascript
// Utiliser une base de données
use testdb

// Insérer des documents
db.users.insertOne({ nom: "Dupont", prenom: "Jean", role: "admin" })
db.users.insertOne({ nom: "Martin", prenom: "Sophie", role: "user" })
db.users.insertOne({ nom: "Bernard", prenom: "Luc", role: "user" })

// Vérifier l'insertion
db.users.find()
```

### B. Vérifier la réplication sur un SECONDARY

**Ouvrez un nouveau terminal** et connectez-vous à `mongo2` :

```bash
docker exec -it mongo2 mongosh
```

Le prompt devrait indiquer `[direct: secondary]`.

**Par défaut, vous ne pouvez pas lire sur un SECONDARY.** Activez la lecture :

```javascript
// Autoriser la lecture sur ce nœud SECONDARY
rs.secondaryOk()
// ou la nouvelle syntaxe :
db.getMongo().setReadPref("secondaryPreferred")

// Utiliser la même base de données
use testdb

// Lire les données répliquées
db.users.find()
```

**Résultat :** Vous devriez voir les 3 documents insérés sur le PRIMARY ! ✅

**Explication :**
Les données ont été automatiquement **répliquées** de `mongo1` (PRIMARY) vers `mongo2` (SECONDARY).

### C. Vérifier sur le troisième nœud

Répétez la même opération sur `mongo3` :

```bash
docker exec -it mongo3 mongosh
```

```javascript
rs.secondaryOk()
use testdb
db.users.find()
```

Les données sont également présentes ! Les 3 nœuds ont bien la même copie des données.

---

## 🔄 Étape 7 : Tester le basculement automatique (Failover)

Le grand intérêt d'un Replica Set est la **haute disponibilité**. Si le PRIMARY tombe, un SECONDARY devient automatiquement le nouveau PRIMARY.

### A. Identifier le PRIMARY actuel

```javascript
// Depuis n'importe quel nœud
rs.isMaster().primary
```

Résultat (exemple) : `"mongo1:27017"`

### B. Simuler une panne du PRIMARY

**Dans un nouveau terminal**, arrêtez `mongo1` :

```bash
docker stop mongo1
```

### C. Observer l'élection

Connectez-vous à `mongo2` ou `mongo3` :

```bash
docker exec -it mongo2 mongosh
```

Attendez quelques secondes (généralement 10-30 secondes), puis vérifiez :

```javascript
rs.isMaster()
```

**Résultat attendu :**

```javascript
{
  ismaster: true,           // mongo2 est devenu PRIMARY !
  primary: "mongo2:27017",  // Nouveau PRIMARY
  // ...
}
```

🎉 **Un nouveau PRIMARY a été élu automatiquement !**

### D. Vérifier que les écritures fonctionnent

Depuis le nouveau PRIMARY (`mongo2`), insérez de nouvelles données :

```javascript
use testdb
db.users.insertOne({ nom: "Nouveau", prenom: "User", role: "test" })
db.users.find()
```

Les écritures fonctionnent ! Le cluster est toujours opérationnel.

### E. Vérifier la réplication sur mongo3

```bash
docker exec -it mongo3 mongosh
```

```javascript
rs.secondaryOk()
use testdb
db.users.find()
```

Le nouveau document est bien répliqué sur `mongo3`.

### F. Redémarrer mongo1

```bash
docker start mongo1
```

Attendez 10-20 secondes, puis vérifiez son état :

```bash
docker exec -it mongo1 mongosh
```

```javascript
rs.isMaster()
```

**Résultat :**

```javascript
{
  ismaster: false,          // mongo1 n'est plus PRIMARY
  primary: "mongo2:27017",  // mongo2 reste PRIMARY
  // ...
}
```

`mongo1` a rejoint le cluster en tant que **SECONDARY** ! Il récupère automatiquement les données manquées pendant son arrêt.

---

## 📊 Étape 8 : Comprendre les états des nœuds

### États possibles d'un nœud

| État | Description | Peut lire ? | Peut écrire ? |
|------|-------------|-------------|---------------|
| **PRIMARY** | Nœud principal | ✅ Oui | ✅ Oui |
| **SECONDARY** | Nœud secondaire | ⚠️ Avec `rs.secondaryOk()` | ❌ Non |
| **RECOVERING** | En cours de synchronisation | ❌ Non | ❌ Non |
| **STARTUP** | Démarrage initial | ❌ Non | ❌ Non |
| **STARTUP2** | Synchronisation initiale | ❌ Non | ❌ Non |
| **ARBITER** | Arbitre (élections seulement) | ❌ Non | ❌ Non |
| **DOWN** | Hors ligne | ❌ Non | ❌ Non |
| **UNKNOWN** | État inconnu | ❌ Non | ❌ Non |

### Commandes de monitoring

```javascript
// État complet du Replica Set
rs.status()

// Résumé court
rs.printReplicationInfo()

// Configuration actuelle
rs.conf()

// Vérifier qui est PRIMARY
rs.isMaster().primary
```

---

## 🔐 Étape 9 : Version avec authentification (optionnelle)

Pour sécuriser votre Replica Set, ajoutez l'authentification.

### A. Créer un fichier keyfile

Le keyfile permet aux nœuds de s'authentifier entre eux.

```bash
# Créer un keyfile sécurisé
openssl rand -base64 756 > keyfile

# Définir les bonnes permissions (important !)
chmod 400 keyfile
```

### B. Modifier le docker-compose.yml

```yaml
version: '3.8'

services:
  mongo1:
    image: mongo:7.0
    container_name: mongo1
    restart: unless-stopped
    ports:
      - "27017:27017"
    volumes:
      - ./data/mongo1:/data/db
      - ./keyfile:/data/keyfile    # ⭐ Nouveau
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin_password_123
    command: mongod --replSet rs0 --bind_ip_all --keyFile /data/keyfile
    networks:
      - mongo-cluster

  mongo2:
    image: mongo:7.0
    container_name: mongo2
    restart: unless-stopped
    ports:
      - "27018:27017"
    volumes:
      - ./data/mongo2:/data/db
      - ./keyfile:/data/keyfile    # ⭐ Nouveau
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin_password_123
    command: mongod --replSet rs0 --bind_ip_all --keyFile /data/keyfile
    networks:
      - mongo-cluster

  mongo3:
    image: mongo:7.0
    container_name: mongo3
    restart: unless-stopped
    ports:
      - "27019:27017"
    volumes:
      - ./data/mongo3:/data/db
      - ./keyfile:/data/keyfile    # ⭐ Nouveau
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin_password_123
    command: mongod --replSet rs0 --bind_ip_all --keyFile /data/keyfile
    networks:
      - mongo-cluster

networks:
  mongo-cluster:
    driver: bridge
```

### C. Se connecter avec authentification

```bash
docker exec -it mongo1 mongosh -u admin -p admin_password_123 --authenticationDatabase admin
```

---

## 🌐 Étape 10 : Se connecter depuis une application

### URI de connexion au Replica Set

Pour connecter votre application au Replica Set, utilisez cette URI :

```
mongodb://mongo1:27017,mongo2:27017,mongo3:27017/?replicaSet=rs0
```

**Avec authentification :**

```
mongodb://admin:admin_password_123@localhost:27017,localhost:27018,localhost:27019/?replicaSet=rs0&authSource=admin
```

### Exemple Node.js

```javascript
const { MongoClient } = require('mongodb');

const uri = "mongodb://localhost:27017,localhost:27018,localhost:27019/?replicaSet=rs0";
const client = new MongoClient(uri);

async function connecter() {
  try {
    await client.connect();
    console.log("Connecté au Replica Set !");

    const db = client.db("testdb");
    const collection = db.collection("users");

    // L'application se connecte automatiquement au PRIMARY
    await collection.insertOne({ nom: "Test", prenom: "App" });

  } finally {
    await client.close();
  }
}

connecter();
```

**Avantage :** Si le PRIMARY tombe, le driver MongoDB détecte automatiquement le nouveau PRIMARY et reconnecte l'application !

---

## 🛑 Étape 11 : Gestion du Replica Set

### Commandes utiles

```bash
# Voir les logs d'un nœud
docker logs mongo1

# Redémarrer un nœud
docker restart mongo2

# Arrêter tous les nœuds
docker-compose stop

# Redémarrer tous les nœuds
docker-compose start

# Voir l'état des conteneurs
docker-compose ps
```

### Commandes MongoDB (dans le shell)

```javascript
// Forcer une élection (utile pour les tests)
rs.stepDown()

// Voir les statistiques de réplication
rs.printReplicationInfo()

// Ajouter un nœud (avancé)
rs.add("mongo4:27017")

// Retirer un nœud (avancé)
rs.remove("mongo3:27017")

// Reconfigurer le Replica Set
rs.reconfig(nouvelleConfig)
```

---

## 🧹 Étape 12 : Suppression complète

**⚠️ ATTENTION : Ceci supprime toutes les données !**

```bash
# 1. Arrêter et supprimer les conteneurs
docker-compose down

# 2. Supprimer les données
rm -rf data/

# 3. Supprimer le keyfile (si créé)
rm keyfile

# 4. Supprimer les fichiers de configuration
rm docker-compose.yml
```

---

## ❓ Questions fréquentes (FAQ)

**Q : Combien de nœuds minimum pour un Replica Set ?**
R : **3 nœuds minimum** pour la production. 1 PRIMARY + 2 SECONDARY (ou 2 SECONDARY + 1 ARBITER).

**Q : Pourquoi 3 et pas 2 ?**
R : Pour l'élection d'un nouveau PRIMARY, il faut une **majorité** (plus de 50%). Avec 2 nœuds, si 1 tombe, il n'y a pas de majorité.

**Q : Qu'est-ce qu'un ARBITER ?**
R : Un nœud qui ne stocke pas de données mais participe aux élections. Utile pour avoir un nombre impair de nœuds sans stocker 3 copies complètes.

**Q : Puis-je avoir plus de 3 nœuds ?**
R : Oui ! Vous pouvez avoir jusqu'à 50 membres (mais max 7 votants). Plus de nœuds = plus de résilience mais aussi plus de ressources.

**Q : Les SECONDARY répliquent-ils en temps réel ?**
R : Presque ! La réplication est **asynchrone** mais très rapide (généralement < 1 seconde de décalage).

**Q : Puis-je lire depuis un SECONDARY par défaut ?**
R : Non, pour garantir la cohérence. Vous devez explicitement autoriser la lecture avec `rs.secondaryOk()` ou configurer le Read Preference de votre application.

**Q : Comment MongoDB choisit le nouveau PRIMARY ?**
R : Par **élection** : les nœuds votent pour le membre avec les données les plus à jour et la priorité la plus élevée.

**Q : Que se passe-t-il si tous les nœuds tombent ?**
R : Le cluster devient indisponible jusqu'à ce qu'au moins 2 nœuds (majorité) soient redémarrés.

**Q : Cette configuration est-elle adaptée à la production ?**
R : ❌ Non, pas sur Docker Compose. Pour la production :
- Utilisez des serveurs physiques/VMs séparés
- Configurez la réplication sur WAN (réseau étendu)
- Ajoutez du monitoring (Ops Manager, Cloud Manager)
- Testez les plans de reprise après sinistre

---

## 🎓 Pour aller plus loin

Cette configuration Replica Set est idéale pour :
- ✅ Comprendre la haute disponibilité
- ✅ Tester le failover en développement
- ✅ Apprendre les concepts de réplication
- ✅ Préparer une architecture de production

**Prochaines étapes recommandées :**
- 👉 [Annexe E - Dépannage](/annexes/E-depannage.md)
- 👉 [Cas pratique 02 - Stack MEAN](/cas-pratiques/02-stack-mean.md)
- 👉 Documentation MongoDB : [Replication](https://www.mongodb.com/docs/manual/replication/)
- 👉 MongoDB University : Cours gratuits sur la réplication

---

## 📝 Résumé de ce que vous avez appris

| Concept | Ce que vous savez maintenant |
|---------|------------------------------|
| **Replica Set** | Groupe de nœuds MongoDB répliquant les données |
| **PRIMARY** | Nœud qui reçoit toutes les écritures |
| **SECONDARY** | Nœuds qui répliquent le PRIMARY |
| **Failover** | Basculement automatique si PRIMARY tombe |
| **Élection** | Processus de choix du nouveau PRIMARY |
| **Réplication** | Copie automatique des données entre nœuds |
| **Haute disponibilité** | Service continue même si un nœud tombe |

---

## 🎯 Points clés à retenir

- 🔄 **Replica Set = réplication automatique** des données
- 🛡️ **Haute disponibilité** : résistance aux pannes
- 👑 **Un seul PRIMARY** à la fois
- 🗳️ **Élections automatiques** en cas de panne
- 📊 **3 nœuds minimum** pour la production
- ⚠️ **Docker Compose = apprentissage uniquement**
- 🔐 **Keyfile obligatoire** pour la production
- 🚀 **Les applications reconnectent automatiquement** au nouveau PRIMARY

---

## 🆘 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. **Vérifiez que les 3 nœuds sont UP** : `docker-compose ps`
2. **Consultez les logs** : `docker logs mongo1`
3. **Vérifiez l'état du cluster** : `rs.status()`
4. **Problème de connexion** : Vérifiez le réseau Docker
5. **Problème d'élection** : Assurez-vous d'avoir 3 nœuds actifs
6. **Documentation** : [MongoDB Replication](https://www.mongodb.com/docs/manual/replication/)
7. **Redémarrage propre** :
   ```bash
   docker-compose down
   rm -rf data/
   docker-compose up -d
   # Puis réinitialiser le Replica Set (Étape 4)
   ```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
