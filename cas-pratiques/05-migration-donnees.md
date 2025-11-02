# Migration de Données entre Bases de Données

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

La **migration de données** consiste à transférer des données d'un système de gestion de base de données (SGBD) vers un autre. C'est une opération courante dans le cycle de vie d'une application, mais qui peut sembler intimidante pour les débutants.

**Ce guide vous apprendra :**
- 🔄 Les différentes stratégies de migration
- 🛠️ Les outils disponibles pour chaque type de migration
- 📝 Comment planifier une migration
- 🐳 Comment utiliser Docker pour tester vos migrations
- ⚠️ Les pièges courants à éviter
- ✅ Comment valider vos migrations

**Durée estimée :** Lecture 30 minutes, mise en pratique variable selon complexité

---

## 🎯 Pourquoi Migrer ?

### Raisons courantes de migration

| Raison | Exemple | Fréquence |
|--------|---------|-----------|
| **Changement technologique** | MySQL → PostgreSQL | Très courant |
| **Montée en charge** | SQLite → PostgreSQL | Courant |
| **Évolution architecture** | Monolithe SQL → Microservices NoSQL | Courant |
| **Coûts** | Oracle → MariaDB (open source) | Courant |
| **Performances** | MySQL → MongoDB pour certains cas | Moyen |
| **Fonctionnalités spécifiques** | SQL classique → Neo4j (graphes) | Moins courant |
| **Cloud migration** | On-premise → Cloud (AWS RDS, etc.) | Très courant |

### 💡 Scénarios réels

**Scénario 1 : Startup qui grandit**
```
SQLite (prototype) → PostgreSQL (production)
Raison : SQLite limité pour la concurrence
```

**Scénario 2 : Optimisation des coûts**
```
Oracle → PostgreSQL
Raison : Réduire les licences tout en gardant les fonctionnalités
```

**Scénario 3 : Modernisation**
```
MySQL → MongoDB
Raison : Schéma flexible pour données JSON
```

---

## 🗺️ Types de Migrations

### 1. Migrations Homogènes (Même Type)

**Définition :** Migration entre deux SGBD du même type (SQL → SQL, NoSQL → NoSQL)

**Exemples :**
- MySQL → MariaDB
- PostgreSQL 12 → PostgreSQL 15
- MongoDB 4.4 → MongoDB 6.0

**Difficulté :** 🟢 Facile à 🟡 Moyenne

**Avantages :**
- ✅ Structure des données similaire
- ✅ Outils natifs souvent disponibles
- ✅ Moins de transformation de données

### 2. Migrations Hétérogènes (Type Différent)

**Définition :** Migration entre SGBD de types différents

**Exemples :**
- SQL → NoSQL : MySQL → MongoDB
- NoSQL → SQL : MongoDB → PostgreSQL
- Relationnel → Graphe : PostgreSQL → Neo4j
- SQL → Time-Series : MySQL → InfluxDB

**Difficulté :** 🟡 Moyenne à 🔴 Difficile

**Défis :**
- ⚠️ Modèles de données très différents
- ⚠️ Transformation complexe nécessaire
- ⚠️ Perte potentielle de fonctionnalités
- ⚠️ Nécessite souvent du code personnalisé

---

## 📊 Matrice de Compatibilité

### Facilité de Migration

| Source ↓ / Cible → | MySQL | PostgreSQL | MongoDB | Redis | Elasticsearch | Neo4j |
|-------------------|-------|------------|---------|-------|---------------|-------|
| **MySQL** | 🟢 Trivial | 🟡 Moyen | 🟡 Moyen | 🔴 Difficile | 🟡 Moyen | 🔴 Difficile |
| **PostgreSQL** | 🟡 Moyen | 🟢 Trivial | 🟡 Moyen | 🔴 Difficile | 🟡 Moyen | 🔴 Difficile |
| **MongoDB** | 🔴 Difficile | 🔴 Difficile | 🟢 Trivial | 🟡 Moyen | 🟢 Facile | 🟡 Moyen |
| **Redis** | 🔴 Difficile | 🔴 Difficile | 🟡 Moyen | 🟢 Trivial | 🟡 Moyen | 🔴 Difficile |
| **SQLite** | 🟢 Facile | 🟢 Facile | 🟡 Moyen | 🔴 Difficile | 🟡 Moyen | 🔴 Difficile |

**Légende :**
- 🟢 Facile : Outils automatiques disponibles, peu de transformation
- 🟡 Moyen : Nécessite scripts, transformation modérée
- 🔴 Difficile : Transformation complexe, code personnalisé nécessaire

---

## 🎓 Concepts Clés

### 1. ETL (Extract, Transform, Load)

Le processus ETL est le cœur de toute migration de données.

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   EXTRACT   │────→│  TRANSFORM  │────→│    LOAD     │
│ (Source DB) │     │ (Conversion)│     │ (Target DB) │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Détail des étapes :**

#### Extract (Extraction)
```bash
# Exemple : Exporter depuis MySQL
mysqldump -u root -p ma_base > data.sql

# Exemple : Exporter depuis MongoDB
mongoexport --db ma_base --collection users --out users.json
```

