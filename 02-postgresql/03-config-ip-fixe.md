# PostgreSQL - Configuration avec IP Fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Par défaut, Docker attribue des adresses IP aléatoires aux conteneurs. Si vous redémarrez votre conteneur PostgreSQL, il peut obtenir une nouvelle adresse IP, ce qui complique la configuration d'autres services qui doivent s'y connecter.

Cette fiche vous guide pour assigner une **adresse IP fixe** à votre conteneur PostgreSQL, garantissant qu'il aura toujours la même adresse réseau.

**Ce que vous allez apprendre :**
- Pourquoi utiliser une IP fixe
- Créer un réseau Docker personnalisé
- Assigner une IP statique à PostgreSQL
- Connecter d'autres conteneurs au même réseau
- Gérer plusieurs bases de données avec IPs fixes

**Durée estimée :** 10-15 minutes

---

## 🎯 Objectif

À la fin de cette fiche, vous aurez :
- ✅ Un réseau Docker personnalisé
- ✅ PostgreSQL avec une IP fixe (ex: 172.20.0.10)
- ✅ La possibilité de connecter d'autres services facilement
- ✅ Une configuration stable et prévisible

---

## 📦 Prérequis

Avant de commencer :
- ✅ Avoir suivi la [Configuration basique](01-config-basique-docker-compose.md)
- ✅ Comprendre les [Concepts Docker - Réseaux](/00-introduction/02-concepts-docker.md)
- ✅ PostgreSQL doit fonctionner en configuration basique

---

## 🧠 Comprendre les Réseaux Docker

### IP Dynamique vs IP Fixe

**Sans configuration (IP dynamique) :**
```
Démarrage 1 : PostgreSQL → 172.17.0.2
Démarrage 2 : PostgreSQL → 172.17.0.5
Démarrage 3 : PostgreSQL → 172.17.0.3
```
👎 L'adresse change à chaque fois !

**Avec IP fixe :**
```
Démarrage 1 : PostgreSQL → 172.20.0.10
Démarrage 2 : PostgreSQL → 172.20.0.10
Démarrage 3 : PostgreSQL → 172.20.0.10
```
👍 Toujours la même adresse !

### Pourquoi Utiliser une IP Fixe ?

| Avantage | Description |
|----------|-------------|
| **Prévisibilité** | Vous savez toujours où trouver votre base de données |
| **Configuration stable** | Pas besoin de changer la config à chaque redémarrage |
| **Multi-conteneurs** | Facilite la connexion entre services (web → BDD) |
| **Documentation** | IP figée dans la doc, dans les scripts... |
| **Développement** | Configurations cohérentes entre développeurs |

### Quand Utiliser une IP Fixe ?

✅ **Utilisez une IP fixe si :**
- Vous avez plusieurs conteneurs qui communiquent (web + BDD)
- Vous devez configurer des pare-feu avec des règles spécifiques
- Vous voulez une configuration reproductible et documentée
- Vous testez des configurations réseau complexes

❌ **Pas nécessaire si :**
- Vous avez un seul conteneur
- Vous utilisez Docker Compose (il gère les noms de services automatiquement)
- Vous vous connectez uniquement depuis votre PC (localhost)

---

## 🚀 Méthode 1 : IP Fixe avec Docker Compose (Recommandée)

Cette méthode est la plus simple et la mieux intégrée avec Docker Compose.

### Étape 1 : Créer le Réseau Docker

**Pourquoi créer un réseau personnalisé ?**
Le réseau par défaut de Docker (`bridge`) ne permet pas d'assigner des IPs fixes. Nous devons créer notre propre réseau.

```bash
# Créer un réseau avec un sous-réseau défini
docker network create --subnet=172.20.0.0/16 postgres_network
```

**Explication :**
- `--subnet=172.20.0.0/16` : Plage d'adresses disponibles (172.20.0.1 à 172.20.255.254)
- `postgres_network` : Nom du réseau (choisissez un nom descriptif)

**Vérifier la création :**
```bash
docker network ls
```

**Vous devriez voir :**
```
NETWORK ID     NAME              DRIVER    SCOPE
abc123...      postgres_network  bridge    local
```

### Étape 2 : Modifier docker-compose.yml

Modifiez votre `docker-compose.yml` pour utiliser ce réseau :

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_dev
    restart: unless-stopped

    environment:
      POSTGRES_PASSWORD: changez_moi_123!
      POSTGRES_DB: ma_base
      POSTGRES_USER: mon_utilisateur

    ports:
      - "5432:5432"

    volumes:
      - ./data:/var/lib/postgresql/data

    # ✨ CONFIGURATION RÉSEAU
    networks:
      postgres_network:
        # Assignation de l'IP fixe
        ipv4_address: 172.20.0.10

