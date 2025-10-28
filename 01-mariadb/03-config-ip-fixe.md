# Configuration de MariaDB avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous apprend à assigner une **adresse IP statique** à votre conteneur MariaDB. C'est particulièrement utile quand vous avez plusieurs conteneurs qui doivent communiquer entre eux de manière prévisible.

**Ce que vous allez apprendre :**
- Comprendre pourquoi utiliser une IP fixe
- Créer un réseau Docker personnalisé
- Assigner une IP statique à votre conteneur MariaDB
- Faire communiquer plusieurs conteneurs via leurs IP fixes
- Combiner IP fixe avec configuration personnalisée (my.cnf)

**Durée estimée :** 20 minutes

---

## 🎯 Prérequis

Avant de commencer, vous devez :

- ✅ Avoir suivi la [fiche 1.1 - Configuration basique](01-config-basique-docker-compose.md)
- ✅ Comprendre les bases des réseaux (optionnel mais utile)
- ✅ Savoir utiliser Docker Compose

---

## 🤔 Pourquoi utiliser une IP fixe ?

### Comportement par défaut de Docker

Par défaut, Docker assigne des IP **dynamiques** à vos conteneurs :
- Les IP changent à chaque redémarrage
- Difficile de prévoir quelle IP aura un conteneur
- Communication entre conteneurs via les noms (DNS interne)

### Avantages d'une IP fixe

✅ **Prévisibilité** : Vous connaissez toujours l'IP de votre conteneur
✅ **Configuration externe** : Applications qui nécessitent une IP spécifique
✅ **Débogage facilité** : Connexion directe via IP pour les tests
✅ **Documentation claire** : "Le serveur de base de données est sur 172.18.0.10"
✅ **Règles réseau** : Facilite la configuration de pare-feu ou proxy

### Cas d'usage typiques

| Situation | Exemple |
|-----------|---------|
| **Architecture micro-services** | Plusieurs conteneurs (API, BDD, Cache) avec IPs fixes |
| **Configuration applicative** | Votre app lit une IP depuis un fichier de config |
| **Tests de réseau** | Simuler une infrastructure réelle avec IPs spécifiques |
| **Environnement multi-BDD** | PostgreSQL sur .10, MariaDB sur .20, MongoDB sur .30 |

---

## 🌐 Comprendre les réseaux Docker

### Concepts de base

Docker utilise des **réseaux virtuels** pour connecter les conteneurs :

```
┌────────────────────────────────────────────────────┐
│  Réseau Docker : mariadb_net (172.18.0.0/16)       │
│                                                    │
│  ┌──────────────────┐      ┌──────────────────┐    │
│  │  MariaDB         │      │  Application     │    │
│  │  172.18.0.10     │◄────►│  172.18.0.50     │    │
│  └──────────────────┘      └──────────────────┘    │
└────────────────────────────────────────────────────┘
```

### Types de réseaux Docker

| Type | Description | Usage |
|------|-------------|-------|
| `bridge` (défaut) | Réseau isolé sur l'hôte | Conteneurs sur la même machine |
| `host` | Utilise le réseau de l'hôte | Performances maximales |
| `none` | Aucun réseau | Conteneurs isolés |
| **Custom bridge** | Réseau personnalisé avec subnet | **IP fixes (notre cas)** |

---

## 📝 Étape 1 : Créer un réseau Docker personnalisé

### 1.1 Créer le réseau

Avant de lancer vos conteneurs, vous devez créer un réseau avec une plage d'adresses définie :

```bash
# Créer un réseau nommé 'mariadb_net' avec un subnet spécifique
docker network create --subnet=172.18.0.0/16 mariadb_net
```

**Détail de la commande :**
- `docker network create` : Crée un nouveau réseau
- `--subnet=172.18.0.0/16` : Définit la plage d'adresses IP disponibles
- `mariadb_net` : Nom du réseau (vous pouvez le changer)

### 1.2 Comprendre le subnet

**Format :** `172.18.0.0/16`

- `172.18.0.0` : Adresse de base du réseau
- `/16` : Masque de sous-réseau (définit la plage)