#### Transform (Transformation)
```python
# Exemple : Convertir des données
import json

# Lire depuis MongoDB (JSON)
with open('users.json') as f:
    users_mongo = [json.loads(line) for line in f]

# Transformer pour PostgreSQL (SQL)
sql_inserts = []
for user in users_mongo:
    sql = f"INSERT INTO users (id, name, email) VALUES ({user['_id']}, '{user['name']}', '{user['email']}');"
    sql_inserts.append(sql)

# Sauvegarder
with open('users.sql', 'w') as f:
    f.write('\n'.join(sql_inserts))
```

#### Load (Chargement)
```bash
# Charger dans PostgreSQL
psql -U postgres -d ma_base -f users.sql
```

### 2. Schéma vs Schemaless

**Bases relationnelles (SQL) - Schéma strict**
```sql
-- Structure définie à l'avance
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Bases NoSQL - Schéma flexible**
```javascript
// Structure libre
{
    "_id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "preferences": {  // Peut varier d'un document à l'autre
        "theme": "dark",
        "notifications": true
    }
}
```

**💡 Implication pour la migration :**
- SQL → NoSQL : Facile (ajouter flexibilité)
- NoSQL → SQL : Difficile (contraindre structure)

### 3. Types de Données

Chaque SGBD a ses propres types de données.

**Exemple de correspondance MySQL → PostgreSQL :**

| MySQL | PostgreSQL | Notes |
|-------|------------|-------|
| `INT` | `INTEGER` | Identique |
| `VARCHAR(n)` | `VARCHAR(n)` | Identique |
| `TEXT` | `TEXT` | Identique |
| `DATETIME` | `TIMESTAMP` | Comportement différent avec fuseaux horaires |
| `TINYINT(1)` | `BOOLEAN` | MySQL utilise 0/1, PostgreSQL TRUE/FALSE |
| `ENUM('a','b')` | `TEXT` ou custom ENUM | PostgreSQL plus strict |

### 4. Clés et Contraintes

**Éléments à migrer :**
- ✅ Clés primaires (PRIMARY KEY)
- ✅ Clés étrangères (FOREIGN KEY)
- ✅ Index
- ✅ Contraintes UNIQUE
- ✅ Contraintes CHECK
- ✅ Valeurs par défaut (DEFAULT)
- ✅ Triggers
- ✅ Vues
- ✅ Procédures stockées

⚠️ **Attention :** La syntaxe varie entre SGBD !

---

## 🛠️ Outils de Migration

### 1. Outils Natifs (Dumps)

Chaque SGBD fournit des outils pour exporter/importer.

#### MySQL / MariaDB

**Export :**
```bash
# Export complet
mysqldump -u root -p --all-databases > full_backup.sql

# Export d'une base spécifique
mysqldump -u root -p ma_base > ma_base.sql

# Export structure uniquement (sans données)
mysqldump -u root -p --no-data ma_base > structure.sql

# Export données uniquement (sans structure)
mysqldump -u root -p --no-create-info ma_base > donnees.sql
```

**Import :**
```bash
# Import
mysql -u root -p < full_backup.sql

# Ou dans le shell MySQL
mysql -u root -p
source /chemin/vers/fichier.sql;
```

#### PostgreSQL

**Export :**
```bash
# Export complet (toutes bases)
pg_dumpall -U postgres > full_backup.sql

# Export d'une base
pg_dump -U postgres ma_base > ma_base.sql

# Export en format personnalisé (compressé)
pg_dump -U postgres -Fc ma_base > ma_base.dump
```

**Import :**
```bash
# Import SQL
psql -U postgres ma_base < ma_base.sql

# Import format personnalisé
pg_restore -U postgres -d ma_base ma_base.dump
```

#### MongoDB

**Export :**
```bash
# Export d'une base (format JSON)
mongodump --db ma_base --out /backup/

# Export d'une collection (JSON)
mongoexport --db ma_base --collection users --out users.json

# Export en CSV
mongoexport --db ma_base --collection users --type=csv --fields name,email --out users.csv
```

**Import :**
```bash
# Import d'une base
mongorestore --db ma_base /backup/ma_base/

# Import d'une collection
mongoimport --db ma_base --collection users --file users.json
```

#### Redis

**Export :**
```bash
# Redis sauvegarde automatiquement dans dump.rdb
# Forcer une sauvegarde
redis-cli SAVE

# Copier le fichier dump.rdb
cp /var/lib/redis/dump.rdb /backup/
```

**Import :**
```bash
# Restaurer : remplacer dump.rdb et redémarrer Redis
cp /backup/dump.rdb /var/lib/redis/
redis-cli SHUTDOWN
redis-server
```

### 2. Outils Tiers

#### pgLoader (Très populaire pour migrations vers PostgreSQL)

**Installation :**
```bash
# Avec Docker (recommandé)
docker pull dimitri/pgloader
```

**Migration MySQL → PostgreSQL :**
```bash
# Commande simple
docker run --rm -it dimitri/pgloader \
    pgloader mysql://user:pass@mysql_host/db_source \
             postgresql://user:pass@pg_host/db_target

# Avec fichier de configuration (plus de contrôle)
docker run --rm -it -v $(pwd):/data dimitri/pgloader \
    pgloader /data/migration.load
```

**Fichier de configuration `migration.load` :**
```lisp
LOAD DATABASE
    FROM mysql://root:password@mariadb_source:3306/ma_base
    INTO postgresql://postgres:password@postgres_target:5432/ma_base

