# Environnement de Développement Complet Multi-BDD avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous guide dans la création d'un **environnement de développement complet** intégrant plusieurs types de bases de données. C'est l'outil idéal pour les développeurs qui travaillent sur des projets complexes nécessitant différentes technologies de stockage de données.

**Ce que vous allez apprendre :**
- Comprendre pourquoi utiliser plusieurs bases de données
- Déployer un environnement multi-BDD avec Docker Compose
- Configurer MariaDB, PostgreSQL, MongoDB et Redis ensemble
- Accéder et utiliser chaque base de données
- Gérer un environnement de développement complet
- Optimiser les ressources et les performances

**Durée estimée :** 45-60 minutes

---

## 🎯 Pourquoi un environnement multi-BDD ?

### Le concept de Polyglot Persistence

**Polyglot Persistence** (persistance polyglotte) est une approche architecturale qui consiste à utiliser différents types de bases de données au sein d'une même application, en choisissant la technologie la plus adaptée à chaque besoin spécifique.

### Cas d'usage typiques

| Scénario | Problème | Solution multi-BDD |
|----------|----------|-------------------|
| **E-commerce** | Catalogue produits + panier + cache + sessions | PostgreSQL (produits) + Redis (cache/sessions) + MongoDB (logs) |
| **Application web** | Données relationnelles + recherche full-text + temps réel | MariaDB (users/orders) + Elasticsearch (search) + Redis (pub/sub) |
| **SaaS complexe** | Données structurées + analytics + cache | PostgreSQL (app data) + MongoDB (analytics) + Redis (cache) |
| **Plateforme sociale** | Profils + posts + relations + recommandations | PostgreSQL (profils) + Neo4j (graphe social) + Redis (recommandations) |

### Avantages d'un environnement multi-BDD de développement

| Avantage | Explication |
|----------|-------------|
| **Flexibilité** | Tester différentes technologies sans installation complexe |
| **Réalisme** | Reproduire l'environnement de production |
| **Apprentissage** | Comparer et apprendre plusieurs BDD |
| **Isolation** | Chaque BDD dans son conteneur, pas d'interférence |
| **Reproductibilité** | Même environnement pour toute l'équipe |
| **Portabilité** | Fonctionne sur Windows, macOS, Linux |

---

## 🏗️ Architecture de l'environnement

### Vue d'ensemble

Nous allons créer un environnement comprenant :

1. **MariaDB** : Base de données relationnelle (SQL) - Compatible MySQL
2. **PostgreSQL** : Base de données relationnelle avancée (SQL)
3. **MongoDB** : Base de données NoSQL orientée documents (JSON)
4. **Redis** : Cache en mémoire et base clé-valeur
5. **Interfaces graphiques** : pgAdmin, Mongo Express, Redis Commander

### Schéma de l'architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    VOTRE APPLICATION                        │
│              (Node.js, Python, PHP, Java...)                │
└──────────┬──────────┬──────────┬──────────┬─────────────────┘
           │          │          │          │
           │          │          │          │
     ┌─────▼───┐ ┌────▼─────┐ ┌──▼──────┐ ┌─▼───────┐
     │ MariaDB │ │PostgreSQL│ │ MongoDB │ │  Redis  │
     │:3306    │ │:5432     │ │:27017   │ │:6379    │
     └─────────┘ └──────────┘ └─────────┘ └─────────┘
           │          │          │          │
     ┌─────▼────┐ ┌────▼────┐ ┌──▼──────┐ ┌─▼───────┐
     │phpMyAdmin│ │ pgAdmin │ │  Mongo  │ │  Redis  │
     │  :8080   │ │  :5050  │ │ Express │ │Commander│
     └──────────┘ └─────────┘ │  :8081  │ │  :8082  │
                              └─────────┘ └─────────┘

     Tous connectés au réseau Docker : dev_network
```

### Répartition des rôles

| Base de données | Type | Usage typique | Port |
|----------------|------|---------------|------|
| **MariaDB** | SQL relationnel | Données structurées, transactions | 3306 |
| **PostgreSQL** | SQL relationnel avancé | Données complexes, JSON, GIS | 5432 |
| **MongoDB** | NoSQL document | Données flexibles, JSON natif | 27017 |
| **Redis** | Cache / Key-Value | Cache, sessions, pub/sub | 6379 |

---

## 🔧 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ Docker installé (version 20.10+)
- ✅ Docker Compose installé (version 2.0+)
- ✅ **Au moins 6 GB de RAM disponible** pour Docker (8 GB recommandé)
- ✅ **15 GB d'espace disque libre**
- ✅ Un éditeur de texte (VS Code recommandé)

**Vérification rapide :**

```bash
docker --version
docker-compose --version

# Vérifier la mémoire allouée à Docker
docker info | grep Memory
```

**⚠️ Important :** Cet environnement nécessite plus de ressources qu'une stack simple.

**Sur Windows/macOS (Docker Desktop) :**
1. Ouvrir Docker Desktop → Settings → Resources
2. Augmenter la RAM à au moins 6 GB (idéalement 8 GB)
3. Cliquer sur "Apply & Restart"

---

## 📁 Étape 1 : Structure du projet

### 1.1 Créer l'arborescence

```bash
# Créer le dossier principal
mkdir dev-multi-db
cd dev-multi-db