# ✨ DÉCLARATION DU RÉSEAU EXTERNE
networks:
  postgres_network:
    external: true  # Le réseau existe déjà (créé manuellement)
```

### Étape 3 : Lancer PostgreSQL

```bash
# Arrêter l'ancien conteneur (si existant)
docker-compose down

# Démarrer avec la nouvelle configuration
docker-compose up -d
```

### Étape 4 : Vérifier l'IP Fixe

```bash
# Inspecter le conteneur
docker inspect postgres_dev | grep IPAddress

# Ou plus précis
docker inspect postgres_dev -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

**Résultat attendu :**
```
172.20.0.10
```

🎉 Votre PostgreSQL a maintenant une IP fixe !

---

## 🔧 Méthode 2 : IP Fixe Sans Réseau Externe

Si vous préférez que Docker Compose gère le réseau (sans création manuelle), utilisez cette méthode.

### docker-compose.yml Complet

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_dev
    restart: unless-stopped

    environment:
      POSTGRES_PASSWORD: changez_moi_123!
      POSTGRES_DB: ma_base
      POSTGRES_USER: mon_utilisateur

    ports:
      - "5432:5432"

    volumes:
      - ./data:/var/lib/postgresql/data

    networks:
      postgres_network:
        ipv4_address: 172.20.0.10

# ✨ RÉSEAU GÉRÉ PAR DOCKER COMPOSE
networks:
  postgres_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```

**Différences avec la Méthode 1 :**
- ❌ Pas besoin de créer le réseau manuellement
- ✅ Docker Compose crée et supprime le réseau automatiquement
- ⚠️ Le réseau est supprimé avec `docker-compose down` (mais recréé au prochain `up`)

### Lancer

```bash
docker-compose up -d
```

Docker Compose crée automatiquement le réseau `postgres_network` avec la bonne configuration.

---

## 🌐 Se Connecter avec l'IP Fixe

### Depuis un Autre Conteneur (Même Réseau)

Créons un conteneur web (Node.js par exemple) qui se connecte à PostgreSQL :

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_dev
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: changez_moi_123!
      POSTGRES_DB: ma_base
      POSTGRES_USER: mon_utilisateur
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
    networks:
      postgres_network:
        ipv4_address: 172.20.0.10

  # ✨ APPLICATION WEB
  web:
    image: node:18
    container_name: web_app
    restart: unless-stopped
    working_dir: /app
    volumes:
      - ./app:/app
    ports:
      - "3000:3000"
    networks:
      postgres_network:
        ipv4_address: 172.20.0.20
    environment:
      # Connexion à PostgreSQL via son IP fixe
      DB_HOST: 172.20.0.10
      DB_PORT: 5432
      DB_NAME: ma_base
      DB_USER: mon_utilisateur
      DB_PASSWORD: changez_moi_123!
    command: sh -c "npm install && npm start"

networks:
  postgres_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Résultat :**
- PostgreSQL : `172.20.0.10:5432`
- Web App : `172.20.0.20:3000`
- Les deux peuvent communiquer via leurs IPs fixes

**💡 Alternative (meilleure pratique) :** Au lieu de l'IP, utilisez le nom du service :
```yaml
environment:
  DB_HOST: postgres  # Docker Compose résout automatiquement
```

### Depuis Votre PC (Hôte)

```bash
# Via psql
psql -h 127.0.0.1 -p 5432 -U mon_utilisateur -d ma_base

# Via un client GUI (DBeaver, pgAdmin...)
# Hôte: localhost ou 127.0.0.1
# Port: 5432
```

**⚠️ Attention :** Depuis votre PC, vous utilisez toujours `localhost`, pas `172.20.0.10`. L'IP fixe est pour les connexions **entre conteneurs**.

### Depuis un Autre Réseau Docker

Si vous avez des conteneurs sur un autre réseau et qu'ils doivent accéder à PostgreSQL :

```bash
# Connecter un conteneur existant au réseau postgres_network
docker network connect postgres_network mon_autre_conteneur

# Maintenant mon_autre_conteneur peut accéder à 172.20.0.10
```

---

## 🎯 Configuration Multi-Bases avec IPs Fixes

Besoin de plusieurs bases de données ? Attribuez une IP fixe à chacune !

### docker-compose.yml Multi-Bases

```yaml
version: '3.8'