WITH include drop, create tables, create indexes, reset sequences

SET maintenance_work_mem to '512MB',
    work_mem to '128MB'

CAST type datetime to timestamp
      drop default drop not null using zero-dates-to-null;
```

#### DBeaver (Interface graphique)

**Avantages :**
- ✅ Interface visuelle
- ✅ Support de nombreux SGBD
- ✅ Assistant de migration intégré

**Utilisation :**
1. Connecter les deux bases (source + cible)
2. Clic droit sur table source → "Export Data"
3. Choisir le format et la destination
4. Import dans la base cible

#### ETL Tools (Pour migrations complexes)

| Outil | Type | Usage | Gratuit |
|-------|------|-------|---------|
| **Apache NiFi** | ETL Visuel | Flux de données complexes | ✅ |
| **Talend** | ETL | Transformations avancées | ⚠️ Community |
| **Pentaho** | ETL | Business Intelligence | ⚠️ Community |
| **Airbyte** | ELT Moderne | Pipelines de données | ✅ |

### 3. Scripts Personnalisés

Pour les migrations complexes, souvent nécessaire d'écrire des scripts.

**Langages populaires :**
- 🐍 Python (pandas, SQLAlchemy)
- ☕ Java (JDBC)
- 🟦 Node.js (Sequelize, Mongoose)
- 🐹 Go (database/sql)

**Exemple Python (MySQL → PostgreSQL) :**
```python
import pymysql
import psycopg2
from psycopg2 import sql

# Connexion source (MySQL)
mysql_conn = pymysql.connect(
    host='localhost',
    user='root',
    password='mysql_pass',
    database='source_db'
)

# Connexion cible (PostgreSQL)
pg_conn = psycopg2.connect(
    host='localhost',
    user='postgres',
    password='pg_pass',
    database='target_db'
)

# Extraction depuis MySQL
with mysql_conn.cursor() as cursor:
    cursor.execute("SELECT id, name, email FROM users")
    users = cursor.fetchall()

# Chargement dans PostgreSQL
with pg_conn.cursor() as cursor:
    for user in users:
        cursor.execute(
            "INSERT INTO users (id, name, email) VALUES (%s, %s, %s)",
            user
        )
    pg_conn.commit()

print(f"Migré {len(users)} utilisateurs")

# Fermeture
mysql_conn.close()
pg_conn.close()
```

---

## 📝 Guide Pratique par Type de Migration

### Migration 1 : MySQL → PostgreSQL

**Difficulté :** 🟡 Moyenne

**Étapes :**

#### Méthode 1 : pgLoader (Recommandée)

```bash
# 1. Préparer l'environnement Docker
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  mysql_source:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: mysql_pass
      MYSQL_DATABASE: source_db
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  postgres_target:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: pg_pass
      POSTGRES_DB: target_db
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

volumes:
  mysql_data:
  pg_data:
EOF

# 2. Démarrer les bases
docker-compose up -d

# 3. Insérer des données de test dans MySQL
docker exec -it <mysql_container> mysql -u root -pmysql_pass -e "
USE source_db;
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(255)
);
INSERT INTO users (name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com');
"

# 4. Créer le fichier de migration
cat > migration.load << 'EOF'
LOAD DATABASE
    FROM mysql://root:mysql_pass@mysql_source:3306/source_db
    INTO postgresql://postgres:pg_pass@postgres_target:5432/target_db

WITH include drop, create tables
EOF

# 5. Exécuter la migration avec pgLoader
docker run --rm --network=host dimitri/pgloader \
    pgloader mysql://root:mysql_pass@localhost:3306/source_db \
             postgresql://postgres:pg_pass@localhost:5432/target_db

# 6. Vérifier dans PostgreSQL
docker exec -it <postgres_container> psql -U postgres -d target_db -c "SELECT * FROM users;"
```

#### Méthode 2 : Dump + Conversion manuelle

```bash
# 1. Export depuis MySQL
mysqldump -u root -p --compatible=postgresql source_db > mysql_dump.sql

# 2. Conversion manuelle (exemples de changements nécessaires)
# MySQL:  AUTO_INCREMENT -> PostgreSQL: SERIAL
# MySQL:  TINYINT(1) -> PostgreSQL: BOOLEAN
# MySQL:  ` (backticks) -> PostgreSQL: " (guillemets)

# Exemple de script sed pour conversions simples
sed -i 's/AUTO_INCREMENT/SERIAL/g' mysql_dump.sql
sed -i "s/\`/\"/g" mysql_dump.sql

# 3. Import dans PostgreSQL
psql -U postgres -d target_db -f mysql_dump.sql
```

**Pièges courants :**
- ⚠️ Les types DATETIME vs TIMESTAMP (gestion timezone)
- ⚠️ Les quotes : MySQL utilise `, PostgreSQL utilise "
- ⚠️ Les ENUM : Syntaxe différente
- ⚠️ Les AUTO_INCREMENT vs SERIAL

### Migration 2 : MongoDB → PostgreSQL

**Difficulté :** 🔴 Difficile (schemaless → schema)

**Stratégie :** Normaliser les documents JSON en tables relationnelles

**Exemple :**

**Document MongoDB (users) :**
```json
{
    "_id": ObjectId("..."),
    "name": "Alice",
    "email": "alice@example.com",
    "addresses": [
        {
            "type": "home",
            "street": "123 Main St",
            "city": "Paris"
        },
        {
            "type": "work",
            "street": "456 Work Ave",
            "city": "Lyon"
        }
    ],
    "created_at": ISODate("2024-01-15T10:30:00Z")
}
```

**Schéma PostgreSQL (normalisé) :**
```sql
-- Table principale
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP
);

-- Table liée (relation 1-N)
CREATE TABLE addresses (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    type VARCHAR(20),
    street VARCHAR(255),
    city VARCHAR(100)
);
```

**Script Python de migration :**
```python
from pymongo import MongoClient
import psycopg2

# Connexion MongoDB
mongo_client = MongoClient('mongodb://localhost:27017/')
mongo_db = mongo_client['source_db']

# Connexion PostgreSQL
pg_conn = psycopg2.connect(
    host='localhost',
    user='postgres',
    password='password',
    database='target_db'
)
pg_cursor = pg_conn.cursor()

# Migration
for doc in mongo_db.users.find():
    # Insérer utilisateur
    pg_cursor.execute(
        "INSERT INTO users (name, email, created_at) VALUES (%s, %s, %s) RETURNING id",
        (doc['name'], doc['email'], doc['created_at'])
    )
    user_id = pg_cursor.fetchone()[0]

    # Insérer adresses liées
    for addr in doc.get('addresses', []):
        pg_cursor.execute(
            "INSERT INTO addresses (user_id, type, street, city) VALUES (%s, %s, %s, %s)",
            (user_id, addr['type'], addr['street'], addr['city'])
        )

pg_conn.commit()
pg_cursor.close()
pg_conn.close()
```

**Défis spécifiques :**
- 🔄 Normalisation des données imbriquées
- 🔄 Gestion des tableaux (arrays)
- 🔄 Choix du schéma (normalisation vs JSONB)
- 🔄 ObjectId MongoDB → ID PostgreSQL

**Alternative : Utiliser JSONB**

Si vous voulez conserver la flexibilité :
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    data JSONB  -- Stocker le document complet
);