# Créer les sous-dossiers
mkdir -p mariadb/{data,init,config}
mkdir -p postgres/{data,init}
mkdir -p mongodb/{data,init}
mkdir -p redis/{data,config}
mkdir -p pgadmin/data
```

**Votre structure sera :**

```
dev-multi-db/
├── docker-compose.yml          # Configuration de tous les services
├── .env                        # Variables d'environnement
├── README.md                   # Documentation du projet
│
├── mariadb/
│   ├── data/                   # Données MariaDB (généré automatiquement)
│   ├── init/                   # Scripts SQL d'initialisation
│   │   └── 01-init.sql
│   └── config/
│       └── my.cnf              # Configuration personnalisée
│
├── postgres/
│   ├── data/                   # Données PostgreSQL
│   └── init/                   # Scripts SQL d'initialisation
│       └── 01-init.sql
│
├── mongodb/
│   ├── data/                   # Données MongoDB
│   └── init/                   # Scripts JS d'initialisation
│       └── 01-init.js
│
├── redis/
│   ├── data/                   # Données Redis (snapshots)
│   └── config/
│       └── redis.conf          # Configuration Redis
│
└── pgadmin/
    └── data/                   # Configuration pgAdmin
```

---

## 🗄️ Étape 2 : Configuration des variables d'environnement

### 2.1 Créer le fichier .env

Créez le fichier `.env` à la racine du projet :

```bash
# ==========================================
# ENVIRONNEMENT DE DÉVELOPPEMENT MULTI-BDD
# ==========================================

# ==========================================
# MariaDB
# ==========================================
MARIADB_ROOT_PASSWORD=mariadb_root_pass_2024
MARIADB_DATABASE=dev_db
MARIADB_USER=dev_user
MARIADB_PASSWORD=dev_pass_2024

# ==========================================
# PostgreSQL
# ==========================================
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres_pass_2024
POSTGRES_DB=dev_db

# ==========================================
# MongoDB
# ==========================================
MONGO_INITDB_ROOT_USERNAME=mongo_admin
MONGO_INITDB_ROOT_PASSWORD=mongo_pass_2024
MONGO_INITDB_DATABASE=dev_db

# ==========================================
# Redis
# ==========================================
REDIS_PASSWORD=redis_pass_2024

# ==========================================
# pgAdmin
# ==========================================
PGADMIN_DEFAULT_EMAIL=admin@dev.local
PGADMIN_DEFAULT_PASSWORD=pgadmin_pass_2024

# ==========================================
# Réseau
# ==========================================
NETWORK_SUBNET=172.25.0.0/16
```

### 2.2 Créer le fichier .env.example

Pour partager avec l'équipe (sans les vrais mots de passe) :

```bash
# Copier .env en .env.example
cp .env .env.example

# Remplacer les mots de passe par des placeholders
sed -i 's/=.*_pass_.*/=CHANGEME/g' .env.example
sed -i 's/=.*_admin/=admin@example.com/g' .env.example
```

---

## 🐳 Étape 3 : Configuration Docker Compose

### 3.1 Créer le fichier docker-compose.yml

Créez le fichier `docker-compose.yml` avec tous les services :

```yaml
version: '3.8'

services:
  # ==========================================
  # SERVICE 1 : MARIADB
  # ==========================================
  mariadb:
    image: mariadb:10.11
    container_name: dev_mariadb
    restart: unless-stopped

    environment:
      MYSQL_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MARIADB_DATABASE}
      MYSQL_USER: ${MARIADB_USER}
      MYSQL_PASSWORD: ${MARIADB_PASSWORD}

    ports:
      - "3306:3306"

    volumes:
      - ./mariadb/data:/var/lib/mysql
      - ./mariadb/init:/docker-entrypoint-initdb.d:ro
      - ./mariadb/config/my.cnf:/etc/mysql/conf.d/custom.cnf:ro

    networks:
      dev_network:
        ipv4_address: 172.25.0.10

    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 2 : POSTGRESQL
  # ==========================================
  postgres:
    image: postgres:15
    container_name: dev_postgres
    restart: unless-stopped

    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

    ports:
      - "5432:5432"

    volumes:
      - ./postgres/data:/var/lib/postgresql/data
      - ./postgres/init:/docker-entrypoint-initdb.d:ro

    networks:
      dev_network:
        ipv4_address: 172.25.0.11

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 3 : MONGODB
  # ==========================================
  mongodb:
    image: mongo:7.0
    container_name: dev_mongodb
    restart: unless-stopped

    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}

    ports:
      - "27017:27017"

    volumes:
      - ./mongodb/data:/data/db
      - ./mongodb/init:/docker-entrypoint-initdb.d:ro

    networks:
      dev_network:
        ipv4_address: 172.25.0.12

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 4 : REDIS
  # ==========================================
  redis:
    image: redis:7-alpine
    container_name: dev_redis
    restart: unless-stopped

    command: redis-server /usr/local/etc/redis/redis.conf --requirepass ${REDIS_PASSWORD}

    ports:
      - "6379:6379"

    volumes:
      - ./redis/data:/data
      - ./redis/config/redis.conf:/usr/local/etc/redis/redis.conf:ro

    networks:
      dev_network:
        ipv4_address: 172.25.0.13

    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 5 : PHPMYADMIN (Interface MariaDB)
  # ==========================================
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: dev_phpmyadmin
    restart: unless-stopped

    depends_on:
      - mariadb

    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: ${MARIADB_ROOT_PASSWORD}
      UPLOAD_LIMIT: 50M

    ports:
      - "8080:80"

    networks:
      - dev_network

  # ==========================================
  # SERVICE 6 : PGADMIN (Interface PostgreSQL)
  # ==========================================
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: dev_pgadmin
    restart: unless-stopped

    depends_on:
      - postgres

    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}
      PGADMIN_CONFIG_SERVER_MODE: 'False'
      PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED: 'False'

    ports:
      - "5050:80"

    volumes:
      - ./pgadmin/data:/var/lib/pgadmin

    networks:
      - dev_network

  # ==========================================
  # SERVICE 7 : MONGO EXPRESS (Interface MongoDB)
  # ==========================================
  mongo-express:
    image: mongo-express:latest
    container_name: dev_mongo_express
    restart: unless-stopped

    depends_on:
      - mongodb

    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_INITDB_ROOT_USERNAME}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
      ME_CONFIG_MONGODB_URL: mongodb://${MONGO_INITDB_ROOT_USERNAME}:${MONGO_INITDB_ROOT_PASSWORD}@mongodb:27017/
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}

    ports:
      - "8081:8081"

    networks:
      - dev_network

  # ==========================================
  # SERVICE 8 : REDIS COMMANDER (Interface Redis)
  # ==========================================
  redis-commander:
    image: rediscommander/redis-commander:latest
    container_name: dev_redis_commander
    restart: unless-stopped

    depends_on:
      - redis

    environment:
      REDIS_HOSTS: local:redis:6379:0:${REDIS_PASSWORD}

    ports:
      - "8082:8081"

    networks:
      - dev_network