**Plage disponible :**
- Première IP utilisable : `172.18.0.1`
- Dernière IP utilisable : `172.18.255.254`
- Total : 65 534 adresses disponibles

**💡 Astuce :** Pour simplifier, utilisez des IP comme :
- `.10` pour les bases de données
- `.20` pour les services web
- `.30` pour les caches, etc.

### 1.3 Vérifier le réseau

```bash
# Lister tous les réseaux Docker
docker network ls

# Inspecter le réseau créé
docker network inspect mariadb_net
```

Vous devriez voir une sortie JSON avec les détails du réseau, dont le subnet `172.18.0.0/16`.

---

## 🐳 Étape 2 : Créer le docker-compose.yml avec IP fixe

### 2.1 Structure du projet

Créez un dossier pour votre projet :

```bash
mkdir mariadb-ip-fixe
cd mariadb-ip-fixe
mkdir data
```

### 2.2 Fichier docker-compose.yml

Créez le fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_ip_fixe
    restart: unless-stopped

    environment:
      # Mot de passe root (⚠️ CHANGEZ-LE !)
      MYSQL_ROOT_PASSWORD: mon_mot_de_passe_securise

      # Base de données initiale (optionnel)
      MYSQL_DATABASE: ma_base_dev

      # Utilisateur supplémentaire (optionnel)
      MYSQL_USER: dev_user
      MYSQL_PASSWORD: mot_de_passe_user

    ports:
      # Expose le port 3306 vers l'hôte
      - "3306:3306"

    volumes:
      # Données persistantes
      - ./data:/var/lib/mysql

    # 🔑 SECTION CLÉ : Configuration réseau avec IP fixe
    networks:
      mariadb_net:
        # Assignation de l'IP statique
        ipv4_address: 172.18.0.10

# 🔑 SECTION CLÉ : Déclaration du réseau externe
networks:
  mariadb_net:
    # Indique que ce réseau existe déjà (créé à l'étape 1)
    external: true
```

### 2.3 Points clés du fichier

#### Section networks dans le service

```yaml
networks:
  mariadb_net:
    ipv4_address: 172.18.0.10
```

- `mariadb_net` : Nom du réseau à rejoindre
- `ipv4_address` : IP fixe à assigner (doit être dans la plage du subnet)

#### Section networks globale

```yaml
networks:
  mariadb_net:
    external: true
```

- `external: true` : Indique que le réseau existe déjà (créé manuellement à l'étape 1)
- Sans cette ligne, Docker Compose essaierait de créer le réseau (et échouerait car il existe déjà)

---

## ▶️ Étape 3 : Démarrer MariaDB avec l'IP fixe

### 3.1 Lancer le conteneur

```bash
# Depuis le dossier mariadb-ip-fixe
docker-compose up -d
```

### 3.2 Vérifier l'IP assignée

```bash
# Méthode 1 : Inspection du conteneur
docker inspect mariadb_ip_fixe | grep "IPAddress"

# Méthode 2 : Inspection du réseau
docker network inspect mariadb_net
```

Vous devriez voir `"IPAddress": "172.18.0.10"` dans la sortie.

### 3.3 Vérifier les logs

```bash
docker-compose logs -f
```

Si tout fonctionne, vous verrez des messages indiquant que MariaDB a démarré correctement.

---

## 🔌 Étape 4 : Se connecter via l'IP fixe

### 4.1 Depuis votre machine hôte

Vous pouvez toujours vous connecter via `localhost` (grâce au mapping de port `3306:3306`) :

```bash
docker exec -it mariadb_ip_fixe mariadb -u root -p
```

**Paramètres pour un client graphique :**

| Paramètre | Valeur |
|-----------|--------|
| Hôte | `localhost` ou `127.0.0.1` |
| Port | `3306` |
| Utilisateur | `root` |
| Mot de passe | Celui défini dans `docker-compose.yml` |

### 4.2 Depuis un autre conteneur Docker

Pour tester la connexion via l'IP fixe depuis un autre conteneur :

```bash
# Lancer un conteneur temporaire dans le même réseau
docker run -it --rm --network mariadb_net mysql:8.0 mysql -h 172.18.0.10 -u root -p

