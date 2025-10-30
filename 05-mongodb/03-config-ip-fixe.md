# 5.3 - Configuration de MongoDB avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans les fiches précédentes, nous avons mis en place MongoDB de manière basique ([5.1](01-config-basique-docker-compose.md)) puis avec authentification ([5.2](02-config-avec-authentification.md)). Ces configurations fonctionnent très bien, mais **l'adresse IP du conteneur MongoDB change à chaque redémarrage**.

### 🤔 Pourquoi c'est un problème ?

**Sans IP fixe :**
```
Premier démarrage  → MongoDB a l'IP 172.17.0.2
Redémarrage        → MongoDB a l'IP 172.17.0.5  ❌ Changé !
```

**Conséquences :**
- ❌ Vos applications qui utilisent l'IP directement ne fonctionnent plus
- ❌ Configurations réseau complexes difficiles à maintenir
- ❌ Communication entre conteneurs imprévisible

**Avec IP fixe :**
```
Premier démarrage  → MongoDB a l'IP 172.18.0.10
Redémarrage        → MongoDB a l'IP 172.18.0.10  ✅ Stable !
Toujours pareil    → MongoDB a l'IP 172.18.0.10  ✅ Prévisible !
```

Dans cette fiche, nous allons créer un **réseau Docker personnalisé** et assigner une **adresse IP fixe** à notre conteneur MongoDB.

---

## 🎯 Objectifs de cette fiche

À la fin de ce tutoriel, vous saurez :

- ✅ Créer un réseau Docker personnalisé avec une plage d'adresses IP
- ✅ Assigner une IP fixe à votre conteneur MongoDB
- ✅ Comprendre les concepts de réseau Docker
- ✅ Connecter d'autres conteneurs sur le même réseau
- ✅ Nettoyer proprement les réseaux Docker

---

## 🛠️ Prérequis

Avant de commencer :