INSERT INTO users (data) VALUES ('{"name": "Alice", "email": "alice@example.com"}');

-- Requête sur le JSON
SELECT data->>'name' AS name FROM users WHERE data->>'email' = 'alice@example.com';
```

### Migration 3 : PostgreSQL → MongoDB

**Difficulté :** 🟡 Moyenne (schema → schemaless)

**Stratégie :** Dénormaliser les tables relationnelles en documents

**Schéma PostgreSQL :**
```sql
-- Tables normalisées
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    product VARCHAR(100),
    amount DECIMAL(10,2),
    created_at TIMESTAMP
);
```

**Documents MongoDB (dénormalisés) :**
```javascript
// Un seul document contenant tout
{
    "_id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "orders": [
        {
            "id": 101,
            "product": "Laptop",
            "amount": 999.99,
            "created_at": ISODate("2024-01-15")
        },
        {
            "id": 102,
            "product": "Mouse",
            "amount": 29.99,
            "created_at": ISODate("2024-01-16")
        }
    ]
}
```

**Script Python :**
```python
import psycopg2
from pymongo import MongoClient

# Connexion PostgreSQL
pg_conn = psycopg2.connect(
    host='localhost',
    user='postgres',
    password='password',
    database='source_db'
)

# Connexion MongoDB
mongo_client = MongoClient('mongodb://localhost:27017/')
mongo_db = mongo_client['target_db']

# Extraction et transformation
with pg_conn.cursor() as cursor:
    cursor.execute("SELECT id, name, email FROM users")
    users = cursor.fetchall()

    for user in users:
        user_id, name, email = user

        # Récupérer les commandes de l'utilisateur
        cursor.execute(
            "SELECT id, product, amount, created_at FROM orders WHERE user_id = %s",
            (user_id,)
        )
        orders = cursor.fetchall()

        # Créer le document MongoDB
        doc = {
            "_id": user_id,
            "name": name,
            "email": email,
            "orders": [
                {
                    "id": o[0],
                    "product": o[1],
                    "amount": float(o[2]),
                    "created_at": o[3]
                }
                for o in orders
            ]
        }

        # Insérer dans MongoDB
        mongo_db.users.insert_one(doc)

print("Migration terminée")
```

### Migration 4 : SQLite → PostgreSQL

**Difficulté :** 🟢 Facile

**Méthode recommandée : pgLoader**

```bash
# Commande directe
pgloader sqlite:///path/to/database.db postgresql://user:pass@localhost/dbname

# Ou via Docker
docker run --rm -v $(pwd):/data dimitri/pgloader \
    pgloader /data/database.db postgresql://user:pass@host/dbname
```

**Méthode alternative : Dump + Import**

```bash
# 1. Exporter SQLite en SQL
sqlite3 database.db .dump > sqlite_dump.sql

# 2. Quelques ajustements (optionnels)
# SQLite utilise des types flexibles, PostgreSQL plus stricts
sed -i 's/AUTOINCREMENT/SERIAL/g' sqlite_dump.sql

# 3. Créer la base PostgreSQL
psql -U postgres -c "CREATE DATABASE target_db;"

