# 5.1 - Configuration basique de MongoDB avec docker-compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**MongoDB** est une base de données NoSQL orientée documents, très populaire pour les applications web modernes. Contrairement aux bases de données relationnelles (SQL), MongoDB stocke les données sous forme de documents JSON flexibles, ce qui facilite le développement rapide et l'adaptation aux changements de structure.

Dans cette fiche, nous allons mettre en place une instance MongoDB simple avec Docker, sans authentification (idéal pour débuter et faire des tests en local).

---

## 🎯 Objectifs de cette fiche

À la fin de ce tutoriel, vous saurez :

- ✅ Démarrer une instance MongoDB avec `docker-compose`
- ✅ Accéder à MongoDB depuis votre machine (client GUI ou ligne de commande)
- ✅ Persister vos données même après suppression du conteneur
- ✅ Gérer le cycle de vie du conteneur (démarrage, arrêt, suppression)

---

## 🛠️ Prérequis

Avant de commencer, assurez-vous d'avoir :

- **Docker** installé sur votre machine
- **Docker Compose** (généralement inclus avec Docker Desktop)
- Un éditeur de texte (VS Code, Notepad++, nano, vim...)
- Des notions de base sur le terminal/ligne de commande

**Vérification rapide :**

```bash
# Vérifier Docker
docker --version

# Vérifier Docker Compose
docker-compose --version
```

---

## 📁 Étape 1 : Préparation de l'environnement

### A. Créer un dossier pour le projet

Créez un nouveau dossier pour organiser votre configuration MongoDB :

```bash
# Créer le dossier
mkdir mongodb_basique

# Se déplacer dedans
cd mongodb_basique
```

### B. Créer le dossier de données

MongoDB aura besoin d'un endroit pour stocker ses données de manière persistante :

```bash
# Créer le dossier data
mkdir data
```

Votre structure ressemble maintenant à ceci :

```
mongodb_basique/
└── data/          (vide pour l'instant)
```

---

## 📄 Étape 2 : Création du fichier `docker-compose.yml`

Dans le dossier `mongodb_basique`, créez un fichier nommé **`docker-compose.yml`** avec le contenu suivant :

```yaml
version: '3.8'

services:
  mongodb:
    # Image officielle MongoDB (version 7.0)
    image: mongo:7.0

    # Nom du conteneur (facilite les commandes docker)
    container_name: mongodb_dev

    # Redémarre automatiquement le conteneur sauf si arrêté manuellement
    restart: unless-stopped

    # Port pour accéder à MongoDB depuis votre machine
    # Format : "port_hôte:port_conteneur"
    ports:
      - "27017:27017"

    # Volume pour la persistance des données
    # Les données seront stockées dans le dossier ./data
    volumes:
      - ./data:/data/db
```

### 📖 Explication ligne par ligne

| Ligne | Signification |
|-------|---------------|
| `version: '3.8'` | Version du format docker-compose utilisé |
| `services:` | Début de la liste des services (conteneurs) |
| `mongodb:` | Nom du service (vous pouvez le changer) |
| `image: mongo:7.0` | Image Docker officielle de MongoDB version 7.0 |
| `container_name: mongodb_dev` | Nom du conteneur créé (facilite son identification) |
| `restart: unless-stopped` | Redémarre automatiquement si le système reboot |
| `ports: - "27017:27017"` | Expose le port 27017 de MongoDB sur votre machine |
| `volumes: - ./data:/data/db` | Les données sont sauvegardées dans le dossier `./data` |

---

## ▶️ Étape 3 : Démarrage de MongoDB

Maintenant que tout est prêt, lancez MongoDB avec une seule commande :

```bash
docker-compose up -d
```

**Que fait cette commande ?**

- `docker-compose` : Outil pour gérer plusieurs conteneurs
- `up` : Démarre les services définis dans le fichier
- `-d` : Mode "detached" (arrière-plan), le terminal reste libre

**Sortie attendue :**