# ==========================================
# RÉSEAU PERSONNALISÉ
# ==========================================
networks:
  dev_network:
    driver: bridge
    ipam:
      config:
        - subnet: ${NETWORK_SUBNET}
```

---

## 📝 Étape 4 : Scripts d'initialisation

### 4.1 Script MariaDB

Créez le fichier `mariadb/init/01-init.sql` :

```sql
-- ==========================================
-- Script d'initialisation MariaDB
-- ==========================================

USE dev_db;

-- Table utilisateurs
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Table produits
CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    category VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_category (category),
    INDEX idx_price (price)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Table commandes
CREATE TABLE IF NOT EXISTS orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'processing', 'completed', 'cancelled') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user (user_id),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Données de test
INSERT INTO users (username, email, password_hash) VALUES
    ('alice', 'alice@dev.local', '$2y$10$abcdefghijklmnopqrstuvwxyz'),
    ('bob', 'bob@dev.local', '$2y$10$abcdefghijklmnopqrstuvwxyz'),
    ('charlie', 'charlie@dev.local', '$2y$10$abcdefghijklmnopqrstuvwxyz');

INSERT INTO products (name, description, price, stock, category) VALUES
    ('Laptop Pro', 'Ordinateur portable haute performance', 1299.99, 50, 'electronics'),
    ('Wireless Mouse', 'Souris sans fil ergonomique', 29.99, 200, 'accessories'),
    ('USB-C Cable', 'Câble USB-C 2m', 9.99, 500, 'accessories'),
    ('Monitor 27"', 'Écran 4K 27 pouces', 399.99, 30, 'electronics'),
    ('Keyboard Mech', 'Clavier mécanique RGB', 149.99, 75, 'accessories');

INSERT INTO orders (user_id, total_amount, status) VALUES
    (1, 1329.98, 'completed'),
    (2, 429.98, 'processing'),
    (3, 39.98, 'pending');

-- Afficher les statistiques
SELECT 'Users created:' AS Info, COUNT(*) AS Count FROM users
UNION ALL
SELECT 'Products created:', COUNT(*) FROM products
UNION ALL
SELECT 'Orders created:', COUNT(*) FROM orders;
```

### 4.2 Configuration MariaDB

Créez le fichier `mariadb/config/my.cnf` :

```ini
[mysqld]
# Encodage
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Performance
max_connections = 200
innodb_buffer_pool_size = 256M

# Logs
slow_query_log = ON
long_query_time = 2
```

### 4.3 Script PostgreSQL

Créez le fichier `postgres/init/01-init.sql` :

```sql
-- ==========================================
-- Script d'initialisation PostgreSQL
-- ==========================================

-- Connexion à la base de données
\c dev_db;

-- Table articles (blog)
CREATE TABLE IF NOT EXISTS articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(200) NOT NULL UNIQUE,
    content TEXT NOT NULL,
    author VARCHAR(100),
    published BOOLEAN DEFAULT FALSE,
    tags TEXT[],
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index pour améliorer les performances
CREATE INDEX idx_articles_slug ON articles(slug);
CREATE INDEX idx_articles_author ON articles(author);
CREATE INDEX idx_articles_published ON articles(published);
CREATE INDEX idx_articles_tags ON articles USING GIN(tags);
CREATE INDEX idx_articles_metadata ON articles USING GIN(metadata);

-- Table commentaires
CREATE TABLE IF NOT EXISTS comments (
    id SERIAL PRIMARY KEY,
    article_id INTEGER NOT NULL REFERENCES articles(id) ON DELETE CASCADE,
    author VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    rating INTEGER CHECK (rating >= 1 AND rating <= 5),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_comments_article ON comments(article_id);

-- Données de test
INSERT INTO articles (title, slug, content, author, published, tags, metadata) VALUES
    (
        'Introduction à PostgreSQL',
        'intro-postgresql',
        'PostgreSQL est une base de données relationnelle avancée...',
        'Alice Developer',
        TRUE,
        ARRAY['postgresql', 'database', 'sql'],
        '{"views": 1250, "likes": 89, "featured": true}'::jsonb
    ),
    (
        'Docker pour débutants',
        'docker-beginners',
        'Docker révolutionne le déploiement d''applications...',
        'Bob DevOps',
        TRUE,
        ARRAY['docker', 'devops', 'containers'],
        '{"views": 3500, "likes": 234, "featured": true}'::jsonb
    ),
    (
        'Guide des API REST',
        'rest-api-guide',
        'Les API REST sont essentielles pour les applications modernes...',
        'Charlie Backend',
        FALSE,
        ARRAY['api', 'rest', 'backend'],
        '{"views": 0, "likes": 0, "featured": false}'::jsonb
    );

INSERT INTO comments (article_id, author, content, rating) VALUES
    (1, 'John Doe', 'Excellent article, très clair !', 5),
    (1, 'Jane Smith', 'Merci pour ce tutoriel détaillé.', 5),
    (2, 'Mike Johnson', 'Docker a vraiment changé ma façon de travailler.', 4);

-- Afficher les statistiques
SELECT 'Articles créés: ' || COUNT(*) FROM articles;
SELECT 'Commentaires créés: ' || COUNT(*) FROM comments;
```

### 4.4 Script MongoDB

Créez le fichier `mongodb/init/01-init.js` :

```javascript
// ==========================================
// Script d'initialisation MongoDB
// ==========================================

// Connexion à la base de données
db = db.getSiblingDB('dev_db');

// Collection : Sessions utilisateurs
db.createCollection('sessions', {
    validator: {
        $jsonSchema: {
            bsonType: 'object',
            required: ['userId', 'token', 'expiresAt'],
            properties: {
                userId: {
                    bsonType: 'string',
                    description: 'ID de l\'utilisateur'
                },
                token: {
                    bsonType: 'string',
                    description: 'Token de session'
                },
                deviceInfo: {
                    bsonType: 'object',
                    properties: {
                        userAgent: { bsonType: 'string' },
                        ip: { bsonType: 'string' }
                    }
                },
                expiresAt: {
                    bsonType: 'date',
                    description: 'Date d\'expiration'
                }
            }
        }
    }
});

// Index TTL pour supprimer automatiquement les sessions expirées
db.sessions.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 });
db.sessions.createIndex({ userId: 1 });

