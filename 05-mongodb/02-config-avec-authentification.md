# 5.2 - Configuration de MongoDB avec authentification

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans la [fiche précédente (5.1)](01-config-basique-docker-compose.md), nous avons mis en place MongoDB sans authentification. C'était parfait pour débuter, mais **très peu sécurisé** : n'importe qui peut accéder à vos données !

Dans cette fiche, nous allons **activer l'authentification** pour protéger votre base de données. Cela signifie que vous devrez fournir un nom d'utilisateur et un mot de passe pour vous connecter.

### 🔐 Pourquoi activer l'authentification ?

**Sans authentification :**
- ❌ Toute personne ayant accès au port peut lire/modifier/supprimer vos données
- ❌ Impossible d'utiliser sur un réseau partagé
- ❌ Non conforme aux bonnes pratiques de sécurité

**Avec authentification :**
- ✅ Seuls les utilisateurs autorisés peuvent accéder aux données
- ✅ Vous pouvez créer plusieurs utilisateurs avec différents niveaux de permissions
- ✅ Approche plus proche d'un environnement de production

---

## 🎯 Objectifs de cette fiche

À la fin de ce tutoriel, vous saurez :

- ✅ Activer l'authentification sur MongoDB
- ✅ Créer un utilisateur administrateur
- ✅ Créer des utilisateurs avec des permissions spécifiques
- ✅ Vous connecter à MongoDB avec authentification (shell et client GUI)
- ✅ Gérer les utilisateurs et leurs droits

---

## 🛠️ Prérequis

Avant de commencer :

- **Docker** et **Docker Compose** installés
- Avoir suivi la [fiche 5.1](01-config-basique-docker-compose.md) ou comprendre les bases de Docker Compose
- Un éditeur de texte

---

## 📁 Étape 1 : Préparation de l'environnement

### A. Créer un nouveau dossier de projet

```bash
# Créer le dossier
mkdir mongodb_avec_auth

# Se déplacer dedans
cd mongodb_avec_auth

# Créer le dossier pour les données
mkdir data
```

Votre structure :

```
mongodb_avec_auth/
└── data/          (vide pour l'instant)
```

---

## 📄 Étape 2 : Création du fichier `docker-compose.yml`

Créez un fichier **`docker-compose.yml`** avec le contenu suivant :

```yaml
version: '3.8'

services:
  mongodb:
    # Image officielle MongoDB (version 7.0)
    image: mongo:7.0

    # Nom du conteneur
    container_name: mongodb_secure

    # Redémarre automatiquement
    restart: unless-stopped

    # Variables d'environnement pour créer l'utilisateur root
    environment:
      # Nom d'utilisateur de l'administrateur principal
      MONGO_INITDB_ROOT_USERNAME: admin

      # ⚠️ CHANGEZ CE MOT DE PASSE ⚠️
      # Utilisez un mot de passe fort (lettres, chiffres, symboles)
      MONGO_INITDB_ROOT_PASSWORD: motdepasse_securise_123

      # Base de données par défaut (optionnel)
      MONGO_INITDB_DATABASE: admin

    # Port pour accéder à MongoDB
    ports:
      - "27017:27017"

    # Volume pour la persistance des données
    volumes:
      - ./data:/data/db
```

### 📖 Explication des nouveautés

| Élément | Signification |
|---------|---------------|
| `environment:` | Variables d'environnement pour configurer MongoDB |
| `MONGO_INITDB_ROOT_USERNAME` | Nom de l'utilisateur administrateur créé au premier démarrage |
| `MONGO_INITDB_ROOT_PASSWORD` | Mot de passe de cet administrateur |
| `MONGO_INITDB_DATABASE` | Base de données initiale (généralement `admin`) |

### 🔒 Important : Sécurité des mots de passe

**⚠️ NE JAMAIS utiliser des mots de passe simples comme :**
- `password`
- `123456`
- `admin`
- `mongodb`

**✅ Utilisez des mots de passe forts :**
- Au moins 12 caractères
- Mélange de lettres (majuscules et minuscules)
- Chiffres
- Symboles (`!@#$%^&*`)

**Exemple de bon mot de passe :** `Mdb$2024!SecurePass#`

---

## ▶️ Étape 3 : Démarrage de MongoDB avec authentification

Lancez MongoDB :

```bash
docker-compose up -d
```

**Sortie attendue :**

```
Creating network "mongodb_avec_auth_default" with the default driver
Creating mongodb_secure ... done
```

### ✅ Vérification du démarrage

```bash
docker-compose ps
```

Résultat attendu :