# Entrez votre mot de passe root
# Vous devriez être connecté à MariaDB !
```

**Explication :**
- `--network mariadb_net` : Connecte ce conteneur au réseau `mariadb_net`
- `-h 172.18.0.10` : Se connecte via l'IP fixe
- `mysql:8.0` : Image contenant le client MySQL/MariaDB

---

## 🎨 Étape 5 : Configuration avancée avec my.cnf

Vous pouvez combiner l'IP fixe avec une configuration personnalisée !

### 5.1 Créer le fichier my.cnf

Dans votre dossier `mariadb-ip-fixe`, créez un fichier `my.cnf` :

```ini
[mysqld]
# Encodage
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Event Scheduler
event_scheduler = ON

# Connexions
max_connections = 200
```

### 5.2 Modifier le docker-compose.yml

Ajoutez le montage du fichier `my.cnf` dans la section `volumes` :

```yaml
volumes:
  - ./data:/var/lib/mysql
  # Ajout du fichier de configuration
  - ./my.cnf:/etc/mysql/conf.d/custom.cnf
```

### 5.3 Redémarrer

```bash
docker-compose restart
```

Votre MariaDB a maintenant :
- ✅ Une IP fixe (172.18.0.10)
- ✅ Une configuration personnalisée (my.cnf)
- ✅ Des données persistantes

---

## 🏗️ Exemple : Architecture multi-conteneurs

Créons une configuration avec plusieurs services ayant des IP fixes.

### Architecture cible

```
┌────────────────────────────────────────────┐
│  Réseau : app_network (172.20.0.0/16)      │
│                                            │
│  ┌────────────────┐    ┌────────────────┐  │
│  │  MariaDB       │    │  Application   │  │
│  │  172.20.0.10   │◄───│  172.20.0.50   │  │
│  └────────────────┘    └────────────────┘  │
│                              │             │
│                              ▼             │
│                        ┌────────────────┐  │
│                        │  Redis Cache   │  │
│                        │  172.20.0.30   │  │
│                        └────────────────┘  │
└────────────────────────────────────────────┘
```

### Étape 1 : Créer le réseau

```bash
docker network create --subnet=172.20.0.0/16 app_network
```

### Étape 2 : docker-compose.yml complet

```yaml
version: '3.8'

services:
  # Base de données MariaDB
  mariadb:
    image: mariadb:10.11
    container_name: app_mariadb
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: app_db
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
    networks:
      app_network:
        ipv4_address: 172.20.0.10

  # Cache Redis
  redis:
    image: redis:7-alpine
    container_name: app_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      app_network:
        ipv4_address: 172.20.0.30

  # Application (exemple avec Node.js)
  app:
    image: node:18-alpine
    container_name: app_backend
    restart: unless-stopped
    command: sh -c "echo 'App running' && sleep infinity"
    environment:
      # L'application peut utiliser ces IP fixes
      DB_HOST: 172.20.0.10
      DB_PORT: 3306
      REDIS_HOST: 172.20.0.30
      REDIS_PORT: 6379
    ports:
      - "3000:3000"
    networks:
      app_network:
        ipv4_address: 172.20.0.50
    depends_on:
      - mariadb
      - redis

volumes:
  mariadb_data:

networks:
  app_network:
    external: true
```

### Étape 3 : Lancer l'architecture

```bash
docker-compose up -d
```

### Étape 4 : Tester la communication

```bash
# Depuis le conteneur app, ping MariaDB
docker exec app_backend ping -c 3 172.20.0.10

# Depuis le conteneur app, ping Redis
docker exec app_backend ping -c 3 172.20.0.30
```

---

## 🛠️ Gestion du réseau et des IP

### Commandes utiles

```bash
# Lister les réseaux
docker network ls

# Inspecter un réseau (voir tous les conteneurs connectés)
docker network inspect mariadb_net

# Voir l'IP d'un conteneur spécifique
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mariadb_ip_fixe

# Déconnecter un conteneur d'un réseau
docker network disconnect mariadb_net mariadb_ip_fixe