// Collection : Logs d'événements
db.createCollection('event_logs');

db.event_logs.createIndex({ timestamp: -1 });
db.event_logs.createIndex({ level: 1 });
db.event_logs.createIndex({ source: 1 });

// Collection : Analytics
db.createCollection('analytics', {
    validator: {
        $jsonSchema: {
            bsonType: 'object',
            required: ['event', 'timestamp'],
            properties: {
                event: {
                    bsonType: 'string',
                    description: 'Type d\'événement'
                },
                userId: {
                    bsonType: 'string'
                },
                properties: {
                    bsonType: 'object'
                },
                timestamp: {
                    bsonType: 'date'
                }
            }
        }
    }
});

db.analytics.createIndex({ event: 1, timestamp: -1 });
db.analytics.createIndex({ userId: 1 });

// Insérer des données de test
db.sessions.insertMany([
    {
        userId: 'user_123',
        token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        deviceInfo: {
            userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)',
            ip: '192.168.1.100'
        },
        createdAt: new Date(),
        expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000) // 24h
    },
    {
        userId: 'user_456',
        token: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        deviceInfo: {
            userAgent: 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)',
            ip: '192.168.1.101'
        },
        createdAt: new Date(),
        expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
    }
]);

db.event_logs.insertMany([
    {
        level: 'INFO',
        source: 'authentication',
        message: 'User logged in successfully',
        userId: 'user_123',
        timestamp: new Date()
    },
    {
        level: 'WARN',
        source: 'payment',
        message: 'Payment retry after failure',
        orderId: 'ORD-98765',
        timestamp: new Date()
    },
    {
        level: 'ERROR',
        source: 'email',
        message: 'Failed to send notification email',
        error: 'SMTP connection timeout',
        timestamp: new Date()
    }
]);

db.analytics.insertMany([
    {
        event: 'page_view',
        userId: 'user_123',
        properties: {
            page: '/products',
            referrer: 'https://google.com',
            duration: 45
        },
        timestamp: new Date()
    },
    {
        event: 'add_to_cart',
        userId: 'user_123',
        properties: {
            productId: 'prod_001',
            quantity: 1,
            price: 29.99
        },
        timestamp: new Date()
    },
    {
        event: 'purchase',
        userId: 'user_456',
        properties: {
            orderId: 'ORD-12345',
            total: 149.99,
            items: 3
        },
        timestamp: new Date()
    }
]);

// Afficher les statistiques
print('=================================');
print('MongoDB initialisé avec succès !');
print('Sessions : ' + db.sessions.count());
print('Event logs : ' + db.event_logs.count());
print('Analytics : ' + db.analytics.count());
print('=================================');
```

### 4.5 Configuration Redis

Créez le fichier `redis/config/redis.conf` :

```conf
# ==========================================
# Configuration Redis pour développement
# ==========================================

# Bind sur toutes les interfaces (Docker)
bind 0.0.0.0

# Port (défaut)
port 6379

# Mot de passe (défini via docker-compose)
# requirepass redis_pass_2024

# Base de données (16 par défaut, 0-15)
databases 16

# Persistance sur disque
save 900 1
save 300 10
save 60 10000

dir /data
dbfilename dump.rdb

# AOF (Append Only File) pour plus de sécurité
appendonly yes
appendfilename "appendonly.aof"

# Logs
loglevel notice

# Mémoire maximum (éviter de consommer toute la RAM)
maxmemory 256mb
maxmemory-policy allkeys-lru
```

---

## ▶️ Étape 5 : Démarrer l'environnement

### 5.1 Premiers démarrages

```bash
# Depuis le dossier dev-multi-db

# Build et démarrage
docker-compose up -d
```

**⏳ Patience :** Le premier démarrage peut prendre 5-10 minutes :
- Téléchargement des images (~3-4 GB)
- Initialisation des bases de données
- Exécution des scripts d'init

### 5.2 Vérifier que tout fonctionne

```bash
# Voir les conteneurs actifs
docker-compose ps

# Résultat attendu (tous "Up" ou "Up (healthy)") :
# NAME                    STATE           PORTS
# dev_mariadb            Up (healthy)    3306/tcp
# dev_postgres           Up (healthy)    5432/tcp
# dev_mongodb            Up (healthy)    27017/tcp
# dev_redis              Up (healthy)    6379/tcp
# dev_phpmyadmin         Up              8080/tcp
# dev_pgadmin            Up              5050/tcp
# dev_mongo_express      Up              8081/tcp
# dev_redis_commander    Up              8082/tcp
```

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Logs d'un service spécifique
docker-compose logs mariadb
```

### 5.3 Vérifier les healthchecks

```bash
# Healthcheck de tous les services
docker-compose ps --format json | jq '.[] | {name: .Name, health: .Health}'
```