# 4. Importer
psql -U postgres -d target_db -f sqlite_dump.sql
```

### Migration 5 : MySQL → MongoDB

**Difficulté :** 🟡 Moyenne

**Script Node.js (avec Sequelize + Mongoose) :**

```javascript
const mysql = require('mysql2/promise');
const mongoose = require('mongoose');

// Connexion MySQL
const mysqlConn = await mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'source_db'
});

// Connexion MongoDB
await mongoose.connect('mongodb://localhost:27017/target_db');

// Schéma MongoDB
const UserSchema = new mongoose.Schema({
    name: String,
    email: String,
    created_at: Date
}, { _id: false, autoIndex: false });

const User = mongoose.model('User', UserSchema);

// Migration
const [rows] = await mysqlConn.execute('SELECT id, name, email, created_at FROM users');

for (const row of rows) {
    await User.create({
        _id: row.id,  // Utiliser l'ID MySQL comme _id MongoDB
        name: row.name,
        email: row.email,
        created_at: row.created_at
    });
}

console.log(`Migré ${rows.length} utilisateurs`);

// Fermeture
await mysqlConn.end();
await mongoose.disconnect();
```

---

## ✅ Validation Post-Migration

Après toute migration, il est **crucial** de valider l'intégrité des données.

### Checklist de Validation

#### 1. Comptage des Enregistrements

```sql
-- Source (MySQL)
SELECT COUNT(*) FROM users;

-- Cible (PostgreSQL)
SELECT COUNT(*) FROM users;

-- Les deux doivent être identiques !
```

#### 2. Vérification des Valeurs

```sql
-- Comparer des échantillons aléatoires
-- Source
SELECT * FROM users ORDER BY RAND() LIMIT 10;

-- Cible
SELECT * FROM users ORDER BY RANDOM() LIMIT 10;
```

#### 3. Vérification des Contraintes

```sql
-- Vérifier les clés primaires
SELECT COUNT(DISTINCT id) FROM users;  -- Doit être égal au COUNT(*) total

-- Vérifier les clés étrangères
SELECT COUNT(*) FROM orders WHERE user_id NOT IN (SELECT id FROM users);  -- Doit être 0

-- Vérifier les valeurs NULL
SELECT COUNT(*) FROM users WHERE email IS NULL;  -- Si email est NOT NULL
```

#### 4. Vérification des Types de Données

```sql
-- PostgreSQL : Vérifier les types
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'users';
```

#### 5. Tests Fonctionnels

```python
# Exemple : Tester des requêtes typiques de l'application
import psycopg2

conn = psycopg2.connect(...)
cursor = conn.cursor()

# Requête métier 1 : Obtenir un utilisateur par email
cursor.execute("SELECT * FROM users WHERE email = %s", ('alice@example.com',))
user = cursor.fetchone()
assert user is not None, "Utilisateur non trouvé après migration"

# Requête métier 2 : Jointure
cursor.execute("""
    SELECT u.name, COUNT(o.id)
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.name
""")
results = cursor.fetchall()
assert len(results) > 0, "Aucun résultat après jointure"

print("✅ Tous les tests passent")
```

### Script de Validation Automatisé

```python
import psycopg2
import pymysql

def validate_migration(mysql_config, pg_config):
    # Connexions
    mysql_conn = pymysql.connect(**mysql_config)
    pg_conn = psycopg2.connect(**pg_config)

    mysql_cursor = mysql_conn.cursor()
    pg_cursor = pg_conn.cursor()

    # Test 1 : Comptage
    mysql_cursor.execute("SELECT COUNT(*) FROM users")
    mysql_count = mysql_cursor.fetchone()[0]

    pg_cursor.execute("SELECT COUNT(*) FROM users")
    pg_count = pg_cursor.fetchone()[0]

    assert mysql_count == pg_count, f"Comptage différent : MySQL={mysql_count}, PG={pg_count}"
    print(f"✅ Comptage OK : {mysql_count} enregistrements")

    # Test 2 : Échantillons
    mysql_cursor.execute("SELECT id, name, email FROM users ORDER BY id LIMIT 100")
    mysql_sample = set(mysql_cursor.fetchall())

    pg_cursor.execute("SELECT id, name, email FROM users ORDER BY id LIMIT 100")
    pg_sample = set(pg_cursor.fetchall())

    assert mysql_sample == pg_sample, "Échantillons différents"
    print("✅ Échantillons identiques")

    # Test 3 : Clés étrangères
    pg_cursor.execute("""
        SELECT COUNT(*)
        FROM orders
        WHERE user_id NOT IN (SELECT id FROM users)
    """)
    orphaned = pg_cursor.fetchone()[0]
    assert orphaned == 0, f"{orphaned} enregistrements orphelins détectés"
    print("✅ Intégrité référentielle OK")

    print("\n🎉 Migration validée avec succès")

    # Fermeture
    mysql_conn.close()
    pg_conn.close()

