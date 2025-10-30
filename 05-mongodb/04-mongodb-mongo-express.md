# 5.4 - MongoDB avec Mongo Express (Interface Web)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Jusqu'à présent, nous avons interagi avec MongoDB via :
- Le shell en ligne de commande (`mongosh`)
- Des clients graphiques à installer (MongoDB Compass, Robo 3T...)

Ces outils sont excellents, mais ils nécessitent une installation et une configuration. **Mongo Express** est une alternative simple : une **interface web** qui s'exécute dans un conteneur Docker et vous permet de gérer MongoDB directement depuis votre navigateur !

### 🌐 Qu'est-ce que Mongo Express ?

**Mongo Express** est une interface d'administration web pour MongoDB, écrite en Node.js. C'est l'équivalent de **phpMyAdmin** pour MySQL ou **pgAdmin** pour PostgreSQL.

**Avantages :**
- ✅ Interface web intuitive (accessible depuis un navigateur)
- ✅ Pas d'installation supplémentaire sur votre machine
- ✅ Visualisation simple des bases, collections et documents
- ✅ Édition directe des documents (JSON)
- ✅ Exécution de requêtes
- ✅ Import/Export de données
- ✅ Parfait pour le développement et l'apprentissage

**Limitations :**
- ❌ **NE PAS utiliser en production** (sécurité limitée)
- ❌ Moins de fonctionnalités que MongoDB Compass
- ❌ Performances limitées pour de très grandes bases

---

## 🎯 Objectifs de cette fiche

À la fin de ce tutoriel, vous saurez :

- ✅ Déployer MongoDB et Mongo Express ensemble avec Docker Compose
- ✅ Configurer l'authentification entre les deux services
- ✅ Accéder à l'interface web de Mongo Express
- ✅ Gérer vos bases de données via le navigateur
- ✅ Comprendre la communication entre conteneurs

---

## 🛠️ Prérequis

Avant de commencer :

- **Docker** et **Docker Compose** installés
- Avoir suivi les fiches précédentes (recommandé mais pas obligatoire)
- Un navigateur web moderne (Chrome, Firefox, Edge...)

---

## 🏗️ Architecture de la solution

Voici ce que nous allons mettre en place :

```
┌─────────────────────────────────────────────────┐
│            Votre navigateur web                 │
│         http://localhost:8081                   │
└────────────────┬────────────────────────────────┘
                 │
                 │ Interface Web
                 ▼
┌─────────────────────────────────────────────────┐
│         Conteneur Mongo Express                 │
│              Port 8081                          │
└────────────────┬────────────────────────────────┘
                 │
                 │ Connexion MongoDB
                 │ (authentification)
                 ▼
┌─────────────────────────────────────────────────┐
│          Conteneur MongoDB                      │
│              Port 27017                         │
│           (avec authentification)               │
└─────────────────────────────────────────────────┘
```

---

## 📁 Étape 1 : Préparation de l'environnement

### Créer un nouveau dossier de projet

```bash
# Créer le dossier
mkdir mongodb_mongo_express

# Se déplacer dedans
cd mongodb_mongo_express

# Créer le dossier pour les données MongoDB
mkdir data
```

Votre structure :

```
mongodb_mongo_express/
└── data/          (stockage des données MongoDB)
```

---

## 📄 Étape 2 : Configuration avec `docker-compose.yml`

### Version simple (sans authentification MongoDB)

Créez un fichier **`docker-compose.yml`** :

```yaml
version: '3.8'

services:
  # Service MongoDB
  mongodb:
    image: mongo:7.0
    container_name: mongodb_dev
    restart: unless-stopped
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db

  # Service Mongo Express (Interface Web)
  mongo-express:
    image: mongo-express:latest
    container_name: mongo_express_web
    restart: unless-stopped
    ports:
      # Interface web accessible sur http://localhost:8081
      - "8081:8081"
    environment:
      # Adresse du serveur MongoDB
      # On utilise le nom du service 'mongodb' (résolution DNS Docker)
      ME_CONFIG_MONGODB_SERVER: mongodb

      # Port MongoDB (par défaut : 27017)
      ME_CONFIG_MONGODB_PORT: "27017"

      # ⚠️ OPTIONNEL : Protéger l'accès à Mongo Express
      # Identifiants pour accéder à l'interface web
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: motdepasse_web

    # Mongo Express démarre après MongoDB
    depends_on:
      - mongodb
```

### 📖 Explication des paramètres

**Service MongoDB :**