---

## 🌐 Étape 6 : Accéder aux interfaces graphiques

### 6.1 Tableau des accès

| Interface | URL | Identifiants | Base de données |
|-----------|-----|--------------|-----------------|
| **phpMyAdmin** | http://localhost:8080 | root / mariadb_root_pass_2024 | MariaDB |
| **pgAdmin** | http://localhost:5050 | admin@dev.local / pgadmin_pass_2024 | PostgreSQL |
| **Mongo Express** | http://localhost:8081 | admin / mongo_pass_2024 | MongoDB |
| **Redis Commander** | http://localhost:8082 | (pas d'auth) | Redis |

### 6.2 Configurer pgAdmin

**Première utilisation de pgAdmin :**

1. **Accéder à** http://localhost:5050
2. **Se connecter** avec les identifiants du `.env`
3. **Ajouter un serveur** :
   - Clic droit sur "Servers" → Register → Server
   - **General tab** :
     - Name : `PostgreSQL Dev`
   - **Connection tab** :
     - Host name/address : `postgres`
     - Port : `5432`
     - Username : `postgres`
     - Password : `postgres_pass_2024`
     - Save password : ✅
   - **Save**

4. **Explorer** : Servers → PostgreSQL Dev → Databases → dev_db

---

## 🔌 Étape 7 : Se connecter depuis votre application

### 7.1 Connexion depuis Node.js

**Installation des drivers :**

```bash
npm install mysql2 pg mongodb redis
```

**Exemples de connexion :**

```javascript
// ==========================================
// MARIADB (mysql2)
// ==========================================
const mysql = require('mysql2/promise');

async function connectMariaDB() {
    const connection = await mysql.createConnection({
        host: 'localhost',
        port: 3306,
        user: 'dev_user',
        password: 'dev_pass_2024',
        database: 'dev_db'
    });

    const [rows] = await connection.execute('SELECT * FROM users LIMIT 5');
    console.log('MariaDB users:', rows);

    await connection.end();
}

// ==========================================
// POSTGRESQL (pg)
// ==========================================
const { Client } = require('pg');

async function connectPostgreSQL() {
    const client = new Client({
        host: 'localhost',
        port: 5432,
        user: 'postgres',
        password: 'postgres_pass_2024',
        database: 'dev_db'
    });

    await client.connect();

    const res = await client.query('SELECT * FROM articles LIMIT 5');
    console.log('PostgreSQL articles:', res.rows);

    await client.end();
}

// ==========================================
// MONGODB (mongodb)
// ==========================================
const { MongoClient } = require('mongodb');

async function connectMongoDB() {
    const url = 'mongodb://mongo_admin:mongo_pass_2024@localhost:27017';
    const client = new MongoClient(url);

    await client.connect();

    const db = client.db('dev_db');
    const sessions = await db.collection('sessions').find({}).limit(5).toArray();
    console.log('MongoDB sessions:', sessions);

    await client.close();
}

// ==========================================
// REDIS (redis)
// ==========================================
const redis = require('redis');

async function connectRedis() {
    const client = redis.createClient({
        host: 'localhost',
        port: 6379,
        password: 'redis_pass_2024'
    });

    await client.connect();

    // Set/Get
    await client.set('test_key', 'Hello Redis!');
    const value = await client.get('test_key');
    console.log('Redis value:', value);

    await client.quit();
}

// Exécuter toutes les connexions
(async () => {
    try {
        await connectMariaDB();
        await connectPostgreSQL();
        await connectMongoDB();
        await connectRedis();
        console.log('✅ Toutes les connexions réussies !');
    } catch (error) {
        console.error('❌ Erreur:', error.message);
    }
})();
```

### 7.2 Connexion depuis Python

**Installation des drivers :**

```bash
pip install mysql-connector-python psycopg2-binary pymongo redis
```

**Exemples de connexion :**

```python
import mysql.connector
import psycopg2
from pymongo import MongoClient
import redis

# ==========================================
# MARIADB
# ==========================================
def connect_mariadb():
    conn = mysql.connector.connect(
        host='localhost',
        port=3306,
        user='dev_user',
        password='dev_pass_2024',
        database='dev_db'
    )

    cursor = conn.cursor(dictionary=True)
    cursor.execute('SELECT * FROM users LIMIT 5')
    users = cursor.fetchall()
    print('MariaDB users:', users)

    cursor.close()
    conn.close()

# ==========================================
# POSTGRESQL
# ==========================================
def connect_postgresql():
    conn = psycopg2.connect(
        host='localhost',
        port=5432,
        user='postgres',
        password='postgres_pass_2024',
        database='dev_db'
    )

    cursor = conn.cursor()
    cursor.execute('SELECT * FROM articles LIMIT 5')
    articles = cursor.fetchall()
    print('PostgreSQL articles:', articles)

    cursor.close()
    conn.close()

# ==========================================
# MONGODB
# ==========================================
def connect_mongodb():
    client = MongoClient(
        'mongodb://mongo_admin:mongo_pass_2024@localhost:27017'
    )

    db = client['dev_db']
    sessions = list(db.sessions.find().limit(5))
    print('MongoDB sessions:', sessions)

    client.close()

# ==========================================
# REDIS
# ==========================================
def connect_redis():
    client = redis.Redis(
        host='localhost',
        port=6379,
        password='redis_pass_2024',
        decode_responses=True
    )

    client.set('test_key', 'Hello Redis!')
    value = client.get('test_key')
    print('Redis value:', value)

    client.close()

# Exécuter toutes les connexions
if __name__ == '__main__':
    try:
        connect_mariadb()
        connect_postgresql()
        connect_mongodb()
        connect_redis()
        print('✅ Toutes les connexions réussies !')
    except Exception as e:
        print(f'❌ Erreur: {e}')
```

### 7.3 Connexion depuis PHP