# Utilisation
validate_migration(
    mysql_config={'host': 'localhost', 'user': 'root', 'password': 'pass', 'database': 'source_db'},
    pg_config={'host': 'localhost', 'user': 'postgres', 'password': 'pass', 'database': 'target_db'}
)
```

---

## 🎯 Stratégies de Migration

### 1. Big Bang (Tout d'un coup)

```
┌─────────────┐
│  Old DB     │───────┐
│  (Active)   │       │  Migration complète
└─────────────┘       │  (weekend, nuit)
                      ↓
                ┌─────────────┐
                │  New DB     │
                │  (Active)   │
                └─────────────┘
```

**Avantages :**
- ✅ Simple à planifier
- ✅ Migration unique
- ✅ Pas de code de compatibilité

**Inconvénients :**
- ❌ Downtime nécessaire
- ❌ Rollback difficile
- ❌ Risque élevé

**Quand utiliser :**
- Applications avec fenêtre de maintenance acceptable
- Migrations simples
- Petits volumes de données

### 2. Migration Progressive (Strangler Pattern)

```
                ┌────────────────┐
                │  Application   │
                └────────┬───────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ↓              ↓              ↓
    ┌─────────┐    ┌─────────┐    ┌─────────┐
    │ Old DB  │───→│ Sync    │───→│ New DB  │
    │ (Read)  │    │ Process │    │ (Write) │
    └─────────┘    └─────────┘    └─────────┘

    Phase 1: Dual write
    Phase 2: Dual read (new first)
    Phase 3: Old DB deprecated
```

**Avantages :**
- ✅ Pas de downtime
- ✅ Rollback facile
- ✅ Migration par étapes

**Inconvénients :**
- ❌ Complexe à implémenter
- ❌ Code de compatibilité nécessaire
- ❌ Sync bidirectionnelle difficile

**Quand utiliser :**
- Applications critiques (pas de downtime)
- Migrations complexes
- Gros volumes

### 3. Réplication Continue

```
┌─────────────┐
│  Old DB     │───────────┐
│  (Master)   │           │ Replication
└─────────────┘           │ en continu
                          ↓
                    ┌─────────────┐
                    │  New DB     │
                    │  (Replica)  │
                    └─────────────┘

    À J-day: Switch
```

**Outils :**
- MySQL → PostgreSQL : pgLoader en mode suivi
- Avec Debezium (Change Data Capture)
- Avec Kafka Connect

**Avantages :**
- ✅ Données toujours à jour
- ✅ Fenêtre de migration courte
- ✅ Rollback possible

**Inconvénients :**
- ❌ Nécessite outils spécifiques
- ❌ Configuration complexe

---

## ⚠️ Pièges Courants et Solutions

### 1. Encodage de Caractères

**Problème :**
```
MySQL (latin1) → PostgreSQL (UTF-8)
Résultat : � � � (caractères corrompus)
```

**Solution :**
```bash
# Vérifier l'encodage source
mysql -e "SHOW VARIABLES LIKE 'character_set%';"

# Convertir avant export
mysqldump --default-character-set=utf8mb4 ...