| Paramètre | Explication |
|-----------|-------------|
| `image: mongo:7.0` | Image officielle MongoDB |
| `container_name` | Nom du conteneur |
| `ports: "27017:27017"` | Exposition du port MongoDB |
| `volumes` | Persistance des données |

**Service Mongo Express :**

| Paramètre | Explication |
|-----------|-------------|
| `image: mongo-express:latest` | Image officielle Mongo Express |
| `ports: "8081:8081"` | Port pour accéder à l'interface web |
| `ME_CONFIG_MONGODB_SERVER` | Nom d'hôte de MongoDB (nom du service) |
| `ME_CONFIG_MONGODB_PORT` | Port MongoDB (27017 par défaut) |
| `ME_CONFIG_BASICAUTH_USERNAME` | Identifiant pour protéger l'accès web |
| `ME_CONFIG_BASICAUTH_PASSWORD` | Mot de passe pour protéger l'accès web |
| `depends_on` | Assure que MongoDB démarre avant Mongo Express |

---

## 🔐 Étape 3 : Configuration avec authentification MongoDB

### Version sécurisée (recommandée)

Si MongoDB utilise l'authentification (comme dans la [fiche 5.2](02-config-avec-authentification.md)), voici la configuration complète :

```yaml
version: '3.8'

services:
  # Service MongoDB avec authentification
  mongodb:
    image: mongo:7.0
    container_name: mongodb_secure
    restart: unless-stopped
    environment:
      # Création d'un utilisateur administrateur
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: mongodb_admin_pass_123
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db

  # Service Mongo Express
  mongo-express:
    image: mongo-express:latest
    container_name: mongo_express_web
    restart: unless-stopped
    ports:
      - "8081:8081"
    environment:
      # Connexion à MongoDB avec authentification
      ME_CONFIG_MONGODB_SERVER: mongodb
      ME_CONFIG_MONGODB_PORT: "27017"

      # ⭐ Identifiants MongoDB (doivent correspondre à ceux de MongoDB)
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: mongodb_admin_pass_123

      # Base de données d'authentification
      ME_CONFIG_MONGODB_AUTH_DATABASE: admin

      # Protection de l'interface web Mongo Express
      ME_CONFIG_BASICAUTH_USERNAME: webadmin
      ME_CONFIG_BASICAUTH_PASSWORD: webpass_securise_456

      # ⭐ OPTIONNEL : Personnalisation de l'interface
      ME_CONFIG_SITE_BASEURL: /
      ME_CONFIG_SITE_COOKIESECRET: secret_cookie_123
      ME_CONFIG_SITE_SESSIONSECRET: secret_session_456

    depends_on:
      - mongodb
```

### 🔑 Paramètres d'authentification supplémentaires

| Paramètre | Explication |
|-----------|-------------|
| `ME_CONFIG_MONGODB_ADMINUSERNAME` | Utilisateur MongoDB (doit exister) |
| `ME_CONFIG_MONGODB_ADMINPASSWORD` | Mot de passe de cet utilisateur |
| `ME_CONFIG_MONGODB_AUTH_DATABASE` | Base où sont stockées les infos d'auth (généralement `admin`) |
| `ME_CONFIG_BASICAUTH_USERNAME` | Identifiant pour l'interface web |
| `ME_CONFIG_BASICAUTH_PASSWORD` | Mot de passe pour l'interface web |

**⚠️ Important :** Les identifiants MongoDB (`ADMINUSERNAME` et `ADMINPASSWORD`) doivent correspondre **exactement** à ceux définis dans le service MongoDB.

---

## ▶️ Étape 4 : Démarrage des services

### Lancer MongoDB et Mongo Express

```bash
docker-compose up -d
```

**Sortie attendue :**

```
Creating network "mongodb_mongo_express_default" with the default driver
Creating mongodb_secure ... done
Creating mongo_express_web ... done
```

### ✅ Vérification du démarrage

```bash
docker-compose ps
```

**Résultat attendu :**

```
       Name                     Command              State           Ports
-----------------------------------------------------------------------------------
mongodb_secure        docker-entrypoint.sh mongod    Up      0.0.0.0:27017->27017/tcp
mongo_express_web     tini -- /docker-entrypoint...  Up      0.0.0.0:8081->8081/tcp
```

Les deux conteneurs doivent être en état **"Up"**.

### 🔍 Vérifier les logs

Si vous rencontrez des problèmes :

```bash
# Logs de MongoDB
docker-compose logs mongodb

# Logs de Mongo Express
docker-compose logs mongo-express

# Tous les logs en direct
docker-compose logs -f
```

---

## 🌐 Étape 5 : Accéder à Mongo Express