```php
<?php
// ==========================================
// Configuration
// ==========================================
$config = [
    'mariadb' => [
        'host' => 'localhost',
        'port' => 3306,
        'user' => 'dev_user',
        'password' => 'dev_pass_2024',
        'database' => 'dev_db'
    ],
    'postgres' => [
        'host' => 'localhost',
        'port' => 5432,
        'user' => 'postgres',
        'password' => 'postgres_pass_2024',
        'database' => 'dev_db'
    ],
    'mongodb' => [
        'uri' => 'mongodb://mongo_admin:mongo_pass_2024@localhost:27017',
        'database' => 'dev_db'
    ],
    'redis' => [
        'host' => 'localhost',
        'port' => 6379,
        'password' => 'redis_pass_2024'
    ]
];

// ==========================================
// MARIADB (PDO)
// ==========================================
try {
    $pdo = new PDO(
        "mysql:host={$config['mariadb']['host']};port={$config['mariadb']['port']};dbname={$config['mariadb']['database']};charset=utf8mb4",
        $config['mariadb']['user'],
        $config['mariadb']['password']
    );

    $stmt = $pdo->query('SELECT * FROM users LIMIT 5');
    $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
    echo "MariaDB users: " . count($users) . " trouvés\n";
} catch (PDOException $e) {
    echo "Erreur MariaDB: " . $e->getMessage() . "\n";
}

// ==========================================
// POSTGRESQL (PDO)
// ==========================================
try {
    $pdo = new PDO(
        "pgsql:host={$config['postgres']['host']};port={$config['postgres']['port']};dbname={$config['postgres']['database']}",
        $config['postgres']['user'],
        $config['postgres']['password']
    );

    $stmt = $pdo->query('SELECT * FROM articles LIMIT 5');
    $articles = $stmt->fetchAll(PDO::FETCH_ASSOC);
    echo "PostgreSQL articles: " . count($articles) . " trouvés\n";
} catch (PDOException $e) {
    echo "Erreur PostgreSQL: " . $e->getMessage() . "\n";
}

// ==========================================
// MONGODB (Extension mongodb requise)
// ==========================================
try {
    $client = new MongoDB\Client($config['mongodb']['uri']);
    $db = $client->{$config['mongodb']['database']};

    $sessions = $db->sessions->find([], ['limit' => 5]);
    $count = iterator_count($sessions);
    echo "MongoDB sessions: {$count} trouvées\n";
} catch (Exception $e) {
    echo "Erreur MongoDB: " . $e->getMessage() . "\n";
}

// ==========================================
// REDIS (Extension redis ou Predis requis)
// ==========================================
try {
    $redis = new Redis();
    $redis->connect($config['redis']['host'], $config['redis']['port']);
    $redis->auth($config['redis']['password']);

    $redis->set('test_key', 'Hello Redis!');
    $value = $redis->get('test_key');
    echo "Redis value: {$value}\n";
} catch (Exception $e) {
    echo "Erreur Redis: " . $e->getMessage() . "\n";
}

echo "✅ Tests de connexion terminés !\n";
?>
```

---

## 📊 Étape 8 : Scénarios d'utilisation

### 8.1 Scénario 1 : Application e-commerce

**Répartition des données :**

| Base | Usage | Exemple |
|------|-------|---------|
| **MariaDB** | Données transactionnelles | Users, Products, Orders |
| **PostgreSQL** | Données analytiques | Reviews, Reports, Analytics |
| **MongoDB** | Données flexibles | Session, Logs, Events |
| **Redis** | Cache et temps réel | Cart, Sessions, Counters |

**Flux de données :**

```
1. Utilisateur se connecte
   → Session stockée dans Redis (cache rapide)
   → User data depuis MariaDB

2. Utilisateur consulte un produit
   → Product depuis MariaDB (ou cache Redis)
   → Event logged dans MongoDB

3. Utilisateur ajoute au panier
   → Cart stocké dans Redis (temporaire)
   → Event logged dans MongoDB

4. Utilisateur passe commande
   → Order créée dans MariaDB (transaction)
   → Cart supprimé de Redis
   → Event logged dans MongoDB

5. Analyse des ventes
   → Données agrégées depuis PostgreSQL
```

### 8.2 Scénario 2 : Application SaaS multi-tenant

```
Tenant 1 : Entreprise A
├─ MariaDB    → Données clients, facturation
├─ PostgreSQL → Analytics, reporting
├─ MongoDB    → Logs, events, audits
└─ Redis      → Cache, rate limiting

Tenant 2 : Entreprise B
├─ MariaDB    → Données clients, facturation
├─ PostgreSQL → Analytics, reporting
├─ MongoDB    → Logs, events, audits
└─ Redis      → Cache, rate limiting
```

### 8.3 Scénario 3 : Microservices

```
Service Authentication
└─ MariaDB (users) + Redis (sessions)

Service Catalog
└─ PostgreSQL (products) + Redis (cache)

Service Orders
└─ MariaDB (orders) + MongoDB (logs)

Service Analytics
└─ MongoDB (events) + PostgreSQL (reports)
```

---

## 🛠️ Étape 9 : Gestion et maintenance

### 9.1 Commandes utiles

```bash
# Voir tous les conteneurs
docker-compose ps

# Logs de tous les services
docker-compose logs

# Logs en temps réel d'un service
docker-compose logs -f mariadb

# Statistiques de ressources
docker stats

# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer tous les services
docker-compose start

# Redémarrer un service spécifique
docker-compose restart redis

# Arrêter et supprimer les conteneurs (données conservées dans volumes)
docker-compose down

# Supprimer tout (⚠️ SUPPRIME LES DONNÉES)
docker-compose down -v
```

### 9.2 Backup des bases de données

**MariaDB :**