# Spécifier encodage dans PostgreSQL
psql -c "CREATE DATABASE target_db ENCODING 'UTF8';"
```

### 2. Pertes de Données

**Problème :**
```
Type source: BIGINT (jusqu'à 2^63)
Type cible: INT (jusqu'à 2^31)
Résultat : Overflow ou troncature
```

**Solution :**
```sql
-- Analyser les valeurs max avant migration
SELECT MAX(id) FROM big_table;  -- Si > 2^31, utiliser BIGINT

-- Adapter le schéma cible
CREATE TABLE big_table (
    id BIGINT PRIMARY KEY,  -- Pas INT
    ...
);
```

### 3. Contraintes Non Migrées

**Problème :**
```sql
-- MySQL : Contrainte implicite
email VARCHAR(255) UNIQUE

-- PostgreSQL : Pas d'index créé automatiquement
```

**Solution :**
```sql
-- Vérifier les index après migration
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Recréer si nécessaire
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

### 4. Performances Dégradées

**Problème :**
Requêtes lentes après migration

**Solutions :**
```sql
-- 1. Reconstruire les statistiques
ANALYZE;

-- 2. Reconstruire les index
REINDEX TABLE users;

-- 3. Vacuum (PostgreSQL)
VACUUM FULL users;

-- 4. Vérifier les plans de requêtes
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

### 5. Timezone et Dates

**Problème :**
```
MySQL DATETIME: 2024-01-15 10:30:00 (pas de timezone)
PostgreSQL TIMESTAMP: 2024-01-15 10:30:00+01:00 (avec timezone)
```

**Solution :**
```sql
-- Spécifier la timezone lors de la conversion
ALTER TABLE users ALTER COLUMN created_at
    TYPE TIMESTAMP WITH TIME ZONE
    USING created_at AT TIME ZONE 'Europe/Paris';
```

---

## 📋 Checklist Pré-Migration

Avant de démarrer une migration, vérifiez :

### Préparation

- [ ] **Backup complet** de la base source
- [ ] **Environnement de test** identique à production
- [ ] **Plan de rollback** documenté
- [ ] **Estimation du temps** de migration
- [ ] **Fenêtre de maintenance** réservée (si big bang)
- [ ] **Outils installés** et testés
- [ ] **Équipe disponible** (et formée)

### Analyse de la Source

- [ ] **Schéma documenté** (tables, relations, contraintes)
- [ ] **Volume de données** mesuré
- [ ] **Types de données** inventoriés
- [ ] **Triggers/procédures** listés
- [ ] **Index et contraintes** recensés
- [ ] **Requêtes critiques** identifiées
- [ ] **Utilisateurs et permissions** documentés

### Configuration de la Cible

- [ ] **Schéma cible** défini et validé
- [ ] **Capacité suffisante** (disque, RAM)
- [ ] **Réseau** configuré (si migration distante)
- [ ] **Monitoring** en place
- [ ] **Logs** activés

### Tests

- [ ] **Migration test** réussie (sur copie)
- [ ] **Validation** des données réussie
- [ ] **Tests de performance** OK
- [ ] **Tests fonctionnels** de l'application OK

---

## 🐳 Utiliser Docker pour Tester les Migrations

Docker est **parfait** pour tester des migrations sans risque !

### Environnement de Test Complet

```yaml
version: '3.8'

services:
  # Source : MySQL
  mysql_source:
    image: mysql:8
    container_name: migration_source
    environment:
      MYSQL_ROOT_PASSWORD: source_password
      MYSQL_DATABASE: source_db
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init_data.sql:/docker-entrypoint-initdb.d/init.sql  # Données de test

  # Cible : PostgreSQL
  postgres_target:
    image: postgres:15
    container_name: migration_target
    environment:
      POSTGRES_PASSWORD: target_password
      POSTGRES_DB: target_db
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

  # Outil de migration : pgLoader
  pgloader:
    image: dimitri/pgloader
    container_name: migration_tool
    depends_on:
      - mysql_source
      - postgres_target
    volumes:
      - ./migration.load:/migration.load
    command: pgloader /migration.load

volumes:
  mysql_data:
  pg_data:
```

**Fichier `init_data.sql` (données de test) :**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com'),
    ('Charlie', 'charlie@example.com');
```

**Utilisation :**
```bash
# 1. Démarrer l'environnement
docker-compose up -d

# 2. Attendre que tout soit prêt
sleep 10

# 3. Lancer la migration
docker-compose up pgloader

# 4. Vérifier dans PostgreSQL
docker exec -it migration_target psql -U postgres -d target_db -c "SELECT * FROM users;"

# 5. Nettoyer et recommencer si besoin
docker-compose down -v
```

**Avantages :**
- ✅ Reproductible à l'infini
- ✅ Isolé de votre système
- ✅ Facile à partager avec l'équipe
- ✅ Nettoyage propre

---

## 💡 Bonnes Pratiques

### 1. Toujours Tester

```
❌ Migration directe en production
✅ Test → Validation → Production
```

**Workflow recommandé :**
```
1. Backup source
2. Migration sur copie de test
3. Validation complète
4. Correction des problèmes détectés
5. Migration en production (avec plan B)
6. Validation post-migration
7. Monitoring intensif pendant 24-48h
```

### 2. Planifier le Rollback

**Avant la migration, préparez :**
- 📦 Backup complet récent
- 📝 Procédure de restauration documentée et testée
- ⏱️ Temps de rollback estimé
- 👥 Équipe disponible pour rollback d'urgence

**Conditions de rollback :**
```python
# Définir des seuils
if data_loss_percent > 1:
    trigger_rollback()

if performance_degradation > 50:
    trigger_rollback()

if critical_feature_broken:
    trigger_rollback()
```

### 3. Migration par Phases

Ne migrez pas tout d'un coup si possible.

**Exemple :**
```
Phase 1: Tables de référence (pays, catégories)
Phase 2: Données utilisateurs (non critiques)
Phase 3: Données transactionnelles
Phase 4: Données historiques (archives)
```

### 4. Monitoring et Logs

```bash
# Activer les logs détaillés
# PostgreSQL
ALTER SYSTEM SET log_min_duration_statement = 100;  # Log requêtes > 100ms

# MySQL
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
```

**Métriques à surveiller :**
- 📊 Latence des requêtes
- 💾 Utilisation disque
- 🔥 CPU et RAM
- 🔗 Nombre de connexions
- ⚠️ Erreurs applicatives

### 5. Documentation

Documentez **tout** :
```markdown
# Migration MySQL → PostgreSQL - 2024-11-15

## Contexte
- Raison : Fonctionnalités avancées de PostgreSQL nécessaires
- Base source : MySQL 8.0.35
- Base cible : PostgreSQL 15.4

## Environnement
- Source : mysql-prod-01 (192.168.1.10)
- Cible : postgres-prod-01 (192.168.1.20)

## Données
- Volume : 50 GB
- Tables : 45
- Enregistrements : ~10 millions

## Outils
- pgLoader 3.6.9
- Scripts Python personnalisés

## Durée
- Estimée : 2h
- Réelle : 1h45m

## Problèmes Rencontrés
1. Type DATETIME → TIMESTAMP (résolu en ajoutant timezone)
2. Clé étrangère circulaire (résolu en désactivant temporairement)

## Validation
- Comptage : OK (10,234,567 enregistrements)
- Tests fonctionnels : OK
- Performance : +15% plus rapide

## Équipe
- Lead : Alice (@alice)
- DBA : Bob (@bob)
- Dev : Charlie (@charlie)
```

---

## 📚 Ressources Complémentaires

### Outils Recommandés

| Outil | Usage | Lien |
|-------|-------|------|
| **pgLoader** | MySQL/SQLite → PostgreSQL | [pgloader.io](https://pgloader.io) |
| **DBeaver** | Interface graphique universelle | [dbeaver.io](https://dbeaver.io) |
| **Flyway** | Gestion de versions de schéma | [flywaydb.org](https://flywaydb.org) |
| **Liquibase** | Gestion de versions de schéma | [liquibase.org](https://liquibase.org) |
| **Airbyte** | Pipelines de données modernes | [airbyte.com](https://airbyte.com) |
| **Debezium** | Change Data Capture | [debezium.io](https://debezium.io) |

### Documentation Officielle

- [pgLoader Documentation](https://pgloader.readthedocs.io/)
- [PostgreSQL Migration Guide](https://www.postgresql.org/docs/current/migration.html)
- [MongoDB Migration Tools](https://docs.mongodb.com/database-tools/)
- [AWS Database Migration Service](https://aws.amazon.com/dms/)

### Articles et Tutoriels

- [Migrating from MySQL to PostgreSQL](https://wiki.postgresql.org/wiki/Converting_from_other_Databases_to_PostgreSQL)
- [MongoDB to SQL Migration Strategies](https://www.mongodb.com/blog/post/migration-strategies)

---

## ❓ FAQ - Questions Fréquentes

**Q : Combien de temps prend une migration ?**
R : Très variable. Comptez :
- Petite base (< 1 GB) : 1-2 heures
- Moyenne (1-10 GB) : 2-8 heures
- Grande (> 10 GB) : 1-3 jours

**Q : Faut-il arrêter l'application pendant la migration ?**
R : Dépend de la stratégie :
- Big Bang : Oui (downtime)
- Strangler Pattern : Non
- Réplication continue : Courte fenêtre de switch

**Q : Comment migrer sans perte de données ?**
R :
1. Backup complet avant
2. Test sur environnement identique
3. Validation systématique
4. Monitoring post-migration

**Q : Peut-on annuler une migration ?**
R : Oui, si vous avez :
- Un backup récent
- Un plan de rollback testé
- Détecté le problème rapidement

**Q : MongoDB vers SQL : faut-il normaliser ?**
R : Pas toujours ! Options :
- Normalisation complète (plusieurs tables)
- JSONB dans PostgreSQL (flexibilité)
- Hybride (tables + JSONB)

**Q : pgLoader fonctionne pour toutes les migrations SQL → PostgreSQL ?**
R : Quasiment. Supporte :
- MySQL → PostgreSQL (excellent)
- SQLite → PostgreSQL (excellent)
- MS SQL → PostgreSQL (bon)
- Fichiers CSV (excellent)

**Q : Comment gérer les très gros volumes (100+ GB) ?**
R :
- Migration par tables (parallélisation)
- Compression durant transfert
- Migration incrémentale
- Utiliser des outils spécialisés (AWS DMS, etc.)

**Q : Faut-il migrer les index ?**
R : Oui, mais :
- Migrer le schéma d'abord
- Désactiver les index durant import (vitesse)
- Recréer les index après import

**Q : Comment tester les performances après migration ?**
R : Comparer les métriques clés :
```sql
-- Temps d'exécution requête critique
EXPLAIN ANALYZE SELECT ...;

-- Avant vs Après
```

**Q : Que faire si des données sont corrompues après migration ?**
R :
1. Identifier l'étendue (combien d'enregistrements ?)
2. Comparer avec backup source
3. Migration ciblée des données corrompues
4. Ou rollback complet si trop de corruption

---

## ✅ Récapitulatif

Vous avez appris :

- ✅ **Les types de migrations** : homogènes vs hétérogènes
- ✅ **Le processus ETL** : Extract, Transform, Load
- ✅ **Les outils** : pgLoader, DBeaver, scripts personnalisés
- ✅ **Les migrations courantes** : MySQL→PostgreSQL, MongoDB→SQL, etc.
- ✅ **La validation** : Comptages, échantillons, tests
- ✅ **Les stratégies** : Big Bang, Strangler, Réplication
- ✅ **Les pièges** : Encodage, types, contraintes, performances
- ✅ **Les bonnes pratiques** : Tester, documenter, planifier le rollback
- ✅ **Docker pour tester** : Environnement reproductible

**Points clés à retenir :**
1. 🧪 **Toujours tester** avant production
2. 💾 **Backup obligatoire** avant toute migration
3. ✅ **Valider systématiquement** après migration
4. 📝 **Documenter** chaque étape
5. 🐳 **Utiliser Docker** pour tester sans risque

---

## 🚀 Aller Plus Loin

### Prochaines étapes suggérées

- **[Annexe A - Référence des commandes](../annexes/A-reference-commandes.md)** - Commandes Docker et SQL utiles
- **[Annexe C - Gestion des volumes](../annexes/C-gestion-volumes.md)** - Backups et restaurations
- **[Cas pratique 04 - Env dev complet](04-env-dev-complet.md)** - Tester plusieurs BDD ensemble

### Pour approfondir

- Apprendre SQL avancé (EXPLAIN, index, optimisation)
- Étudier les pipelines de données (Kafka, Airflow)
- Explorer le Change Data Capture (CDC) avec Debezium
- Se former sur les migrations Cloud (AWS DMS, Azure Database Migration)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