# Reconnecter avec une nouvelle IP
docker network connect --ip 172.18.0.20 mariadb_net mariadb_ip_fixe
```

### Gérer les conflits d'IP

Si une IP est déjà utilisée, Docker affichera une erreur :

```
Error response from daemon: Address already in use
```

**Solutions :**
1. Choisir une autre IP dans la plage du subnet
2. Vérifier les conteneurs connectés : `docker network inspect mariadb_net`
3. Arrêter le conteneur qui utilise l'IP

---

## 🛑 Nettoyage complet

Pour supprimer toute la configuration :

```bash
# 1. Arrêter et supprimer les conteneurs
docker-compose down

# 2. Supprimer les données (⚠️ IRRÉVERSIBLE)
rm -rf ./data

# 3. Supprimer le réseau Docker
docker network rm mariadb_net

# 4. Supprimer les fichiers de config (optionnel)
rm docker-compose.yml my.cnf
```

---

## ✅ Récapitulatif

Vous avez appris à :

- ✅ Créer un réseau Docker personnalisé avec un subnet spécifique
- ✅ Assigner une IP fixe à un conteneur MariaDB
- ✅ Déclarer un réseau externe dans Docker Compose
- ✅ Connecter plusieurs conteneurs avec des IP fixes
- ✅ Combiner IP fixe et configuration personnalisée (my.cnf)
- ✅ Gérer et inspecter les réseaux Docker

**Avantages obtenus :**
- 🎯 IP prévisible et documentée
- 🔗 Communication facilitée entre conteneurs
- 🐛 Débogage simplifié
- 📋 Configuration d'applications facilitée

---

## 🚀 Prochaines étapes

Maintenant que vous maîtrisez les IP fixes, vous pouvez explorer :

- **[1.4 Gestion des utilisateurs](04-gestion-utilisateurs.md)** - Créer et gérer les utilisateurs SQL
- **[1.5 Accès réseau local](05-acces-reseau-local.md)** - Accéder depuis d'autres machines de votre réseau

---

## 📚 Ressources complémentaires

- [Documentation Docker - Networking](https://docs.docker.com/network/)
- [Documentation Docker - Custom bridge networks](https://docs.docker.com/network/bridge/)
- [Calculateur de subnet](https://www.subnet-calculator.com/)

---

## ❓ FAQ - Questions fréquentes

**Q : Puis-je utiliser n'importe quelle plage d'IP ?**
R : Oui, mais il est recommandé d'utiliser des plages privées :
- `172.16.0.0` à `172.31.255.255` (notre exemple : `172.18.0.0/16`)
- `192.168.0.0` à `192.168.255.255`
- `10.0.0.0` à `10.255.255.255`

**Q : L'IP fixe est-elle accessible depuis l'extérieur de Docker ?**
R : Non, l'IP fixe n'est accessible que depuis les conteneurs connectés au même réseau. Pour accéder depuis votre machine hôte, utilisez le mapping de port (`3306:3306`).

**Q : Que se passe-t-il si j'oublie de créer le réseau avant le docker-compose up ?**
R : Docker Compose affichera une erreur : `network mariadb_net declared as external, but could not be found`. Créez le réseau avec `docker network create`.

**Q : Puis-je changer l'IP d'un conteneur en cours d'exécution ?**
R : Non, vous devez arrêter le conteneur, modifier le `docker-compose.yml`, et le redémarrer. Ou utiliser `docker network disconnect` puis `docker network connect --ip`.

**Q : Combien de conteneurs puis-je avoir sur un réseau /16 ?**
R : Théoriquement 65 534, mais en pratique vous n'aurez jamais besoin de plus de quelques dizaines de conteneurs.

**Q : Quelle est la différence entre external: true et driver: bridge ?**
R :
- `external: true` : Le réseau existe déjà, Docker Compose ne doit pas le créer
- `driver: bridge` : Type de réseau à créer (si Docker Compose le crée)

**Q : Puis-je connecter un conteneur à plusieurs réseaux ?**
R : Oui ! Un conteneur peut avoir plusieurs interfaces réseau :
```yaml
networks:
  - frontend_net
  - backend_net
```

**Q : L'IP fixe fonctionne-t-elle sur Docker Swarm ou Kubernetes ?**
R : La syntaxe est différente. Docker Swarm utilise des réseaux overlay, Kubernetes utilise des Services et NetworkPolicies.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