```bash
# Backup
docker exec dev_mariadb mysqldump -u root -pmariadb_root_pass_2024 --all-databases > backup_mariadb_$(date +%Y%m%d).sql

# Restore
cat backup_mariadb_20241102.sql | docker exec -i dev_mariadb mysql -u root -pmariadb_root_pass_2024
```

**PostgreSQL :**

```bash
# Backup
docker exec dev_postgres pg_dumpall -U postgres > backup_postgres_$(date +%Y%m%d).sql

# Restore
cat backup_postgres_20241102.sql | docker exec -i dev_postgres psql -U postgres
```

**MongoDB :**

```bash
# Backup
docker exec dev_mongodb mongodump --username mongo_admin --password mongo_pass_2024 --out /dump
docker cp dev_mongodb:/dump ./backup_mongodb_$(date +%Y%m%d)

# Restore
docker exec dev_mongodb mongorestore --username mongo_admin --password mongo_pass_2024 /dump
```

**Redis :**

```bash
# Backup (snapshot automatique selon config)
docker exec dev_redis redis-cli -a redis_pass_2024 SAVE

# Copier le snapshot
docker cp dev_redis:/data/dump.rdb ./backup_redis_$(date +%Y%m%d).rdb
```

### 9.3 Monitoring des ressources

**Script de monitoring simple :**

```bash
#!/bin/bash
# monitoring.sh

echo "==================================="
echo "Monitoring Environnement Multi-BDD"
echo "==================================="
echo ""

# Statut des conteneurs
echo "📊 Statut des services:"
docker-compose ps --format "table {{.Name}}\t{{.Status}}\t{{.Ports}}"
echo ""

# Utilisation ressources
echo "💻 Utilisation des ressources:"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
echo ""

# Espace disque
echo "💾 Espace disque utilisé par les volumes:"
docker system df -v | grep -A 20 "Local Volumes:"
echo ""

# Connexions actives
echo "🔌 Connexions actives:"
echo "MariaDB:"
docker exec dev_mariadb mysql -u root -pmariadb_root_pass_2024 -e "SHOW STATUS LIKE 'Threads_connected';"
echo ""
echo "PostgreSQL:"
docker exec dev_postgres psql -U postgres -c "SELECT count(*) as connections FROM pg_stat_activity;"
echo ""
echo "MongoDB:"
docker exec dev_mongodb mongosh --quiet --username mongo_admin --password mongo_pass_2024 --eval "db.serverStatus().connections"
echo ""
echo "Redis:"
docker exec dev_redis redis-cli -a redis_pass_2024 INFO clients | grep connected_clients
```

Rendre le script exécutable :

```bash
chmod +x monitoring.sh
./monitoring.sh
```

---

## 🛑 Étape 10 : Arrêt et nettoyage

### 10.1 Arrêt propre

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

# Supprimer les volumes (⚠️ SUPPRIME TOUTES LES DONNÉES)
docker-compose down -v