services:
  # PostgreSQL
  postgres:
    image: postgres:15
    container_name: postgres_dev
    environment:
      POSTGRES_PASSWORD: postgres_pass_123!
      POSTGRES_DB: postgres_db
      POSTGRES_USER: postgres_user
    ports:
      - "5432:5432"
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    networks:
      databases_network:
        ipv4_address: 172.25.0.10

  # MariaDB
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_dev
    environment:
      MARIADB_ROOT_PASSWORD: mariadb_pass_123!
      MARIADB_DATABASE: mariadb_db
      MARIADB_USER: mariadb_user
      MARIADB_PASSWORD: mariadb_pass_123!
    ports:
      - "3306:3306"
    volumes:
      - ./data/mariadb:/var/lib/mysql
    networks:
      databases_network:
        ipv4_address: 172.25.0.20

  # MongoDB
  mongodb:
    image: mongo:7
    container_name: mongodb_dev
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongo_user
      MONGO_INITDB_ROOT_PASSWORD: mongo_pass_123!
    ports:
      - "27017:27017"
    volumes:
      - ./data/mongodb:/data/db
    networks:
      databases_network:
        ipv4_address: 172.25.0.30

  # Redis
  redis:
    image: redis:7
    container_name: redis_dev
    command: redis-server --requirepass redis_pass_123!
    ports:
      - "6379:6379"
    volumes:
      - ./data/redis:/data
    networks:
      databases_network:
        ipv4_address: 172.25.0.40

networks:
  databases_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16
```

**Résultat :**
- PostgreSQL : `172.25.0.10:5432`
- MariaDB : `172.25.0.20:3306`
- MongoDB : `172.25.0.30:27017`
- Redis : `172.25.0.40:6379`

**Avantages :**
- ✅ Adresses prévisibles et documentées
- ✅ Facile à configurer dans vos applications
- ✅ Isolation réseau (pas de conflit avec d'autres projets)

---

## 🔍 Inspection et Diagnostic

### Voir les Détails du Réseau

```bash
# Inspecter le réseau
docker network inspect postgres_network
```

**Sortie (extrait) :**
```json
{
    "Name": "postgres_network",
    "Driver": "bridge",
    "IPAM": {
        "Config": [
            {
                "Subnet": "172.20.0.0/16",
                "Gateway": "172.20.0.1"
            }
        ]
    },
    "Containers": {
        "abc123...": {
            "Name": "postgres_dev",
            "IPv4Address": "172.20.0.10/16"
        }
    }
}
```

### Tester la Connectivité Entre Conteneurs

```bash
# Entrer dans un conteneur
docker exec -it postgres_dev bash

# Installer ping (si absent)
apt-get update && apt-get install -y iputils-ping

# Ping un autre conteneur sur le même réseau
ping 172.20.0.20

# Tester la connexion PostgreSQL depuis un autre conteneur
docker exec -it web_app bash
psql -h 172.20.0.10 -p 5432 -U mon_utilisateur -d ma_base
```

### Voir les Routes Réseau

```bash
# Voir les interfaces réseau du conteneur
docker exec postgres_dev ip addr show

# Voir la table de routage
docker exec postgres_dev ip route
```

---

## 🛠️ Gestion Avancée

### Changer l'IP Fixe

```yaml
# Dans docker-compose.yml, modifiez l'IP
networks:
  postgres_network:
    ipv4_address: 172.20.0.15  # Nouvelle IP
```

Puis :
```bash
docker-compose down
docker-compose up -d
```

### Ajouter Plusieurs IPs (Multi-Réseaux)

Un conteneur peut être connecté à plusieurs réseaux avec des IPs différentes :

```yaml
services:
  postgres:
    image: postgres:15
    networks:
      frontend_network:
        ipv4_address: 172.20.0.10
      backend_network:
        ipv4_address: 172.21.0.10

networks:
  frontend_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16

  backend_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

PostgreSQL sera accessible sur deux réseaux différents.

### Créer un Sous-Réseau Personnalisé

```bash
# Réseau avec plage d'IPs spécifique
docker network create \
  --subnet=10.10.0.0/24 \
  --gateway=10.10.0.1 \
  --ip-range=10.10.0.0/26 \
  mon_reseau_custom
```

**Explication :**
- `--subnet` : Plage totale (10.10.0.0 → 10.10.0.255)
- `--gateway` : IP de la passerelle
- `--ip-range` : Sous-plage utilisable pour les conteneurs (10.10.0.0 → 10.10.0.63)

---

## 🧹 Nettoyage

### Supprimer un Réseau (Méthode 1)

```bash
# 1. Arrêter tous les conteneurs utilisant le réseau
docker-compose down

# 2. Supprimer le réseau
docker network rm postgres_network

# 3. (Optionnel) Supprimer les données
rm -rf ./data
```

### Nettoyage Automatique (Méthode 2)

Avec la Méthode 2 (réseau géré par Docker Compose) :