- **Docker** et **Docker Compose** installés
- Avoir suivi les fiches [5.1](01-config-basique-docker-compose.md) et [5.2](02-config-avec-authentification.md) (recommandé)
- Comprendre les bases des réseaux (notion d'adresse IP)

---

## 🧠 Étape 1 : Comprendre les réseaux Docker

### A. Qu'est-ce qu'un réseau Docker ?

Un **réseau Docker** est comme un réseau local virtuel où vos conteneurs peuvent communiquer entre eux. Par défaut, Docker crée un réseau automatique, mais il ne permet pas d'assigner des IP fixes.

### B. Types de réseaux Docker

| Type | Description | Utilisation |
|------|-------------|-------------|
| **bridge** (par défaut) | Réseau isolé sur l'hôte | Conteneurs sur la même machine |
| **host** | Utilise le réseau de l'hôte | Performances maximales |
| **none** | Aucun réseau | Conteneurs isolés |
| **bridge personnalisé** | Réseau avec configuration custom | **IP fixes, DNS personnalisé** ✅ |

Nous allons utiliser un **bridge personnalisé** pour avoir le contrôle sur les adresses IP.

### C. Notion de sous-réseau (subnet)

Un sous-réseau définit une **plage d'adresses IP disponibles**.

**Exemple :**
```
Subnet : 172.18.0.0/16
Signifie : de 172.18.0.1 à 172.18.255.254
         ≈ 65 000 adresses disponibles
```

**Analogie :** C'est comme un immeuble avec 65 000 appartements numérotés. Nous allons réserver l'appartement n°10 pour MongoDB.

---

## 📡 Étape 2 : Création du réseau Docker personnalisé

### A. Créer le réseau

Ouvrez un terminal et exécutez cette commande **une seule fois** :

```bash
docker network create --subnet=172.18.0.0/16 mongodb_net
```

**Explication de la commande :**

| Élément | Signification |
|---------|---------------|
| `docker network create` | Commande pour créer un réseau |
| `--subnet=172.18.0.0/16` | Définit la plage d'IP (de 172.18.0.1 à 172.18.255.254) |
| `mongodb_net` | Nom du réseau (vous pouvez le changer) |

**Sortie attendue :**

```
f8a3b2c1d4e5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1
```

C'est l'identifiant unique du réseau créé.

### B. Vérifier la création du réseau

```bash
docker network ls
```

**Résultat attendu :**

```
NETWORK ID     NAME          DRIVER    SCOPE
abc123def456   bridge        bridge    local
def789ghi012   mongodb_net   bridge    local     ← Notre réseau !
```

### C. Inspecter le réseau

Pour voir les détails du réseau :

```bash
docker network inspect mongodb_net
```

Vous verrez des informations détaillées, notamment :

```json
"Subnet": "172.18.0.0/16",
"Gateway": "172.18.0.1"
```

---

## 📁 Étape 3 : Préparation de l'environnement

### Créer un nouveau dossier de projet

```bash
# Créer le dossier
mkdir mongodb_ip_fixe

# Se déplacer dedans
cd mongodb_ip_fixe

# Créer le dossier pour les données
mkdir data
```

---

## 📄 Étape 4 : Configuration du `docker-compose.yml`

Créez un fichier **`docker-compose.yml`** avec le contenu suivant :

```yaml
version: '3.8'

services:
  mongodb:
    # Image officielle MongoDB
    image: mongo:7.0

    # Nom du conteneur
    container_name: mongodb_fixed_ip

    # Redémarre automatiquement
    restart: unless-stopped

    # Authentification (optionnel mais recommandé)
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: motdepasse_securise_123

    # Port pour accéder depuis l'hôte
    ports:
      - "27017:27017"

    # Volume pour la persistance
    volumes:
      - ./data:/data/db

    # ⭐ NOUVEAUTÉ : Configuration réseau avec IP fixe
    networks:
      mongodb_net:
        ipv4_address: 172.18.0.10

# ⭐ NOUVEAUTÉ : Déclaration du réseau externe
networks:
  mongodb_net:
    external: true
```

### 📖 Explication des nouveautés

| Section | Explication |
|---------|-------------|
| `networks:` (dans le service) | Configure le réseau pour ce conteneur |
| `mongodb_net:` | Nom du réseau à utiliser |
| `ipv4_address: 172.18.0.10` | **IP FIXE assignée au conteneur** |
| `networks:` (en bas) | Déclaration globale des réseaux |
| `external: true` | Indique que le réseau existe déjà (créé à l'étape 2) |

### 🎨 Pourquoi 172.18.0.10 ?

Vous pouvez choisir n'importe quelle IP dans la plage `172.18.0.1` à `172.18.255.254`, **sauf** :
- ❌ `172.18.0.1` (réservée pour la passerelle/gateway)
- ❌ `172.18.0.0` (adresse du réseau)
- ❌ `172.18.255.255` (adresse de broadcast)

**Bonnes pratiques :**
- ✅ Utilisez des IP basses (`172.18.0.10`, `172.18.0.11`, etc.) pour faciliter la mémorisation
- ✅ Documentez vos IP (notez quel conteneur a quelle IP)
- ✅ Laissez de l'espace entre chaque IP (10, 20, 30...) pour ajouter des services plus tard

---

## ▶️ Étape 5 : Démarrage de MongoDB avec IP fixe

Lancez MongoDB :

```bash
docker-compose up -d
```

**Sortie attendue :**

```
Creating mongodb_fixed_ip ... done
```

### ✅ Vérification du démarrage

```bash
docker-compose ps
```

---

## 🔍 Étape 6 : Vérification de l'IP fixe

### A. Méthode 1 : Avec `docker inspect`

```bash
docker inspect mongodb_fixed_ip | grep IPAddress
```

**Résultat attendu :**

```json
"IPAddress": "",
"SecondaryIPAddresses": null,
"IPAddress": "172.18.0.10",    ← Notre IP fixe !
```

### B. Méthode 2 : Inspection complète du réseau

```bash
docker network inspect mongodb_net
```

Cherchez la section `Containers`, vous verrez :

```json
"Containers": {
    "abc123...": {
        "Name": "mongodb_fixed_ip",
        "IPv4Address": "172.18.0.10/16",    ← Confirmé !
        ...
    }
}
```

### C. Test de persistance de l'IP

**Redémarrez le conteneur plusieurs fois :**

```bash
# Test 1 : Redémarrage simple
docker-compose restart
docker inspect mongodb_fixed_ip | grep "IPAddress"

# Test 2 : Arrêt et redémarrage
docker-compose stop
docker-compose start
docker inspect mongodb_fixed_ip | grep "IPAddress"

# Test 3 : Suppression et recréation
docker-compose down
docker-compose up -d
docker inspect mongodb_fixed_ip | grep "IPAddress"
```

**À chaque fois, vous devriez voir : `"IPAddress": "172.18.0.10"` ✅**

---

## 🔗 Étape 7 : Connexion à MongoDB via l'IP fixe

### A. Depuis le shell MongoDB

**Via localhost (depuis votre machine) :**

```bash
docker exec -it mongodb_fixed_ip mongosh -u admin -p motdepasse_securise_123 --authenticationDatabase admin
```

**Via l'IP fixe (depuis un autre conteneur du même réseau) :**

Nous allons créer un conteneur temporaire pour tester :

```bash
# Créer un conteneur Ubuntu connecté au même réseau
docker run -it --rm --network mongodb_net ubuntu bash

# Une fois dans le conteneur Ubuntu, installer mongosh
apt-get update && apt-get install -y wget gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-7.0.list
apt-get update && apt-get install -y mongodb-mongosh

# Se connecter via l'IP fixe
mongosh mongodb://admin:motdepasse_securise_123@172.18.0.10:27017/?authSource=admin
```

✅ Si la connexion fonctionne, votre IP fixe est bien configurée !

### B. Depuis MongoDB Compass

**URI de connexion avec IP fixe :**

```
mongodb://admin:motdepasse_securise_123@172.18.0.10:27017/?authSource=admin
```

**Ou via localhost (fonctionne toujours) :**

```
mongodb://admin:motdepasse_securise_123@localhost:27017/?authSource=admin
```

Les deux fonctionnent ! La différence :
- `localhost` : Connexion depuis votre machine via le port exposé (27017)
- `172.18.0.10` : Connexion directe via le réseau Docker (utile pour d'autres conteneurs)

---

## 🌐 Étape 8 : Ajouter d'autres services sur le même réseau

L'intérêt principal d'un réseau personnalisé est de connecter **plusieurs conteneurs** entre eux.

### Exemple : Ajouter une application Node.js

Modifiez votre `docker-compose.yml` :

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb_fixed_ip
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: motdepasse_securise_123
    ports:
      - "27017:27017"
    volumes:
      - ./data:/data/db
    networks:
      mongodb_net:
        ipv4_address: 172.18.0.10

  # ⭐ NOUVEAU SERVICE : Application Node.js
  app:
    image: node:18
    container_name: node_app
    restart: unless-stopped
    working_dir: /app
    command: node server.js
    ports:
      - "3000:3000"
    volumes:
      - ./app:/app
    networks:
      mongodb_net:
        ipv4_address: 172.18.0.20    # Autre IP fixe
    # Variable d'environnement avec l'IP de MongoDB
    environment:
      - MONGO_URL=mongodb://admin:motdepasse_securise_123@172.18.0.10:27017/?authSource=admin

networks:
  mongodb_net:
    external: true
```

**Avantages :**
- ✅ L'application Node.js connaît toujours l'adresse de MongoDB (`172.18.0.10`)
- ✅ Pas de problème si un conteneur redémarre
- ✅ Communication rapide et directe entre conteneurs

---

## 📊 Étape 9 : Tableau récapitulatif des adresses

| Service | Nom du conteneur | IP fixe | Port | Accessible depuis |
|---------|------------------|---------|------|-------------------|
| MongoDB | `mongodb_fixed_ip` | `172.18.0.10` | 27017 | Réseau `mongodb_net` |
| MongoDB (port exposé) | - | `localhost` | 27017 | Votre machine |
| Application (exemple) | `node_app` | `172.18.0.20` | 3000 | Réseau `mongodb_net` |

---

## 🛑 Étape 10 : Gestion et nettoyage

### A. Arrêter les conteneurs (garder le réseau)

```bash
# Arrêter les conteneurs
docker-compose stop

# Redémarrer
docker-compose start
```

Le réseau `mongodb_net` reste intact.

### B. Supprimer les conteneurs (garder le réseau)

```bash
# Supprimer les conteneurs mais pas le réseau
docker-compose down
```

Le réseau `mongodb_net` existe toujours (car marqué comme `external: true`).

### C. Suppression complète (conteneurs + réseau)

**⚠️ ATTENTION : Toutes les données et la configuration réseau seront perdues !**

```bash
# 1. Arrêter et supprimer les conteneurs
docker-compose down

# 2. Supprimer les données
rm -rf data/

# 3. Supprimer le réseau personnalisé
docker network rm mongodb_net

# 4. (Optionnel) Supprimer les fichiers de configuration
rm docker-compose.yml
```

### D. Vérifier les réseaux existants

```bash
# Lister tous les réseaux
docker network ls

# Voir les détails d'un réseau
docker network inspect mongodb_net

# Supprimer un réseau (si aucun conteneur ne l'utilise)
docker network rm mongodb_net
```

---

## 🧹 Étape 11 : Nettoyage des réseaux inutilisés

Avec le temps, des réseaux inutilisés peuvent s'accumuler.

### A. Voir les réseaux non utilisés

```bash
docker network ls --filter "dangling=true"
```

### B. Supprimer tous les réseaux non utilisés

```bash
docker network prune
```

**Confirmation demandée :**

```
WARNING! This will remove all custom networks not used by at least one container.
Are you sure you want to continue? [y/N] y
```

**Cette commande ne supprimera PAS :**
- Les réseaux actuellement utilisés par des conteneurs
- Les réseaux par défaut (`bridge`, `host`, `none`)

---

## ❓ Questions fréquentes (FAQ)

**Q : Puis-je utiliser n'importe quelle plage d'IP ?**
R : Oui, mais il est recommandé d'utiliser des plages privées :
- `172.16.0.0/12` (de 172.16.0.0 à 172.31.255.255)
- `192.168.0.0/16` (de 192.168.0.0 à 192.168.255.255)
- `10.0.0.0/8` (de 10.0.0.0 à 10.255.255.255)

**Q : Que se passe-t-il si j'assigne la même IP à deux conteneurs ?**
R : Docker refusera de démarrer le second conteneur avec une erreur :
```
Error response from daemon: Address already in use
```

**Q : Puis-je changer l'IP d'un conteneur en cours d'exécution ?**
R : Non, vous devez :
1. Arrêter le conteneur : `docker-compose down`
2. Modifier l'IP dans `docker-compose.yml`
3. Redémarrer : `docker-compose up -d`

**Q : Mon réseau entre en conflit avec mon réseau local, que faire ?**
R : Choisissez une autre plage d'IP. Par exemple :
- Si votre réseau local est `192.168.1.x`, utilisez `172.18.0.0/16`
- Si vous avez déjà `172.18.x.x`, utilisez `172.19.0.0/16`

**Q : Puis-je connecter un conteneur à plusieurs réseaux ?**
R : Oui ! Exemple :
```yaml
networks:
  mongodb_net:
    ipv4_address: 172.18.0.10
  autre_reseau:
    ipv4_address: 192.168.50.10
```

**Q : Est-ce que l'IP fixe fonctionne avec `docker run` ?**
R : Oui, mais c'est plus complexe :
```bash
docker run -d \
  --name mongodb_fixed_ip \
  --network mongodb_net \
  --ip 172.18.0.10 \
  -p 27017:27017 \
  -v $(pwd)/data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=motdepasse_securise_123 \
  mongo:7.0
```
Docker Compose est **beaucoup plus simple** ! 😊

---

## 🎓 Pour aller plus loin

Cette configuration avec IP fixe est idéale pour :
- ✅ Architectures multi-conteneurs stables
- ✅ Développement d'applications microservices
- ✅ Tests d'intégration avec plusieurs services
- ✅ Environnements où la prévisibilité est importante

**Prochaines étapes recommandées :**
- 👉 [5.4 MongoDB avec Mongo Express](04-mongodb-mongo-express.md)
- 👉 [5.5 Replica Set simple](05-replica-set-simple.md)
- 👉 [Annexe B - Gestion des réseaux Docker](/annexes/B-gestion-reseaux.md)
- 👉 [Cas pratique 02 - Stack MEAN](/cas-pratiques/02-stack-mean.md)

---

## 📝 Résumé de ce que vous avez appris

| Concept | Ce que vous savez maintenant |
|---------|------------------------------|
| **Réseaux Docker** | Créer un réseau personnalisé avec `docker network create` |
| **Sous-réseau** | Définir une plage d'IP avec `--subnet` |
| **IP fixe** | Assigner une IP statique avec `ipv4_address` |
| **Réseau externe** | Utiliser un réseau existant avec `external: true` |
| **Multi-conteneurs** | Connecter plusieurs services sur le même réseau |
| **Nettoyage** | Supprimer proprement les réseaux Docker |

---

## 🎯 Points clés à retenir

- 🌐 **Un réseau personnalisé** permet d'assigner des IP fixes
- 📌 **IP fixe = prévisibilité** pour vos architectures multi-conteneurs
- 🔗 **Communication inter-conteneurs** simplifiée
- 📊 **Documentez vos IP** pour éviter les conflits
- 🧹 **Nettoyez régulièrement** les réseaux inutilisés
- ⚙️ **Docker Compose simplifie** la gestion des réseaux

---

## 🔧 Commandes essentielles à retenir

```bash
# Créer un réseau
docker network create --subnet=172.18.0.0/16 mongodb_net

# Lister les réseaux
docker network ls

# Inspecter un réseau
docker network inspect mongodb_net

# Vérifier l'IP d'un conteneur
docker inspect <conteneur> | grep IPAddress

# Supprimer un réseau
docker network rm mongodb_net

# Nettoyer les réseaux inutilisés
docker network prune
```

---

## 🆘 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. **Vérifiez que le réseau existe** : `docker network ls`
2. **Inspectez le réseau** : `docker network inspect mongodb_net`
3. **Vérifiez les conflits d'IP** : Aucun autre conteneur ne doit utiliser `172.18.0.10`
4. **Consultez les logs** : `docker-compose logs mongodb`
5. **Annexe** : [B - Gestion des réseaux](/annexes/B-gestion-reseaux.md)
6. **Documentation Docker** : [Docker Networks](https://docs.docker.com/network/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