# Supprimer les données manuellement
rm -rf mariadb/data/*
rm -rf postgres/data/*
rm -rf mongodb/data/*
rm -rf redis/data/*
rm -rf pgadmin/data/*

# Supprimer les images (optionnel)
docker rmi mariadb:10.11 postgres:15 mongo:7.0 redis:7-alpine
docker rmi phpmyadmin:latest dpage/pgadmin4:latest mongo-express:latest rediscommander/redis-commander:latest

# Supprimer le réseau
docker network rm dev-multi-db_dev_network
```

---

## 🐛 Dépannage

### Problème 1 : "Cannot allocate memory"

**Symptôme :** Les conteneurs se redémarrent en boucle ou ne démarrent pas.

**Cause :** Pas assez de RAM allouée à Docker.

**Solutions :**

1. **Augmenter la RAM Docker Desktop :**
   - Settings → Resources → Memory → 6-8 GB

2. **Réduire les ressources des conteneurs :**
   Dans `docker-compose.yml`, ajouter :
   ```yaml
   deploy:
     resources:
       limits:
         memory: 512M
   ```

3. **Démarrer les services progressivement :**
   ```bash
   docker-compose up -d mariadb postgres
   # Attendre 30 secondes
   docker-compose up -d mongodb redis
   # Attendre 30 secondes
   docker-compose up -d phpmyadmin pgadmin mongo-express redis-commander
   ```

### Problème 2 : Port déjà utilisé

**Symptôme :** "Bind for 0.0.0.0:3306 failed: port is already allocated"

**Solutions :**

1. **Changer le port dans docker-compose.yml :**
   ```yaml
   ports:
     - "3307:3306"  # Utilisez 3307 au lieu de 3306
   ```

2. **Identifier et arrêter le service qui utilise le port :**
   ```bash
   # Linux/macOS
   lsof -i :3306

   # Windows
   netstat -ano | findstr :3306
   ```

### Problème 3 : Données non persistantes

**Symptôme :** Les données disparaissent après `docker-compose down`.

**Cause :** Volumes non montés ou option `-v` utilisée.

**Solutions :**

1. **Vérifier les volumes dans docker-compose.yml :**
   ```yaml
   volumes:
     - ./mariadb/data:/var/lib/mysql  # Doit être présent
   ```

2. **Ne pas utiliser l'option -v lors du down :**
   ```bash
   docker-compose down     # ✅ Conserve les volumes
   docker-compose down -v  # ❌ Supprime les volumes
   ```

### Problème 4 : Connexion refusée depuis l'application

**Symptôme :** "Connection refused" ou "ECONNREFUSED" dans l'application.

**Solutions :**

1. **Vérifier que le service est démarré :**
   ```bash
   docker-compose ps
   ```

2. **Vérifier les healthchecks :**
   ```bash
   docker inspect dev_mariadb | grep -A 5 Health
   ```

3. **Vérifier la connexion depuis le conteneur :**
   ```bash
   # MariaDB
   docker exec dev_mariadb mysql -u root -p -e "SELECT 1;"

   # PostgreSQL
   docker exec dev_postgres psql -U postgres -c "SELECT 1;"

   # MongoDB
   docker exec dev_mongodb mongosh --eval "db.version()"

   # Redis
   docker exec dev_redis redis-cli -a redis_pass_2024 PING
   ```

4. **Utiliser les noms de services Docker (si l'app est dans Docker) :**
   ```javascript
   // ❌ Depuis un conteneur Docker
   host: 'localhost'

   // ✅ Depuis un conteneur Docker
   host: 'mariadb'  // Nom du service dans docker-compose.yml
   ```

### Problème 5 : pgAdmin ne se connecte pas à PostgreSQL

**Symptôme :** "Unable to connect to server"

**Solutions :**

1. **Utiliser le nom du service Docker, pas localhost :**
   - Host : `postgres` (pas `localhost`)

2. **Vérifier le réseau :**
   ```bash
   docker network inspect dev-multi-db_dev_network
   # pgadmin et postgres doivent être listés
   ```

3. **Réinitialiser la configuration pgAdmin :**
   ```bash
   docker-compose stop pgadmin
   rm -rf pgadmin/data/*
   docker-compose start pgadmin
   ```

---

## ✅ Récapitulatif

### Ce que vous avez appris

- ✅ Comprendre le concept de Polyglot Persistence
- ✅ Déployer un environnement multi-BDD complet avec Docker Compose
- ✅ Configurer MariaDB, PostgreSQL, MongoDB et Redis ensemble
- ✅ Utiliser des interfaces graphiques pour chaque base
- ✅ Se connecter depuis différents langages (Node.js, Python, PHP)
- ✅ Gérer les scripts d'initialisation
- ✅ Monitorer et maintenir l'environnement
- ✅ Effectuer des backups

### Technologies maîtrisées

| Technologie | Version | Rôle |
|-------------|---------|------|
| MariaDB | 10.11 | Base relationnelle (MySQL) |
| PostgreSQL | 15 | Base relationnelle avancée |
| MongoDB | 7.0 | Base NoSQL documents |
| Redis | 7 | Cache et Key-Value |
| phpMyAdmin | Latest | Interface MariaDB |
| pgAdmin | Latest | Interface PostgreSQL |
| Mongo Express | Latest | Interface MongoDB |
| Redis Commander | Latest | Interface Redis |

### Ports utilisés

| Service | Port | Accès |
|---------|------|-------|
| MariaDB | 3306 | Base de données |
| PostgreSQL | 5432 | Base de données |
| MongoDB | 27017 | Base de données |
| Redis | 6379 | Base de données |
| phpMyAdmin | 8080 | Interface web |
| pgAdmin | 5050 | Interface web |
| Mongo Express | 8081 | Interface web |
| Redis Commander | 8082 | Interface web |

### Adresses IP fixes

| Service | IP | Utilité |
|---------|-----|---------|
| MariaDB | 172.25.0.10 | Connexion directe par IP |
| PostgreSQL | 172.25.0.11 | Connexion directe par IP |
| MongoDB | 172.25.0.12 | Connexion directe par IP |
| Redis | 172.25.0.13 | Connexion directe par IP |

### Commandes essentielles

```bash
# Démarrer l'environnement
docker-compose up -d

# Voir le statut
docker-compose ps

# Logs
docker-compose logs -f

# Arrêter (données conservées)
docker-compose stop

# Redémarrer
docker-compose start

# Monitoring
docker stats

# Backups
./monitoring.sh

# Nettoyage complet
docker-compose down -v
```

---

## 🚀 Pour aller plus loin

### Extensions possibles

1. **Ajouter Elasticsearch**
   - Pour la recherche full-text
   - Compléter la stack ELK

2. **Ajouter Neo4j**
   - Base de données graphe
   - Relations complexes

3. **Ajouter Cassandra**
   - Base NoSQL distribuée
   - Big Data

4. **Ajouter InfluxDB**
   - Time series database
   - Métriques et IoT

5. **Ajouter RabbitMQ / Kafka**
   - Message broker
   - Architecture événementielle

6. **Configurer les replicas**
   - Haute disponibilité
   - Réplication master-slave

7. **Ajouter Metabase / Grafana**
   - Business Intelligence
   - Visualisation de données

### Scénarios avancés

| Scénario | Description |
|----------|-------------|
| **CQRS** | Command Query Responsibility Segregation |
| **Event Sourcing** | Stockage d'événements avec replay |
| **Multi-tenancy** | Isolation des données par client |
| **Sharding** | Distribution horizontale des données |
| **Data Lake** | Centralisation de toutes les données |

### Ressources complémentaires

- 📖 [Martin Fowler - PolyglotPersistence](https://martinfowler.com/bliki/PolyglotPersistence.html)
- 📖 [Microservices Patterns](https://microservices.io/patterns/data/database-per-service.html)
- 📖 [Docker Multi-container Apps](https://docs.docker.com/compose/)
- 🎓 [Database Design Course](https://www.coursera.org/learn/database-design)

---

## 🎉 Félicitations !

Vous avez maintenant un **environnement de développement complet multi-BDD** ! Cette infrastructure vous permet de :

- 🗄️ Tester différentes technologies de bases de données
- 🎯 Choisir la meilleure BDD pour chaque use case
- 🔄 Reproduire un environnement de production
- 🚀 Développer des applications modernes et scalables
- 📊 Comparer les performances et fonctionnalités
- 🐳 Déployer facilement avec Docker

**Prochain pas :** Explorez les autres guides du tutoriel !

➡️ [Stack LAMP (Apache + MariaDB + PHP)](01-stack-lamp.md)
➡️ [Stack MEAN (MongoDB + Express + Angular + Node)](02-stack-mean.md)
➡️ [Stack ELK (Elasticsearch + Logstash + Kibana)](03-stack-elk.md)
➡️ [Migration de données entre BDD](05-migration-donnees.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