```
     Name                   Command             State            Ports
--------------------------------------------------------------------------------
mongodb_secure   docker-entrypoint.sh mongod   Up      0.0.0.0:27017->27017/tcp
```

### 🔍 Vérification dans les logs

Pour confirmer que l'authentification est activée :

```bash
docker-compose logs mongodb | grep -i auth
```

Vous devriez voir des messages mentionnant l'authentification.

---

## 🔑 Étape 4 : Connexion avec authentification

### A. Depuis le shell MongoDB

Maintenant que l'authentification est activée, vous **devez** fournir les identifiants pour vous connecter.

**Méthode 1 : Connexion directe avec identifiants**

```bash
docker exec -it mongodb_secure mongosh -u admin -p motdepasse_securise_123 --authenticationDatabase admin
```

**Explication des paramètres :**
- `-u admin` : Nom d'utilisateur
- `-p motdepasse_securise_123` : Mot de passe
- `--authenticationDatabase admin` : Base de données contenant les informations d'authentification

**Méthode 2 : Connexion puis authentification**

```bash
# Se connecter sans authentification
docker exec -it mongodb_secure mongosh

# Une fois dans le shell, s'authentifier
use admin
db.auth("admin", "motdepasse_securise_123")
```

**Résultat attendu :**

```javascript
{ ok: 1 }
```

Si vous voyez `{ ok: 1 }`, vous êtes connecté avec succès ! 🎉

### B. Depuis MongoDB Compass (client graphique)

**Paramètres de connexion :**

| Paramètre | Valeur |
|-----------|--------|
| **Type de connexion** | URI |
| **URI complète** | `mongodb://admin:motdepasse_securise_123@localhost:27017/?authSource=admin` |

**Ou en mode formulaire :**

| Champ | Valeur |
|-------|--------|
| **Hôte** | `localhost` |
| **Port** | `27017` |
| **Username** | `admin` |
| **Password** | `motdepasse_securise_123` |
| **Authentication Database** | `admin` |
| **Authentication Method** | Username/Password |

---

## 👥 Étape 5 : Créer des utilisateurs supplémentaires

L'utilisateur `admin` a tous les droits. Pour plus de sécurité, créez des utilisateurs avec des permissions limitées pour vos applications.

### A. Créer un utilisateur pour une base de données spécifique

Connectez-vous avec l'utilisateur `admin` :

```bash
docker exec -it mongodb_secure mongosh -u admin -p motdepasse_securise_123 --authenticationDatabase admin
```

**Créer une base de données et un utilisateur :**

```javascript
// 1. Créer/utiliser une base de données "mabase"
use mabase

// 2. Créer un utilisateur avec droits de lecture/écriture uniquement sur "mabase"
db.createUser({
  user: "user_mabase",
  pwd: "password_user_123",
  roles: [
    { role: "readWrite", db: "mabase" }
  ]
})
```

**Résultat attendu :**

```javascript
{ ok: 1 }
```

**Explications :**

| Élément | Signification |
|---------|---------------|
| `user: "user_mabase"` | Nom du nouvel utilisateur |
| `pwd: "password_user_123"` | Mot de passe (à changer !) |
| `role: "readWrite"` | Peut lire et écrire dans la base |
| `db: "mabase"` | Uniquement sur la base "mabase" |

### B. Tester le nouvel utilisateur

**Quittez le shell actuel :**

```javascript
exit
```

**Reconnectez-vous avec le nouvel utilisateur :**

```bash
docker exec -it mongodb_secure mongosh -u user_mabase -p password_user_123 --authenticationDatabase mabase
```

**Testez les permissions :**

```javascript
// Sélectionner la base "mabase" (fonctionne)
use mabase

// Insérer un document (fonctionne)
db.test.insertOne({ nom: "Test", valeur: 42 })

// Lire les documents (fonctionne)
db.test.find()

// Essayer d'accéder à une autre base (ne fonctionne pas)
use admin
show collections  // ❌ Erreur : pas de permissions
```

Vous verrez une erreur de permissions : c'est normal ! Cet utilisateur n'a accès qu'à `mabase`.

---

## 🛡️ Étape 6 : Comprendre les rôles MongoDB

MongoDB propose plusieurs rôles prédéfinis avec différents niveaux de permissions :

### Rôles principaux