```
Creating network "mongodb_basique_default" with the default driver
Creating mongodb_dev ... done
```

### ✅ Vérification du démarrage

Vérifiez que le conteneur fonctionne bien :

```bash
docker-compose ps
```

Vous devriez voir quelque chose comme :

```
    Name                  Command             State            Ports
-------------------------------------------------------------------------------
mongodb_dev   docker-entrypoint.sh mongod   Up      0.0.0.0:27017->27017/tcp
```

Le statut **"Up"** confirme que MongoDB est en cours d'exécution.

---

## 🔍 Étape 4 : Accéder à MongoDB

### A. Depuis le shell MongoDB (ligne de commande)

Pour vous connecter directement au shell MongoDB à l'intérieur du conteneur :

```bash
docker exec -it mongodb_dev mongosh
```

**Explication :**
- `docker exec` : Exécute une commande dans un conteneur en cours
- `-it` : Mode interactif avec terminal
- `mongodb_dev` : Nom de notre conteneur
- `mongosh` : Shell MongoDB (remplace l'ancien `mongo`)

**Une fois connecté, vous verrez :**

```
Current Mongosh Log ID:	...
Connecting to:		mongodb://127.0.0.1:27017
Using MongoDB:		7.0.x
Using Mongosh:		2.x.x

test>
```

### B. Commandes MongoDB de base

Une fois dans le shell, vous pouvez tester quelques commandes :

```javascript
// Afficher les bases de données existantes
show dbs

// Créer/utiliser une base de données "mabase"
use mabase

// Insérer un document dans la collection "users"
db.users.insertOne({ nom: "Dupont", prenom: "Jean", age: 30 })

// Afficher tous les documents de la collection
db.users.find()

// Quitter le shell
exit
```

### C. Depuis un client graphique

Vous pouvez également utiliser des outils graphiques pour vous connecter à MongoDB :

**Clients recommandés :**
- **MongoDB Compass** (officiel, gratuit) : [Télécharger ici](https://www.mongodb.com/products/compass)
- **NoSQLBooster** (anciennement MongoBooster)
- **Robo 3T** (Studio 3T)

**Paramètres de connexion :**

| Paramètre | Valeur |
|-----------|--------|
| **Hôte** | `localhost` ou `127.0.0.1` |
| **Port** | `27017` |
| **Authentification** | Aucune (mode basique) |

---

## 📊 Étape 5 : Vérification de la persistance des données

L'un des avantages de notre configuration est que les données survivent à l'arrêt du conteneur.

### Test de persistance

1. **Insérez des données** (via le shell ou un client GUI)

```bash
docker exec -it mongodb_dev mongosh
```

```javascript
use testpersistence
db.documents.insertOne({ message: "Ces données doivent persister" })
db.documents.find()
exit
```

2. **Arrêtez le conteneur**

```bash
docker-compose stop
```

3. **Vérifiez que le dossier `data` contient des fichiers**

```bash
ls -la data/
```

Vous devriez voir des fichiers MongoDB (`.wt`, `WiredTiger.*`, etc.)

4. **Redémarrez le conteneur**

```bash
docker-compose start
```

5. **Reconnectez-vous et vérifiez vos données**

```bash
docker exec -it mongodb_dev mongosh
```

```javascript
use testpersistence
db.documents.find()
```

Vos données sont toujours là ! 🎉

---

## 🛑 Étape 6 : Gestion du conteneur

### Commandes utiles

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Arrêter MongoDB (données préservées)
docker-compose stop

# Redémarrer MongoDB
docker-compose start

# Redémarrer complètement (stop + start)
docker-compose restart

# Voir l'état du conteneur
docker-compose ps

# Voir les statistiques de ressources (CPU, RAM)
docker stats mongodb_dev
```

---

## 🧹 Étape 7 : Suppression complète (nettoyage)

Si vous souhaitez tout supprimer (conteneur + données), suivez ces étapes :

### A. Arrêter et supprimer le conteneur

```bash
# Arrête et supprime le conteneur (mais pas les données)
docker-compose down
```

### B. Supprimer les données

**⚠️ ATTENTION : Cette action est irréversible !**

```bash
# Supprime le dossier contenant toutes les données MongoDB
rm -rf data/

# Sur Windows (PowerShell)
Remove-Item -Recurse -Force data
```

### C. Supprimer les fichiers de configuration

Si vous voulez également supprimer le fichier de configuration :

```bash
# Supprimer le fichier docker-compose.yml
rm docker-compose.yml
```

---

## 🎓 Pour aller plus loin

Cette configuration basique est parfaite pour :
- ✅ Apprendre MongoDB
- ✅ Développement en local
- ✅ Tests rapides
- ✅ Prototypes

**Mais elle n'est PAS adaptée pour :**
- ❌ Production (pas de sécurité)
- ❌ Accès depuis d'autres machines (pas d'authentification)
- ❌ Environnements multi-utilisateurs

**Prochaines étapes recommandées :**
- 👉 [5.2 Configuration avec authentification](02-config-avec-authentification.md)
- 👉 [5.3 Configuration avec IP fixe](03-config-ip-fixe.md)
- 👉 [5.4 MongoDB avec Mongo Express (interface web)](04-mongodb-mongo-express.md)

---

## 📝 Résumé de ce que vous avez appris

| Concept | Ce que vous savez maintenant |
|---------|------------------------------|
| **Docker Compose** | Gérer MongoDB avec un simple fichier YAML |
| **Volumes** | Persister les données avec `./data:/data/db` |
| **Ports** | Exposer MongoDB sur `localhost:27017` |
| **Shell MongoDB** | Se connecter avec `docker exec -it mongodb_dev mongosh` |
| **Cycle de vie** | Démarrer, arrêter, supprimer proprement |

---

## ❓ Questions fréquentes (FAQ)

**Q : Puis-je changer le port 27017 ?**
R : Oui ! Modifiez la ligne `ports:` dans le `docker-compose.yml` :
```yaml
ports:
  - "27018:27017"  # MongoDB accessible sur le port 27018 de votre machine
```

**Q : Comment mettre à jour la version de MongoDB ?**
R : Changez `image: mongo:7.0` en `image: mongo:8.0` (ou la version souhaitée) puis :
```bash
docker-compose down
docker-compose up -d
```

**Q : Mes données prennent beaucoup de place, comment les réduire ?**
R : MongoDB utilise beaucoup d'espace pour la performance. Vous pouvez :
- Supprimer les bases de test inutiles
- Utiliser `db.collection.drop()` pour supprimer des collections
- Compacter la base : `db.runCommand({ compact: 'nom_collection' })`

**Q : MongoDB refuse de démarrer, que faire ?**
R : Consultez les logs pour voir l'erreur :
```bash
docker-compose logs mongodb
```
Les erreurs courantes : port déjà utilisé, permissions sur le dossier `data/`

**Q : Comment sauvegarder ma base de données ?**
R : Utilisez `mongodump` :
```bash
docker exec mongodb_dev mongodump --out=/data/backup
```
Le backup sera dans `./data/backup/` sur votre machine.

---

## 🎯 Points clés à retenir

- 🐳 **Docker facilite l'installation** : Pas besoin d'installer MongoDB sur votre système
- 💾 **Les volumes assurent la persistance** : Vos données survivent aux redémarrages
- 🔧 **docker-compose simplifie la gestion** : Une commande pour tout contrôler
- 🚀 **Configuration basique = démarrage rapide** : Parfait pour apprendre et tester
- 🔒 **Pas de sécurité = pas pour la production** : Toujours ajouter l'authentification en environnement réel

---

## 🆘 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. **Vérifiez les logs** : `docker-compose logs mongodb`
2. **Consultez l'annexe** : [E - Dépannage](/annexes/E-depannage.md)
3. **Documentation officielle** : [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
4. **Redémarrez proprement** :
   ```bash
   docker-compose down
   docker-compose up -d
   ```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