```bash
# Tout supprimer (conteneurs + réseau)
docker-compose down

# Le réseau est automatiquement supprimé
```

### Supprimer Tous les Réseaux Inutilisés

```bash
# Supprimer tous les réseaux non utilisés
docker network prune

# Confirmation demandée
Are you sure you want to continue? [y/N] y
```

⚠️ **Attention :** Cela supprime TOUS les réseaux non attachés à un conteneur actif.

---

## ⚠️ Erreurs Courantes

### Erreur : "Address already in use"

**Cause :** L'IP `172.20.0.10` est déjà utilisée par un autre conteneur.

**Solution :**
```bash
# Voir qui utilise l'IP
docker network inspect postgres_network

# Changer l'IP dans docker-compose.yml
ipv4_address: 172.20.0.11  # Nouvelle IP disponible
```

### Erreur : "Pool overlaps with other one"

**Cause :** Le sous-réseau `172.20.0.0/16` chevauche un réseau existant.

**Solution :** Utilisez une autre plage :
```yaml
networks:
  postgres_network:
    ipam:
      config:
        - subnet: 172.30.0.0/16  # Plage différente
```

### Erreur : "Network not found"

**Cause :** Le réseau n'existe pas (Méthode 1 sans avoir créé le réseau).

**Solution :**
```bash
# Créer le réseau
docker network create --subnet=172.20.0.0/16 postgres_network

# Ou utiliser la Méthode 2 (réseau géré par Compose)
```

### Conteneur Ne Peut Pas Se Connecter

**Cause :** Pare-feu, règles iptables, ou réseau mal configuré.

**Diagnostic :**
```bash
# Ping depuis le conteneur
docker exec postgres_dev ping 172.20.0.1

# Vérifier les règles iptables (Linux)
sudo iptables -L -n -v

# Voir les logs Docker
docker logs postgres_dev
```

---

## 📊 Tableau Récapitulatif

| Aspect | Méthode 1 (Réseau Externe) | Méthode 2 (Réseau Compose) |
|--------|---------------------------|---------------------------|
| **Création réseau** | Manuel (`docker network create`) | Automatique (Docker Compose) |
| **Persistance** | Le réseau survit à `docker-compose down` | Supprimé avec `docker-compose down` |
| **Complexité** | Un peu plus complexe | Plus simple |
| **Use Case** | Plusieurs projets partagent le réseau | Réseau dédié à un projet |
| **Recommandation** | Production, infrastructure complexe | **Développement** (plus simple) |

---

## ✅ Checklist de Configuration

Avant de considérer votre configuration IP fixe prête :

### Réseau
- [ ] Réseau créé (Méthode 1) ou déclaré (Méthode 2)
- [ ] Sous-réseau défini (`172.20.0.0/16` ou autre)
- [ ] Pas de conflit avec réseaux existants

### docker-compose.yml
- [ ] Section `networks` configurée
- [ ] IP fixe assignée (`ipv4_address`)
- [ ] IP dans la plage du sous-réseau

### Tests
- [ ] Conteneur démarre sans erreur
- [ ] IP vérifiée avec `docker inspect`
- [ ] IP ne change pas après redémarrage
- [ ] Connexion entre conteneurs fonctionne (si applicable)

### Documentation
- [ ] IP documentée dans README ou commentaires
- [ ] Schéma réseau créé (si architecture complexe)

---

## 🚀 Prochaines Étapes

Maintenant que PostgreSQL a une IP fixe :

1. **Gérer les utilisateurs** → [Gestion des utilisateurs et BDD](04-gestion-utilisateurs-bdd.md)
2. **Ajouter pgAdmin** → [Configuration avec pgAdmin](05-config-avec-pgadmin.md)
3. **Créer une stack complète** → [Cas Pratiques](/cas-pratiques/)
4. **Comprendre les réseaux en détail** → [Annexe B - Gestion des réseaux](/annexes/B-gestion-reseaux.md)

---

## 📚 Ressources Complémentaires

### Documentation Officielle
- **Docker Networking** : [https://docs.docker.com/network/](https://docs.docker.com/network/)
- **Docker Compose Networking** : [https://docs.docker.com/compose/networking/](https://docs.docker.com/compose/networking/)
- **Docker Network Commands** : [https://docs.docker.com/engine/reference/commandline/network/](https://docs.docker.com/engine/reference/commandline/network/)

### Tutoriels
- **Understanding Docker Networking** : [https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client](https://www.digitalocean.com/community/tutorials/)

### Outils
- **Docker Network Visualizer** : [https://github.com/MuhammedKalkan/OpenLens](https://github.com/MuhammedKalkan/OpenLens)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