### A. Ouvrir l'interface web

Ouvrez votre navigateur et accédez à :

```
http://localhost:8081
```

### B. Authentification web

**Si vous avez configuré `ME_CONFIG_BASICAUTH` :**

Une fenêtre de connexion apparaît (pop-up du navigateur) :

| Champ | Valeur |
|-------|--------|
| **Nom d'utilisateur** | `webadmin` (ou ce que vous avez défini) |
| **Mot de passe** | `webpass_securise_456` (ou ce que vous avez défini) |

**Sans `ME_CONFIG_BASICAUTH` :**

L'interface s'ouvre directement (pas sécurisé, déconseillé si accessible depuis le réseau).

### C. Interface principale

Vous arrivez sur la page d'accueil de Mongo Express :

```
┌─────────────────────────────────────────────────┐
│  Mongo Express                                  │
│  Connected to: mongodb://mongodb:27017          │
├─────────────────────────────────────────────────┤
│  Databases:                                     │
│    ▶ admin                    (collections: 2) │
│    ▶ config                   (collections: 1) │
│    ▶ local                    (collections: 0) │
│                                                 │
│  [+ Create Database]                            │
└─────────────────────────────────────────────────┘
```

---

## 🗂️ Étape 6 : Utiliser Mongo Express

### A. Créer une base de données

1. Cliquez sur **"Create Database"** en bas de la page
2. Entrez un nom, par exemple : `mabase`
3. Cliquez sur **"Save"**

### B. Créer une collection

1. Cliquez sur le nom de votre base (`mabase`)
2. Dans le champ "Collection Name", entrez : `users`
3. Cliquez sur **"Create Collection"**

### C. Insérer un document

1. Ouvrez votre collection (`users`)
2. Cliquez sur **"New Document"**
3. Dans l'éditeur JSON, entrez :

```json
{
  "nom": "Dupont",
  "prenom": "Jean",
  "age": 30,
  "email": "jean.dupont@example.com"
}
```

4. Cliquez sur **"Save"**

### D. Visualiser les documents

Tous les documents de la collection apparaissent sous forme de cartes :

```
┌───────────────────────────────────────────┐
│  Document 1                               │
│  _id: ObjectId("...")                     │
│  nom: "Dupont"                            │
│  prenom: "Jean"                           │
│  age: 30                                  │
│  email: "jean.dupont@example.com"         │
│  [Edit] [Delete] [Clone]                  │
└───────────────────────────────────────────┘
```

### E. Modifier un document

1. Cliquez sur **"Edit"** sur le document
2. Modifiez les valeurs dans l'éditeur JSON
3. Cliquez sur **"Update"**

### F. Supprimer un document

1. Cliquez sur **"Delete"** sur le document
2. Confirmez la suppression

### G. Supprimer une collection

1. Ouvrez la collection
2. En haut de la page, cliquez sur **"Delete Collection"**
3. Confirmez

### H. Supprimer une base de données

1. Sur la page d'accueil, cliquez sur la base
2. Cliquez sur **"Delete Database"**
3. Confirmez

---

## 🔧 Étape 7 : Fonctionnalités avancées

### A. Import de données

Mongo Express permet d'importer des documents :

1. Ouvrez une collection
2. Cliquez sur **"Import"**
3. Collez votre JSON (tableau de documents) :

```json
[
  { "nom": "Martin", "prenom": "Sophie", "age": 25 },
  { "nom": "Bernard", "prenom": "Luc", "age": 35 },
  { "nom": "Dubois", "prenom": "Marie", "age": 28 }
]
```

4. Cliquez sur **"Import"**

### B. Export de données

1. Ouvrez une collection
2. Cliquez sur **"Export"**
3. Choisissez le format (JSON, CSV)
4. Téléchargez le fichier

### C. Requêtes personnalisées

En bas de la page d'une collection, vous pouvez exécuter des requêtes :

**Exemple 1 : Rechercher par critère**

```javascript
{ "age": { $gt: 25 } }
```

Affiche tous les documents où `age > 25`.

**Exemple 2 : Projection (sélectionner des champs)**

Dans "Projection", entrez :

```javascript
{ "nom": 1, "prenom": 1, "_id": 0 }
```

Affiche uniquement les champs `nom` et `prenom`.

### D. Index

Vous pouvez créer et gérer des index pour améliorer les performances :

1. Ouvrez une collection
2. Cliquez sur l'onglet **"Indexes"**
3. Cliquez sur **"New Index"**
4. Configurez votre index

---

## 🔒 Étape 8 : Sécurité et bonnes pratiques