| Rôle | Permissions | Utilisation |
|------|-------------|-------------|
| `read` | Lecture seule | Utilisateurs qui consultent les données |
| `readWrite` | Lecture et écriture | Applications standards |
| `dbAdmin` | Administration de la base | Gestion des index, statistiques |
| `userAdmin` | Gestion des utilisateurs | Créer/modifier des utilisateurs |
| `dbOwner` | Tous les droits sur la base | Propriétaire de la base |
| `readAnyDatabase` | Lecture sur toutes les bases | Monitoring |
| `readWriteAnyDatabase` | Lecture/écriture sur toutes les bases | Applications multi-bases |
| `userAdminAnyDatabase` | Gestion des users sur toutes les bases | Administration |
| `dbAdminAnyDatabase` | Administration de toutes les bases | DBA |
| `root` | **TOUS LES DROITS** | Super-administrateur |

### Exemples de création d'utilisateurs avec différents rôles

**Utilisateur en lecture seule :**

```javascript
use mabase
db.createUser({
  user: "lecteur",
  pwd: "motdepasse_lecteur",
  roles: [
    { role: "read", db: "mabase" }
  ]
})
```

**Utilisateur avec plusieurs rôles :**

```javascript
use mabase
db.createUser({
  user: "admin_mabase",
  pwd: "motdepasse_admin_mabase",
  roles: [
    { role: "readWrite", db: "mabase" },
    { role: "dbAdmin", db: "mabase" }
  ]
})
```

**Utilisateur avec accès à plusieurs bases :**

```javascript
use admin
db.createUser({
  user: "multi_bases",
  pwd: "motdepasse_multi",
  roles: [
    { role: "readWrite", db: "base1" },
    { role: "readWrite", db: "base2" },
    { role: "read", db: "base3" }
  ]
})
```

---

## 🔧 Étape 7 : Gestion des utilisateurs

### A. Lister tous les utilisateurs

```javascript
// Se connecter en tant qu'admin
use admin

// Afficher tous les utilisateurs
db.system.users.find()

// Afficher les utilisateurs d'une base spécifique
use mabase
db.getUsers()
```

### B. Modifier le mot de passe d'un utilisateur

```javascript
// Se connecter en tant qu'admin
use admin

// Changer le mot de passe de "user_mabase"
db.changeUserPassword("user_mabase", "nouveau_motdepasse_securise")
```

### C. Modifier les rôles d'un utilisateur

**Ajouter un rôle :**

```javascript
use mabase
db.grantRolesToUser("user_mabase", [
  { role: "dbAdmin", db: "mabase" }
])
```

**Retirer un rôle :**

```javascript
use mabase
db.revokeRolesFromUser("user_mabase", [
  { role: "dbAdmin", db: "mabase" }
])
```

### D. Supprimer un utilisateur

```javascript
use mabase
db.dropUser("user_mabase")
```

---

## 🌐 Étape 8 : Format de l'URI de connexion

Pour vous connecter à MongoDB depuis vos applications, vous utiliserez une **URI de connexion**.

### Format général

```
mongodb://[username]:[password]@[host]:[port]/[database]?authSource=[auth_database]
```

### Exemples d'URI

**Utilisateur admin :**

```
mongodb://admin:motdepasse_securise_123@localhost:27017/?authSource=admin
```

**Utilisateur sur une base spécifique :**

```
mongodb://user_mabase:password_user_123@localhost:27017/mabase?authSource=mabase
```

**Pour une application Node.js (exemple) :**

```javascript
const { MongoClient } = require('mongodb');

const uri = "mongodb://user_mabase:password_user_123@localhost:27017/mabase?authSource=mabase";
const client = new MongoClient(uri);

async function connecter() {
  await client.connect();
  console.log("Connecté à MongoDB !");
}
```

---

## 🛑 Étape 9 : Gestion du conteneur

### Commandes utiles

```bash
# Voir les logs
docker-compose logs -f mongodb

# Arrêter MongoDB (données préservées)
docker-compose stop

# Redémarrer
docker-compose start

# Voir l'état
docker-compose ps
```

---

## 🧹 Étape 10 : Suppression complète

**⚠️ ATTENTION : Ceci supprime TOUTES les données et utilisateurs !**

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer les données
rm -rf data/

