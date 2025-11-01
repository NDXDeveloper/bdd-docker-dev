# Annexe B - Gestion des Réseaux Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Les **réseaux Docker** permettent aux conteneurs de communiquer entre eux et avec le monde extérieur. Comprendre les réseaux est essentiel pour construire des applications multi-conteneurs (par exemple, une application web + une base de données).

**Ce que vous allez apprendre :**
- 🌐 Comprendre les types de réseaux Docker
- 🔧 Créer et gérer des réseaux personnalisés
- 📍 Assigner des adresses IP fixes aux conteneurs
- 💬 Faire communiquer plusieurs conteneurs entre eux
- 🛠️ Résoudre les problèmes réseau courants

**Niveau :** 🟡 Intermédiaire (mais expliqué pour débutants)

**Durée de lecture :** 30 minutes

---

## 📑 Table des Matières

1. [Concepts de Base](#-1-concepts-de-base)
2. [Types de Réseaux Docker](#-2-types-de-réseaux-docker)
3. [Création de Réseaux Personnalisés](#-3-création-de-réseaux-personnalisés)
4. [Attribution d'IP Fixes](#-4-attribution-dip-fixes)
5. [Communication entre Conteneurs](#-5-communication-entre-conteneurs)
6. [Cas d'Usage Pratiques](#-6-cas-dusage-pratiques)
7. [Dépannage Réseau](#-7-dépannage-réseau)

---

## 🎓 1. Concepts de Base

### 1.1 Qu'est-ce qu'un Réseau Docker ?

Un **réseau Docker** est une couche logicielle qui permet aux conteneurs de communiquer. C'est comme un réseau WiFi virtuel qui connecte vos conteneurs.

**Analogie : Le réseau WiFi de votre maison**

```
┌────────────────────────────────────────────────────────┐
│  Votre Réseau WiFi (ex: "MaBox_WiFi")                  │
│                                                        │
│  📱 Téléphone       💻 Ordinateur      📺 Smart TV      │
│  (192.168.1.10)    (192.168.1.20)    (192.168.1.30)    │
│                                                        │
│  Tous ces appareils peuvent communiquer entre eux      │
└────────────────────────────────────────────────────────┘
```

**De la même façon avec Docker :**

```
┌────────────────────────────────────────────────────────┐
│  Réseau Docker : "mon_reseau"                          │
│                                                        │
│  🐳 MariaDB        🐳 App Web       🐳 Redis            │
│  (172.20.0.10)    (172.20.0.20)    (172.20.0.30)       │
│                                                        │
│  Ces conteneurs peuvent se parler entre eux            │
└────────────────────────────────────────────────────────┘
```

---

### 1.2 Pourquoi utiliser des Réseaux ?

| Besoin | Sans réseau | Avec réseau |
|--------|-------------|-------------|
| **Isolation** | Tous les conteneurs visibles | Groupes isolés possibles |
| **Communication** | Difficile, via ports exposés | Facile, communication directe |
| **Organisation** | Désordonné | Structuré par projet |
| **Sécurité** | Moins sécurisé | Plus sécurisé (isolation) |
| **Performance** | Communication via hôte | Communication directe |

---

### 1.3 Vocabulaire Essentiel

| Terme | Définition | Analogie |
|-------|------------|----------|
| **Réseau (Network)** | Espace virtuel où les conteneurs communiquent | WiFi de votre maison |
| **Subnet** | Plage d'adresses IP disponibles | Nombre de places dans le réseau |
| **IP Address** | Adresse unique d'un conteneur | Numéro de chambre dans un hôtel |
| **Gateway** | Point d'entrée/sortie du réseau | Porte d'entrée de votre maison |
| **Bridge** | Type de réseau le plus courant | Pont connectant plusieurs appareils |
| **DNS** | Système de résolution de noms | Annuaire téléphonique |

---

## 🌐 2. Types de Réseaux Docker

Docker propose plusieurs types de réseaux selon vos besoins.

### 2.1 Réseau Bridge (par défaut)

**Description :** Réseau virtuel isolé sur votre machine. C'est le type par défaut.

**Quand l'utiliser :** Pour la plupart des applications sur une seule machine.

```
┌─────────────────────────────────────────────┐
│  Machine Hôte                               │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │  Réseau Bridge Docker (default)       │  │
│  │                                       │  │
│  │  🐳 Conteneur 1    🐳 Conteneur 2     │  │
│  │  172.17.0.2        172.17.0.3         │  │
│  │                                       │  │
│  │  Communication possible entre eux     │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Connexion Internet                         │
└─────────────────────────────────────────────┘
```

**Caractéristiques :**
- ✅ Isolé du réseau hôte
- ✅ Communication entre conteneurs du même réseau
- ✅ Accès Internet via NAT
- ⚠️ IP dynamiques (changent au redémarrage)

---

### 2.2 Réseau Host

**Description :** Le conteneur utilise directement le réseau de la machine hôte (pas d'isolation).

**Quand l'utiliser :** Performances maximales, pas besoin d'isolation.

```
┌─────────────────────────────────────────────┐
│  Machine Hôte (192.168.1.50)                │
│                                             │
│  🐳 Conteneur (partage l'IP de l'hôte)      │
│     Accessible sur 192.168.1.50             │
│                                             │
│  Pas d'isolation réseau                     │
└─────────────────────────────────────────────┘
```

**Caractéristiques :**
- ✅ Meilleures performances réseau
- ✅ Pas de mapping de ports nécessaire
- ❌ Pas d'isolation (moins sécurisé)
- ❌ Conflits de ports possibles

**Exemple :**
```yaml
services:
  app:
    image: mon_app
    network_mode: "host"
```

---

### 2.3 Réseau None

**Description :** Aucun réseau. Le conteneur est complètement isolé.

**Quand l'utiliser :** Conteneurs qui ne doivent pas communiquer (tests, sécurité maximale).

```
┌─────────────────────────────────────────────┐
│  Machine Hôte                               │
│                                             │
│  🐳 Conteneur Isolé                         │
│     Aucune connexion réseau                 │
│     Sécurité maximale                       │
└─────────────────────────────────────────────┘
```

---

### 2.4 Réseaux Personnalisés (Bridge Custom)

**Description :** Réseaux que VOUS créez avec vos propres paramètres.

**Quand l'utiliser :** Pour organiser vos projets et contrôler les IP.

```
┌─────────────────────────────────────────────────────┐
│  Machine Hôte                                       │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │  Réseau "projet_A" (172.20.0.0/16)           │   │
│  │  🐳 MariaDB    🐳 App    🐳 Redis             │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │  Réseau "projet_B" (172.21.0.0/16)           │   │
│  │  🐳 PostgreSQL    🐳 App2                    │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  Projets isolés entre eux                           │
└─────────────────────────────────────────────────────┘
```

**Avantages :**
- ✅ Isolation par projet
- ✅ Résolution DNS automatique (par nom de conteneur)
- ✅ IP fixes possibles
- ✅ Configuration personnalisée (subnet, gateway)

---

### 2.5 Tableau Comparatif

| Type | Isolation | Performances | IP Fixes | DNS | Use Case |
|------|-----------|--------------|----------|-----|----------|
| **Bridge (défaut)** | ✅ | ⭐⭐⭐ | ❌ | ⚠️ | Usage général |
| **Bridge Custom** | ✅ | ⭐⭐⭐ | ✅ | ✅ | **Recommandé pour projets** |
| **Host** | ❌ | ⭐⭐⭐⭐⭐ | ❌ | N/A | Performances critiques |
| **None** | ✅✅ | N/A | N/A | N/A | Sécurité maximale |

**🎯 Recommandation :** Utilisez les **réseaux Bridge personnalisés** pour vos projets !

---

## 🔧 3. Création de Réseaux Personnalisés

### 3.1 Commande de Base

```bash
docker network create <nom_du_reseau>
```

**Exemple simple :**
```bash
# Créer un réseau nommé "mon_projet"
docker network create mon_projet

# Vérifier qu'il est créé
docker network ls
```

**Résultat attendu :**
```
NETWORK ID     NAME         DRIVER    SCOPE
abc123def456   mon_projet   bridge    local
```

---

### 3.2 Création avec Subnet (pour IP fixes)

Pour pouvoir assigner des IP fixes, vous devez définir une **plage d'adresses (subnet)**.

**Syntaxe :**
```bash
docker network create --subnet=<plage_IP> <nom_reseau>
```

**Exemple :**
```bash
# Créer un réseau avec la plage 172.20.0.0/16
docker network create --subnet=172.20.0.0/16 mon_reseau_fixe
```

**Comprendre la notation `/16` :**

| Notation | Plage d'IP disponibles | Nombre d'adresses | Usage |
|----------|------------------------|-------------------|-------|
| `/24` | 172.20.0.1 à 172.20.0.254 | 254 | Petits projets (quelques conteneurs) |
| `/16` | 172.20.0.1 à 172.20.255.254 | 65 534 | **Recommandé** (large marge) |
| `/8` | 172.0.0.1 à 172.255.255.254 | 16 millions | Très grands déploiements |

**💡 Conseil :** Utilisez `/16` pour avoir de la marge sans compliquer.

---

### 3.3 Options Avancées

```bash
docker network create \
  --driver bridge \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  --ip-range=172.20.10.0/24 \
  mon_reseau_avance
```

**Explication des options :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `--driver` | Type de réseau | `bridge`, `overlay` |
| `--subnet` | Plage complète d'IP | `172.20.0.0/16` |
| `--gateway` | IP de la passerelle | `172.20.0.1` |
| `--ip-range` | Sous-plage pour attribution auto | `172.20.10.0/24` |
| `--label` | Métadonnées | `projet=dev` |

---

### 3.4 Avec Docker Compose

**Méthode 1 : Réseau créé par Compose**

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    networks:
      - backend

  app:
    image: mon_app
    networks:
      - backend

# Compose créera automatiquement le réseau "backend"
networks:
  backend:
    driver: bridge
```

---

**Méthode 2 : Réseau avec subnet personnalisé**

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    networks:
      backend:
        ipv4_address: 172.20.0.10

  app:
    image: mon_app
    networks:
      backend:
        ipv4_address: 172.20.0.20

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

---

**Méthode 3 : Utiliser un réseau externe (créé manuellement)**

```bash
# 1. Créer le réseau manuellement
docker network create --subnet=172.20.0.0/16 mon_reseau_externe
```

```yaml
# 2. L'utiliser dans docker-compose.yml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    networks:
      mon_reseau_externe:
        ipv4_address: 172.20.0.10

networks:
  mon_reseau_externe:
    external: true  # Indique que le réseau existe déjà
```

---

### 3.5 Lister et Inspecter les Réseaux

```bash
# Lister tous les réseaux
docker network ls

# Détails d'un réseau spécifique
docker network inspect mon_projet

# Voir quels conteneurs sont connectés
docker network inspect mon_projet --format '{{range .Containers}}{{.Name}} {{.IPv4Address}}{{"\n"}}{{end}}'
```

---

## 📍 4. Attribution d'IP Fixes

### 4.1 Pourquoi des IP Fixes ?

**Sans IP fixe (comportement par défaut) :**
- 🔄 L'IP change à chaque redémarrage du conteneur
- ❓ Difficile de prévoir quelle IP aura le conteneur
- 🔧 Configuration d'applications complexe

**Avec IP fixe :**
- ✅ IP prévisible et stable
- ✅ Documentation claire ("MariaDB est toujours sur 172.20.0.10")
- ✅ Configuration d'applications facilitée
- ✅ Débogage simplifié

---

### 4.2 Prérequis pour les IP Fixes

Pour assigner des IP fixes, il faut :

1. ✅ Un réseau avec un **subnet défini**
2. ✅ Une IP dans la **plage du subnet**
3. ✅ Une IP **unique** (pas déjà utilisée)

---

### 4.3 Méthode 1 : Avec `docker run`

```bash
# 1. Créer le réseau avec subnet
docker network create --subnet=172.20.0.0/16 mon_reseau

# 2. Lancer un conteneur avec IP fixe
docker run -d \
  --name mariadb_fixe \
  --network mon_reseau \
  --ip 172.20.0.10 \
  -e MYSQL_ROOT_PASSWORD=secret \
  mariadb:10.11

# 3. Vérifier l'IP
docker inspect mariadb_fixe | grep "IPAddress"
```

**Résultat attendu :**
```json
"IPAddress": "172.20.0.10"
```

---

### 4.4 Méthode 2 : Avec Docker Compose (Recommandé)

**Étape 1 : Créer le réseau manuellement**

```bash
docker network create --subnet=172.20.0.0/16 app_network
```

**Étape 2 : Configurer docker-compose.yml**

```yaml
version: '3.8'

services:
  # Base de données MariaDB
  mariadb:
    image: mariadb:10.11
    container_name: app_mariadb
    environment:
      MYSQL_ROOT_PASSWORD: root_password
    networks:
      app_network:
        ipv4_address: 172.20.0.10  # IP FIXE

  # Cache Redis
  redis:
    image: redis:7-alpine
    container_name: app_redis
    networks:
      app_network:
        ipv4_address: 172.20.0.30  # IP FIXE

  # Application
  app:
    image: node:18-alpine
    container_name: app_backend
    environment:
      DB_HOST: 172.20.0.10  # Utilise l'IP fixe de MariaDB
      REDIS_HOST: 172.20.0.30  # Utilise l'IP fixe de Redis
    networks:
      app_network:
        ipv4_address: 172.20.0.50  # IP FIXE

# Déclarer le réseau comme externe
networks:
  app_network:
    external: true
```

**Étape 3 : Lancer**

```bash
docker-compose up -d
```

**Étape 4 : Vérifier les IP**

```bash
docker network inspect app_network
```

---

### 4.5 Bonnes Pratiques pour les IP Fixes

**Organisation des IP par type de service :**

```yaml
# Convention recommandée
172.20.0.10 - 172.20.0.19  : Bases de données SQL
172.20.0.20 - 172.20.0.29  : Bases NoSQL
172.20.0.30 - 172.20.0.39  : Caches (Redis, Memcached)
172.20.0.40 - 172.20.0.49  : Services de messages (RabbitMQ, Kafka)
172.20.0.50 - 172.20.0.99  : Applications / APIs
172.20.0.100+              : Services auxiliaires
```

**Exemple concret :**

```yaml
services:
  mariadb:
    networks:
      backend:
        ipv4_address: 172.20.0.10  # BDD SQL

  mongodb:
    networks:
      backend:
        ipv4_address: 172.20.0.20  # BDD NoSQL

  redis:
    networks:
      backend:
        ipv4_address: 172.20.0.30  # Cache

  api:
    networks:
      backend:
        ipv4_address: 172.20.0.50  # Application
```

**Avantages :**
- 📋 Documentation claire
- 🔍 Identification rapide
- 🛠️ Dépannage facilité

---

### 4.6 Erreurs Courantes

#### Erreur 1 : IP déjà utilisée

```
Error response from daemon: Address already in use
```

**Cause :** Un autre conteneur utilise déjà cette IP.

**Solution :**
```bash
# Voir les conteneurs sur le réseau
docker network inspect mon_reseau

# Choisir une IP libre
```

---

#### Erreur 2 : IP hors du subnet

```
Error response from daemon: Invalid address 192.168.1.10
```

**Cause :** L'IP n'est pas dans la plage définie lors de la création du réseau.

**Solution :**
```bash
# Vérifier le subnet du réseau
docker network inspect mon_reseau | grep Subnet

# Utiliser une IP dans cette plage
```

---

#### Erreur 3 : Réseau non trouvé

```
network mon_reseau declared as external, but could not be found
```

**Cause :** Le réseau n'a pas été créé avant le `docker-compose up`.

**Solution :**
```bash
# Créer le réseau d'abord
docker network create --subnet=172.20.0.0/16 mon_reseau

# Puis lancer
docker-compose up -d
```

---

## 💬 5. Communication entre Conteneurs

### 5.1 Résolution DNS Automatique

Sur un réseau Docker personnalisé, les conteneurs peuvent se contacter par **leur nom**.

**Exemple :**

```yaml
version: '3.8'

services:
  # Base de données
  mariadb:
    image: mariadb:10.11
    container_name: db_server
    environment:
      MYSQL_ROOT_PASSWORD: secret
    networks:
      - backend

  # Application
  app:
    image: mon_app
    environment:
      # 🔑 Utiliser le NOM du service, pas l'IP !
      DB_HOST: mariadb  # ← Nom du service
      DB_PORT: 3306
    networks:
      - backend

networks:
  backend:
```

**Comment ça marche :**

```
┌─────────────────────────────────────────────┐
│  Réseau Docker "backend"                    │
│                                             │
│  🐳 mariadb (172.20.0.10)                   │
│      ↑                                      │
│      │ DNS: "mariadb" → 172.20.0.10         │
│      │                                      │
│  🐳 app (172.20.0.20)                       │
│      Peut se connecter via "mariadb:3306"   │
└─────────────────────────────────────────────┘
```

---

### 5.2 Communication par Nom vs par IP

**Méthode 1 : Par nom (RECOMMANDÉ)**

```yaml
environment:
  DB_HOST: mariadb  # Nom du service
```

**Avantages :**
- ✅ Flexible (IP peut changer)
- ✅ Lisible
- ✅ DNS géré automatiquement par Docker

---

**Méthode 2 : Par IP fixe**

```yaml
environment:
  DB_HOST: 172.20.0.10  # IP fixe
```

**Avantages :**
- ✅ Prévisible
- ✅ Utile pour debug
- ⚠️ Moins flexible (IP codée en dur)

---

### 5.3 Tester la Communication

#### Test 1 : Ping entre conteneurs

```bash
# Depuis le conteneur "app", ping le conteneur "mariadb"
docker exec app ping -c 3 mariadb

# Résultat attendu :
# PING mariadb (172.20.0.10): 56 data bytes
# 64 bytes from 172.20.0.10: seq=0 ttl=64 time=0.123 ms
```

---

#### Test 2 : Vérifier la résolution DNS

```bash
# Résoudre le nom en IP
docker exec app nslookup mariadb

# Résultat :
# Server:    127.0.0.11
# Address:   127.0.0.11:53
#
# Name:      mariadb
# Address:   172.20.0.10
```

---

#### Test 3 : Test de connexion à un port

```bash
# Tester la connexion au port 3306 de MariaDB
docker exec app nc -zv mariadb 3306

# Résultat :
# mariadb (172.20.0.10:3306) open
```

---

### 5.4 Communication Multi-Réseaux

Un conteneur peut être connecté à **plusieurs réseaux** simultanément.

**Exemple : Frontend + Backend**

```yaml
version: '3.8'

services:
  # Base de données (réseau backend uniquement)
  mariadb:
    image: mariadb:10.11
    networks:
      - backend

  # API (connectée aux 2 réseaux)
  api:
    image: mon_api
    networks:
      - frontend  # Accessible par le web
      - backend   # Accède à la BDD

  # Nginx (réseau frontend uniquement)
  nginx:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"

networks:
  frontend:
  backend:
```

**Architecture :**

```
                     Internet
                        ↓
                   ┌─────────┐
                   │  Nginx  │
                   │ (front) │
                   └─────────┘
                        ↓
           Réseau Frontend (172.21.0.0/16)
                        ↓
                   ┌─────────┐
                   │   API   │ ← Pont entre les 2 réseaux
                   │(front+  │
                   │backend) │
                   └─────────┘
                        ↓
           Réseau Backend (172.20.0.0/16)
                        ↓
                   ┌─────────┐
                   │ MariaDB │
                   │(backend)│
                   └─────────┘
```

**Avantages :**
- ✅ **Sécurité** : La BDD n'est pas accessible depuis le frontend
- ✅ **Isolation** : Séparation claire des responsabilités
- ✅ **Flexibilité** : Chaque service sur les réseaux nécessaires uniquement

---

### 5.5 Alias de Réseau

Vous pouvez donner plusieurs noms à un conteneur sur un réseau.

```yaml
services:
  mariadb:
    image: mariadb:10.11
    networks:
      backend:
        aliases:
          - db          # Alias 1
          - database    # Alias 2
          - mysql       # Alias 3
```

**Utilisation :**
```bash
# Tous ces noms fonctionnent :
docker exec app ping db
docker exec app ping database
docker exec app ping mysql
```

---

## 🎯 6. Cas d'Usage Pratiques

### 6.1 Architecture LAMP (Linux, Apache, MySQL, PHP)

```yaml
version: '3.8'

services:
  # Base de données
  mysql:
    image: mariadb:10.11
    container_name: lamp_mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_pass
      MYSQL_DATABASE: app_db
    networks:
      lamp_net:
        ipv4_address: 172.25.0.10

  # Serveur web Apache + PHP
  apache:
    image: php:8.1-apache
    container_name: lamp_apache
    volumes:
      - ./www:/var/www/html
    ports:
      - "80:80"
    environment:
      DB_HOST: mysql  # Utilise le nom du service
      DB_NAME: app_db
    networks:
      lamp_net:
        ipv4_address: 172.25.0.20
    depends_on:
      - mysql

networks:
  lamp_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16
```

---

### 6.2 Micro-services (API + BDD + Cache)

```yaml
version: '3.8'

services:
  # Base de données PostgreSQL
  postgres:
    image: postgres:15
    networks:
      backend:
        ipv4_address: 172.30.0.10
    environment:
      POSTGRES_PASSWORD: pg_pass

  # Cache Redis
  redis:
    image: redis:7-alpine
    networks:
      backend:
        ipv4_address: 172.30.0.20

  # API Node.js
  api:
    image: node:18
    networks:
      backend:
        ipv4_address: 172.30.0.50
      frontend:
    environment:
      DATABASE_URL: postgresql://postgres:pg_pass@postgres:5432/mydb
      REDIS_URL: redis://redis:6379

  # Proxy Nginx
  nginx:
    image: nginx
    networks:
      frontend:
    ports:
      - "80:80"
    depends_on:
      - api

networks:
  backend:
    ipam:
      config:
        - subnet: 172.30.0.0/16
  frontend:
```

---

### 6.3 Environnement de Test Isolé

```yaml
version: '3.8'

# Projet A - Environnement de test
services:
  test_db:
    image: mariadb:10.11
    networks:
      test_net:
        ipv4_address: 172.40.0.10

  test_app:
    image: mon_app:test
    networks:
      test_net:
        ipv4_address: 172.40.0.20

networks:
  test_net:
    ipam:
      config:
        - subnet: 172.40.0.0/16
```

**Avantages :**
- ✅ Isolé des autres projets
- ✅ Peut avoir les mêmes noms de services
- ✅ Nettoyage facile

---

### 6.4 Multi-projets sur une même Machine

```bash
# Projet 1 : Blog
docker network create --subnet=172.50.0.0/16 blog_net

# Projet 2 : E-commerce
docker network create --subnet=172.51.0.0/16 shop_net

# Projet 3 : API interne
docker network create --subnet=172.52.0.0/16 api_net
```

**Isolation complète :**
```
┌────────────────────────────────────────────┐
│  Machine Hôte                              │
│                                            │
│  ┌──────────────────┐  ┌──────────────┐    │
│  │ Blog (172.50.x)  │  │ Shop (172.51)│    │
│  │ MariaDB + WP     │  │ Postgres+App │    │
│  └──────────────────┘  └──────────────┘    │
│                                            │
│         ┌────────────────────┐             │
│         │ API (172.52.x)     │             │
│         │ Redis + Node       │             │
│         └────────────────────┘             │
└────────────────────────────────────────────┘
```

---

## 🐛 7. Dépannage Réseau

### 7.1 Problèmes Courants

#### Problème 1 : Conteneurs ne peuvent pas communiquer

**Symptôme :**
```bash
docker exec app ping mariadb
# ping: mariadb: Name or service not known
```

**Causes possibles :**

1. **Pas sur le même réseau**

```bash
# Vérifier les réseaux de chaque conteneur
docker inspect app | grep -A 10 "Networks"
docker inspect mariadb | grep -A 10 "Networks"
```

**Solution :** Connecter au même réseau
```bash
docker network connect mon_reseau app
```

---

2. **Utilisation du réseau par défaut**

Le réseau `bridge` par défaut ne supporte pas la résolution DNS par nom.

**Solution :** Créer un réseau personnalisé
```bash
docker network create mon_reseau_custom
docker network connect mon_reseau_custom app
docker network connect mon_reseau_custom mariadb
```

---

#### Problème 2 : "Address already in use"

**Symptôme :**
```
Error: Address 172.20.0.10 already in use
```

**Solution :**
```bash
# 1. Voir qui utilise cette IP
docker network inspect mon_reseau

# 2. Arrêter le conteneur qui l'utilise
docker stop <conteneur_problematique>

# 3. OU choisir une autre IP
# Dans docker-compose.yml : ipv4_address: 172.20.0.11
```

---

#### Problème 3 : Impossible de créer le réseau

**Symptôme :**
```
Error: Pool overlaps with other one on this address space
```

**Cause :** La plage d'IP chevauche un réseau existant.

**Solution :**
```bash
# Lister les réseaux existants
docker network ls

# Voir leurs subnets
docker network inspect <nom_reseau> | grep Subnet

# Choisir une plage différente
docker network create --subnet=172.99.0.0/16 mon_nouveau_reseau
```

---

#### Problème 4 : Connexion refusée sur un port

**Symptôme :**
```bash
docker exec app telnet mariadb 3306
# Connection refused
```

**Vérifications :**

1. **Le service écoute-t-il sur ce port ?**
```bash
docker exec mariadb netstat -tuln | grep 3306
```

2. **Le port est-il bien exposé dans l'image ?**
```bash
docker inspect mariadb | grep ExposedPorts
```

3. **Le service a-t-il démarré ?**
```bash
docker logs mariadb | tail -20
```

---

### 7.2 Outils de Diagnostic

#### Installer des outils de debug dans un conteneur

```bash
# Exemple avec Alpine
docker exec -it mon_conteneur sh

# Installer des outils
apk add curl wget netcat-openbsd bind-tools

# Tester
ping autre_conteneur
nslookup autre_conteneur
nc -zv autre_conteneur 3306
```

---

#### Utiliser un conteneur de debug

```bash
# Lancer un conteneur temporaire avec outils réseau
docker run -it --rm --network mon_reseau nicolaka/netshoot

# Depuis ce conteneur, tester
ping mariadb
nmap mariadb
curl http://api:8080/health
```

---

### 7.3 Commandes de Diagnostic Réseau

```bash
# Voir tous les réseaux
docker network ls

# Détails d'un réseau (conteneurs connectés, IPs)
docker network inspect mon_reseau

# IP d'un conteneur
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mariadb

# Ports ouverts sur un conteneur
docker port mariadb

# Logs du conteneur
docker logs -f mariadb

# Processus réseau dans le conteneur
docker exec mariadb netstat -tuln

# Test de connexion depuis l'hôte
telnet localhost 3306
```

---

### 7.4 Checklist de Dépannage

Quand quelque chose ne fonctionne pas :

- [ ] Les conteneurs sont-ils démarrés ? (`docker ps`)
- [ ] Sont-ils sur le même réseau ? (`docker network inspect`)
- [ ] Le service écoute-t-il sur le bon port ? (`docker logs`)
- [ ] La résolution DNS fonctionne-t-elle ? (`nslookup`)
- [ ] Le port est-il accessible ? (`telnet` ou `nc`)
- [ ] Les variables d'environnement sont-elles correctes ? (`docker exec env`)
- [ ] Y a-t-il des erreurs dans les logs ? (`docker logs`)

---

## 📊 Tableaux de Référence

### Commandes Réseau Essentielles

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker network ls` | Lister les réseaux | `docker network ls` |
| `docker network create` | Créer un réseau | `docker network create mon_net` |
| `docker network rm` | Supprimer un réseau | `docker network rm mon_net` |
| `docker network inspect` | Détails d'un réseau | `docker network inspect mon_net` |
| `docker network connect` | Connecter un conteneur | `docker network connect mon_net app` |
| `docker network disconnect` | Déconnecter | `docker network disconnect mon_net app` |
| `docker network prune` | Supprimer réseaux inutilisés | `docker network prune` |

---

### Plages d'IP Recommandées

| Plage | Usage | Exemple |
|-------|-------|---------|
| `172.16.0.0/16` à `172.31.0.0/16` | Réseaux Docker | `172.20.0.0/16` |
| `192.168.0.0/16` | Réseaux locaux | `192.168.10.0/24` |
| `10.0.0.0/8` | Grandes infrastructures | `10.0.1.0/24` |

**💡 Conseil :** Pour Docker, préférez `172.x.0.0/16` pour éviter les conflits avec votre réseau local.

---

## 🎓 Résumé des Concepts Clés

### Ce qu'il faut retenir

| Concept | Points Clés |
|---------|-------------|
| **Réseau Bridge** | Type par défaut, isolé, idéal pour la plupart des usages |
| **Réseau Personnalisé** | Permet DNS par nom + IP fixes |
| **Subnet** | Plage d'IP (ex: 172.20.0.0/16 = 65k adresses) |
| **IP Fixe** | Nécessite un réseau avec subnet défini |
| **Communication** | Par nom (DNS) ou par IP |
| **Isolation** | Un conteneur peut être sur plusieurs réseaux |

---

### Workflow Recommandé

```
1. Créer un réseau personnalisé avec subnet
   ↓
2. Assigner des IP fixes selon une convention
   ↓
3. Faire communiquer par NOMS (DNS)
   ↓
4. Organiser en multi-réseaux si sécurité nécessaire
```

---

## 🚀 Pour Aller Plus Loin

### Documentation Officielle

- 📖 [Docker Networking Overview](https://docs.docker.com/network/)
- 📖 [Docker Network Drivers](https://docs.docker.com/network/drivers/)
- 📖 [Compose Networking](https://docs.docker.com/compose/networking/)

### Annexes Connexes

- **[Annexe A - Commandes](A-reference-commandes.md)** - Toutes les commandes réseau
- **[Annexe E - Dépannage](E-depannage.md)** - Résoudre les problèmes
- **[Cas Pratique 04](../cas-pratiques/04-env-dev-complet.md)** - Multi-BDD avec réseaux

### Sujets Avancés (hors scope débutant)

- Réseaux Overlay (Docker Swarm)
- Réseaux Macvlan (adresses IP du réseau physique)
- IPv6 dans Docker
- Network policies et sécurité avancée

---

## 💡 Conseils Finaux

### Bonnes Pratiques

✅ **Toujours utiliser des réseaux personnalisés** (pas le bridge par défaut)
✅ **Organiser les IP par type de service** (BDD = .10-.19, Apps = .50+)
✅ **Utiliser la résolution DNS par nom** plutôt que par IP
✅ **Documenter votre schéma réseau** dans le README du projet
✅ **Tester la communication** entre conteneurs après déploiement

❌ **Ne pas utiliser le réseau par défaut** pour des projets multi-conteneurs
❌ **Ne pas mélanger plusieurs projets** sur le même réseau
❌ **Ne pas oublier de créer le réseau** avant `docker-compose up`

---

## 📝 Template de Configuration

Voici un template prêt à l'emploi pour vos projets :

```yaml
version: '3.8'

services:
  # Base de données
  database:
    image: mariadb:10.11
    container_name: ${PROJECT_NAME}_db
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    networks:
      backend:
        ipv4_address: 172.20.0.10
    volumes:
      - db_data:/var/lib/mysql

  # Application
  app:
    image: mon_app
    container_name: ${PROJECT_NAME}_app
    environment:
      DB_HOST: database  # Résolution DNS
      DB_PORT: 3306
    networks:
      backend:
        ipv4_address: 172.20.0.50
      frontend:
    depends_on:
      - database

  # Proxy
  nginx:
    image: nginx
    container_name: ${PROJECT_NAME}_proxy
    networks:
      frontend:
    ports:
      - "80:80"

volumes:
  db_data:

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  frontend:
    driver: bridge
```

**Fichier `.env` associé :**
```bash
PROJECT_NAME=mon_projet
DB_ROOT_PASSWORD=changez_moi
```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