### ✅ À FAIRE

1. **Toujours activer `ME_CONFIG_BASICAUTH`** pour protéger l'accès web
2. **Utiliser des mots de passe forts** pour MongoDB et l'interface web
3. **Ne pas exposer le port 8081** sur Internet (uniquement localhost)
4. **Limiter l'accès réseau** si possible
5. **Utiliser un fichier `.env`** pour les mots de passe (voir ci-dessous)

### ❌ À ÉVITER

1. ❌ **JAMAIS utiliser Mongo Express en production** (risques de sécurité)
2. ❌ Ne pas laisser l'interface accessible publiquement
3. ❌ Ne pas utiliser les mots de passe par défaut
4. ❌ Ne pas versionner les mots de passe dans Git

---

## 📁 Bonus : Utiliser un fichier `.env`

### A. Créer le fichier `.env`

Dans le dossier `mongodb_mongo_express`, créez un fichier **`.env`** :

```bash
# Identifiants MongoDB
MONGO_ROOT_USER=admin
MONGO_ROOT_PASS=mongodb_admin_pass_123

# Identifiants Mongo Express (interface web)
WEB_USER=webadmin
WEB_PASS=webpass_securise_456
```

### B. Modifier le `docker-compose.yml`

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb_secure
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASS}
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db

  mongo-express:
    image: mongo-express:latest
    container_name: mongo_express_web
    restart: unless-stopped
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongodb
      ME_CONFIG_MONGODB_PORT: "27017"
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_ROOT_USER}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_ROOT_PASS}
      ME_CONFIG_MONGODB_AUTH_DATABASE: admin
      ME_CONFIG_BASICAUTH_USERNAME: ${WEB_USER}
      ME_CONFIG_BASICAUTH_PASSWORD: ${WEB_PASS}
    depends_on:
      - mongodb
```

### C. Créer le `.gitignore`

**IMPORTANT :** Ne jamais versionner le fichier `.env` !

Créez un fichier **`.gitignore`** :

```
# Secrets
.env

# Données
data/
```

---

## 🌐 Étape 9 : Accès depuis le réseau local

Si vous souhaitez accéder à Mongo Express depuis un autre ordinateur de votre réseau :

### A. Trouver l'adresse IP de votre machine

**Windows :**
```bash
ipconfig
```

**Linux/macOS :**
```bash
ip addr
# ou
ifconfig
```

Notez l'adresse IP (exemple : `192.168.1.50`)

### B. Ouvrir le pare-feu

**Windows :**
- Ouvrez "Pare-feu Windows Defender"
- Créez une règle entrante pour le port **8081 TCP**

**Linux (ufw) :**
```bash
sudo ufw allow 8081/tcp
```

### C. Accéder depuis un autre ordinateur

Sur l'autre ordinateur, ouvrez le navigateur et allez à :

```
http://192.168.1.50:8081
```

(Remplacez `192.168.1.50` par votre IP)

---

## 🛑 Étape 10 : Gestion des services

### Commandes utiles

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Arrêter les services (données préservées)
docker-compose stop

# Redémarrer
docker-compose start

# Redémarrer complètement
docker-compose restart

# Voir l'état des conteneurs
docker-compose ps

# Redémarrer uniquement Mongo Express
docker-compose restart mongo-express
```

---

## 🧹 Étape 11 : Suppression complète

**⚠️ ATTENTION : Ceci supprime toutes les données !**

```bash
# 1. Arrêter et supprimer les conteneurs
docker-compose down

# 2. Supprimer les données
rm -rf data/

# 3. Supprimer les fichiers de configuration
rm docker-compose.yml .env

# 4. (Optionnel) Supprimer le dossier
cd ..
rm -rf mongodb_mongo_express
```

---

## ❓ Questions fréquentes (FAQ)

**Q : Mongo Express ne démarre pas, que faire ?**
R : Vérifiez que MongoDB est bien démarré en premier :
```bash
docker-compose logs mongodb
docker-compose logs mongo-express
```
Assurez-vous que les identifiants correspondent.

**Q : "Authentication failed" dans Mongo Express**
R : Vérifiez que :
- `ME_CONFIG_MONGODB_ADMINUSERNAME` correspond à un utilisateur MongoDB existant
- `ME_CONFIG_MONGODB_ADMINPASSWORD` est correct
- `ME_CONFIG_MONGODB_AUTH_DATABASE` est bien `admin`

**Q : Je n'arrive pas à accéder à http://localhost:8081**
R : Vérifiez que :
- Le conteneur Mongo Express est bien en état "Up" : `docker-compose ps`
- Le port 8081 n'est pas déjà utilisé par une autre application
- Essayez `http://127.0.0.1:8081`