# 3. Supprimer le fichier de configuration
rm docker-compose.yml
```

---

## 🔐 Étape 11 : Bonnes pratiques de sécurité

### ✅ À FAIRE

1. **Utilisez des mots de passe forts** (au moins 12 caractères, complexes)
2. **Créez des utilisateurs spécifiques** pour chaque application/service
3. **Donnez le minimum de permissions** nécessaires (principe du moindre privilège)
4. **Ne partagez JAMAIS** le mot de passe `root/admin`
5. **Utilisez des fichiers `.env`** pour stocker les mots de passe (voir plus bas)
6. **Changez les mots de passe régulièrement**

### ❌ À ÉVITER

1. ❌ Utiliser l'utilisateur `admin` dans vos applications
2. ❌ Mettre des mots de passe en dur dans le code source
3. ❌ Donner des permissions `root` à tous les utilisateurs
4. ❌ Utiliser des mots de passe simples ou prédictibles
5. ❌ Versionner les fichiers contenant des mots de passe (Git)

---

## 📁 Bonus : Utiliser un fichier `.env` pour les mots de passe

Pour éviter de mettre les mots de passe directement dans le `docker-compose.yml`, utilisez un fichier `.env`.

### A. Créer un fichier `.env`

Dans le dossier `mongodb_avec_auth`, créez un fichier **`.env`** :

```bash
# Identifiants MongoDB
MONGO_ROOT_USERNAME=admin
MONGO_ROOT_PASSWORD=motdepasse_securise_123
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
      # Utilisation des variables du fichier .env
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: admin
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db
```

### C. Ajouter `.env` au `.gitignore`

**TRÈS IMPORTANT** : Ne jamais versionner le fichier `.env` !

Créez un fichier **`.gitignore`** :

```
# Ne pas versionner les mots de passe
.env
data/
```

---

## ❓ Questions fréquentes (FAQ)

**Q : J'ai oublié mon mot de passe admin, que faire ?**
R : Vous devez supprimer les données et recréer le conteneur :
```bash
docker-compose down
rm -rf data/
docker-compose up -d
```
⚠️ Toutes les données seront perdues !

**Q : Comment savoir si l'authentification est activée ?**
R : Essayez de vous connecter sans identifiants :
```bash
docker exec -it mongodb_secure mongosh
```
Si vous ne pouvez rien faire (erreur d'authentification), c'est activé.

**Q : Puis-je changer les identifiants après le premier démarrage ?**
R : Les variables `MONGO_INITDB_ROOT_*` ne fonctionnent qu'au **premier démarrage**. Pour changer ensuite, utilisez les commandes MongoDB (`db.changeUserPassword()`).

**Q : Comment donner tous les droits à un utilisateur ?**
R : Utilisez le rôle `root` :
```javascript
use admin
db.createUser({
  user: "superadmin",
  pwd: "password_super",
  roles: [ "root" ]
})
```
⚠️ À utiliser avec précaution !

**Q : Mon application ne peut pas se connecter, pourquoi ?**
R : Vérifiez :
1. Les identifiants (username/password)
2. Le nom de la base d'authentification (`authSource`)
3. Que l'utilisateur existe : `db.getUsers()`
4. Les permissions de l'utilisateur

---

## 🎓 Pour aller plus loin

Cette configuration avec authentification est adaptée pour :
- ✅ Développement sécurisé
- ✅ Environnements partagés
- ✅ Tests d'applications réelles
- ✅ Apprentissage des bonnes pratiques

**Prochaines étapes recommandées :**
- 👉 [5.3 Configuration avec IP fixe](03-config-ip-fixe.md)
- 👉 [5.4 MongoDB avec Mongo Express](04-mongodb-mongo-express.md)
- 👉 [Annexe D - Sécurité et bonnes pratiques](/annexes/D-securite-bonnes-pratiques.md)

---

## 📝 Résumé de ce que vous avez appris

| Concept | Ce que vous savez maintenant |
|---------|------------------------------|
| **Authentification** | Activer la sécurité avec `MONGO_INITDB_ROOT_*` |
| **Utilisateurs** | Créer des utilisateurs avec `db.createUser()` |
| **Rôles** | Comprendre et attribuer les rôles MongoDB |
| **Connexion** | Se connecter avec identifiants (shell et URI) |
| **Gestion** | Modifier, supprimer des utilisateurs |
| **Sécurité** | Bonnes pratiques et fichiers `.env` |

---

## 🎯 Points clés à retenir

- 🔐 **L'authentification est essentielle** dès que vous partagez votre environnement
- 👥 **Créez des utilisateurs spécifiques** pour chaque usage
- 🛡️ **Principe du moindre privilège** : donnez uniquement les permissions nécessaires
- 🔑 **Mots de passe forts** toujours
- 📁 **Fichiers `.env`** pour les secrets
- 🚫 **Ne versionnez jamais** les mots de passe

---

## 🆘 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. **Vérifiez les logs** : `docker-compose logs mongodb`
2. **Testez l'authentification** : `db.auth("user", "password")`
3. **Vérifiez les utilisateurs** : `db.getUsers()`
4. **Consultez l'annexe** : [E - Dépannage](/annexes/E-depannage.md)
5. **Documentation officielle** : [MongoDB Security](https://www.mongodb.com/docs/manual/security/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