**Q : Puis-je changer le port de Mongo Express ?**
R : Oui ! Modifiez dans le `docker-compose.yml` :
```yaml
ports:
  - "9090:8081"  # Accessible sur http://localhost:9090
```

**Q : Comment désactiver l'authentification web ?**
R : Supprimez ou commentez ces lignes dans `docker-compose.yml` :
```yaml
# ME_CONFIG_BASICAUTH_USERNAME: webadmin
# ME_CONFIG_BASICAUTH_PASSWORD: webpass_securise_456
```
⚠️ Déconseillé pour des raisons de sécurité !

**Q : Mongo Express vs MongoDB Compass, lequel choisir ?**
R :
- **Mongo Express** : Interface web simple, aucune installation, parfait pour démarrer
- **MongoDB Compass** : Client lourd plus complet, meilleures performances, plus de fonctionnalités

**Q : Puis-je utiliser Mongo Express pour plusieurs instances MongoDB ?**
R : Oui, mais il faut créer plusieurs services Mongo Express avec des ports différents, chacun connecté à une instance MongoDB différente.

---

## 📊 Comparaison des outils d'administration MongoDB

| Critère | Mongo Express | MongoDB Compass | mongosh (CLI) |
|---------|---------------|-----------------|---------------|
| **Type** | Interface web | Application desktop | Ligne de commande |
| **Installation** | Conteneur Docker | Téléchargement | Inclus avec MongoDB |
| **Courbe d'apprentissage** | ⭐⭐ (Facile) | ⭐⭐⭐ (Moyen) | ⭐⭐⭐⭐ (Avancé) |
| **Visualisation** | Bonne | Excellente | Limitée |
| **Performance** | Moyenne | Excellente | Très bonne |
| **Requêtes complexes** | Limitées | Complètes | Complètes |
| **Agrégations** | Non | Oui (pipeline builder) | Oui (code) |
| **Import/Export** | Basique | Avancé | Outils séparés |
| **Production** | ❌ Non | ⚠️ Prudence | ✅ Oui |
| **Idéal pour** | Débutants, tests rapides | Développement | Scripts, automatisation |

---

## 🎓 Pour aller plus loin

Cette configuration MongoDB + Mongo Express est idéale pour :
- ✅ Apprentissage de MongoDB
- ✅ Développement d'applications
- ✅ Prototypage rapide
- ✅ Démonstrations
- ✅ Tests d'API

**Prochaines étapes recommandées :**
- 👉 [5.5 Replica Set simple](05-replica-set-simple.md)
- 👉 [Cas pratique 02 - Stack MEAN](/cas-pratiques/02-stack-mean.md)
- 👉 [Annexe F - Outils de gestion](/annexes/F-outils-gestion.md)
- 👉 [MongoDB Compass](https://www.mongodb.com/products/compass) pour aller plus loin

---

## 📝 Résumé de ce que vous avez appris

| Concept | Ce que vous savez maintenant |
|---------|------------------------------|
| **Mongo Express** | Déployer une interface web pour MongoDB |
| **Multi-conteneurs** | Faire communiquer MongoDB et Mongo Express |
| **Authentification** | Configurer la connexion sécurisée entre services |
| **Interface web** | Gérer bases, collections et documents via navigateur |
| **Import/Export** | Transférer des données facilement |
| **Sécurité** | Protéger l'accès avec BasicAuth |

---

## 🎯 Points clés à retenir

- 🌐 **Mongo Express = interface web** pour gérer MongoDB facilement
- 🔗 **depends_on** assure l'ordre de démarrage des conteneurs
- 🔑 **Deux niveaux d'authentification** : MongoDB + interface web
- 🚫 **NE JAMAIS utiliser en production** (sécurité insuffisante)
- 📁 **Fichiers .env** pour sécuriser les mots de passe
- 💡 **Parfait pour l'apprentissage** et le développement rapide

---

## 🆘 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. **Vérifiez les logs** : `docker-compose logs -f`
2. **Testez MongoDB seul** : `docker exec -it mongodb_secure mongosh`
3. **Vérifiez les identifiants** : ils doivent correspondre exactement
4. **Consultez l'annexe** : [E - Dépannage](/annexes/E-depannage.md)
5. **Documentation Mongo Express** : [GitHub](https://github.com/mongo-express/mongo-express)
6. **Redémarrage propre** :
   ```bash
   docker-compose down
   docker-compose up -d
   ```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
