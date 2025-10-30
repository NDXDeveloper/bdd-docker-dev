# 🐳 SQLite avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que SQLite ?

**SQLite** est une base de données **relationnelle** unique en son genre. Contrairement à MySQL, PostgreSQL ou MariaDB qui fonctionnent comme des **serveurs** séparés, SQLite est une base de données **embarquée** : toute votre base de données tient dans **un seul fichier** !

### 🎯 Caractéristiques Principales

| Caractéristique | SQLite | Autres BDD (MySQL, PostgreSQL...) |
|-----------------|--------|-----------------------------------|
| **Architecture** | Embarquée (pas de serveur) | Client-Serveur |
| **Fichier** | Un seul fichier `.db` | Multiples fichiers système |
| **Installation** | Rien à installer | Serveur à installer et configurer |
| **Configuration** | Aucune | Fichiers de configuration complexes |
| **Utilisateurs** | Pas de gestion d'utilisateurs | Système complet de permissions |
| **Réseau** | Accès local uniquement | Accès réseau natif |
| **Taille** | Bibliothèque de ~600 KB | Serveur de plusieurs centaines de MB |
| **Performances** | Excellentes en local | Excellentes en réseau |

### 💡 En d'autres termes...

**SQLite, c'est comme un fichier Excel intelligent :**
- 📄 Un seul fichier contient tout
- 💼 Vous pouvez le copier, l'envoyer par email, le mettre sur une clé USB
- 🔒 Les données sont stockées localement
- ⚡ Très rapide pour les petites et moyennes bases

**Les autres BDD, c'est comme Google Sheets :**
- 🌐 Un serveur distant stocke les données
- 👥 Plusieurs personnes peuvent y accéder en même temps
- 🔐 Système de permissions avancé
- 📡 Accessible via le réseau

---

## 🤔 Pourquoi Utiliser SQLite ?

### ✅ Avantages

#### 1. **Simplicité Extrême**

```bash
# Créer une base de données = créer un fichier
sqlite3 ma_base.db

# C'est tout ! Pas de serveur à démarrer, pas de configuration
```

#### 2. **Zéro Configuration**

Pas besoin de :
- ❌ Configurer un serveur
- ❌ Gérer des ports
- ❌ Créer des utilisateurs
- ❌ Définir des mots de passe
- ❌ Paramétrer la mémoire

**Vous créez un fichier, c'est prêt !** 🎉

#### 3. **Portabilité Totale**

```bash
# Copiez le fichier .db n'importe où
cp ma_base.db /chemin/de/destination/

# Ou envoyez-le par email, USB, cloud...
# Ça fonctionne sur Windows, macOS, Linux, Android, iOS
```

#### 4. **Fiabilité et Stabilité**

- 🏆 Base de données **la plus utilisée au monde** (présente dans tous les smartphones)
- 🔒 **ACID-compliant** (Atomicité, Cohérence, Isolation, Durabilité)
- ✅ Utilisée par des millions d'applications (navigateurs, smartphones, IoT...)
- 📚 Code source **public domain** (pas de licence à gérer)

#### 5. **Performances Excellentes en Local**

Pour des opérations locales, SQLite peut être **plus rapide** qu'un serveur MySQL ou PostgreSQL car :
- Pas de communication réseau
- Pas de surcharge serveur
- Accès direct au fichier

#### 6. **Idéal pour le Développement**

```python
# Python : Inclus par défaut !
import sqlite3
conn = sqlite3.connect('ma_base.db')

# Pas besoin d'installer de serveur pour tester votre code
```

### ❌ Limitations

#### 1. **Pas de Concurrence Massive**

```
❌ Mauvais choix pour :
- Sites web à fort trafic (> 100 000 requêtes/jour)
- Applications avec des centaines d'utilisateurs simultanés
- Systèmes avec beaucoup d'écritures simultanées

✅ Bon choix pour :
- Sites à faible trafic
- Applications bureautiques
- Outils internes
- Prototypes et MVP
```

#### 2. **Pas de Système d'Utilisateurs**

```sql
-- Ceci N'EXISTE PAS dans SQLite
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON ma_base.* TO 'admin'@'localhost';

-- Les permissions sont gérées au niveau du système d'exploitation
-- (permissions du fichier .db)
```

#### 3. **Pas d'Accès Réseau Natif**

```
❌ Impossible de faire :
mysql -h 192.168.1.10 -u user -p

✅ SQLite = Fichier local uniquement
Mais on peut contourner avec des outils web (voir fiche 4.3)
```

#### 4. **Taille de Base Limitée (en pratique)**

```
Limite théorique : 281 TB (téraoctets !)
Limite pratique recommandée : < 1 GB

Au-delà, les performances peuvent se dégrader
```

#### 5. **Pas de Fonctionnalités Avancées**

Absents de SQLite :
- ❌ Réplication native
- ❌ Clustering
- ❌ Procédures stockées complexes
- ❌ Partitionnement de tables
- ❌ Types de données très spécialisés

---

## 🎯 Quand Utiliser SQLite ?

### ✅ Cas d'Usage Idéaux

#### 1. **Développement et Prototypage**

```bash
# Testez rapidement votre modèle de données
# Sans installer de serveur de BDD
sqlite3 prototype.db

# Parfait pour :
- Apprendre le SQL
- Tester des requêtes
- Créer des MVP (Minimum Viable Product)
- Développer hors ligne
```

#### 2. **Applications Mobiles**

```java
// Android : SQLite est INCLUS dans le système
SQLiteDatabase db = openOrCreateDatabase("app.db", MODE_PRIVATE, null);

// iOS : Pareil, SQLite est natif
```

**Exemples d'apps utilisant SQLite :**
- 📱 WhatsApp (historique des messages)
- 🎵 Spotify (cache local)
- 📧 Gmail (emails en local)

#### 3. **Applications Desktop**

```
Applications locales :
- Gestionnaires de tâches
- Clients email
- Éditeurs de texte (historique)
- Outils de développement
- Gestionnaires de mots de passe
```

#### 4. **Sites Web à Faible Trafic**

```
✅ Blogs personnels
✅ Sites vitrine
✅ Portfolios
✅ Outils internes d'entreprise (< 50 utilisateurs)
✅ Applications intranet
```

#### 5. **IoT et Systèmes Embarqués**

```
Parfait pour :
- Capteurs IoT
- Raspberry Pi
- Drones
- Systèmes embarqués
- Équipements industriels

Raison : Faible consommation mémoire
```

#### 6. **Fichiers de Configuration Avancés**

```python
# Au lieu d'un fichier JSON/YAML complexe
# Utilisez SQLite pour des configs structurées

import sqlite3
config_db = sqlite3.connect('app_config.db')

# Avantages :
# - Requêtes SQL pour filtrer
# - Relations entre paramètres
# - Historique des modifications
```

#### 7. **Cache et Données Temporaires**

```javascript
// Au lieu de stocker en JSON
// SQLite offre des requêtes puissantes
const db = new sqlite3.Database(':memory:'); // En RAM !

// Parfait pour :
// - Cache d'API
// - Données de session
// - Résultats temporaires
```

### ❌ Cas où Éviter SQLite

#### 1. **Applications Web à Fort Trafic**

```
❌ E-commerce avec milliers de commandes/jour
❌ Réseaux sociaux
❌ Plateformes de streaming
❌ Applications bancaires

→ Utilisez PostgreSQL, MySQL, ou MongoDB
```

#### 2. **Besoins de Haute Disponibilité**

```
❌ Systèmes critiques 24/7
❌ Applications nécessitant du failover
❌ Réplication master-slave

→ Utilisez PostgreSQL avec réplication
```

#### 3. **Accès Multi-Utilisateurs Intensif**

```
❌ CRM avec 100+ utilisateurs simultanés
❌ ERP d'entreprise
❌ Systèmes collaboratifs en temps réel

→ Utilisez PostgreSQL ou MySQL
```

#### 4. **Très Grandes Bases de Données**

```
❌ Big Data (> 100 GB)
❌ Data Warehouses
❌ Analytics à grande échelle

→ Utilisez PostgreSQL, MongoDB, ou Cassandra
```

---

## 🏗️ Architecture de SQLite

### Structure d'un Fichier SQLite

```
ma_base.db (fichier unique)
│
├── Header (100 octets)
│   └── Métadonnées (version, taille page...)
│
├── Schema Table (structure des tables)
│   ├── sqlite_master (catalogue système)
│   ├── Définitions de tables
│   ├── Indexes
│   └── Triggers
│
├── Data Pages (données réelles)
│   ├── Table 1 : utilisateurs
│   ├── Table 2 : produits
│   └── Table 3 : commandes
│
└── Free Pages (espace disponible)
```

### Fonctionnement Interne

```
Application
    ↓
Bibliothèque SQLite (libsqlite3)
    ↓
SQL Compiler (analyse des requêtes)
    ↓
Virtual Machine (exécution)
    ↓
B-Tree (structure de stockage)
    ↓
Pager (gestion des pages)
    ↓
Système de Fichiers
    ↓
Fichier .db sur le disque
```

**Tout se passe dans le même processus que votre application !** Pas de serveur séparé.

---

## 🔍 SQLite vs Autres Bases de Données

### SQLite vs MySQL/MariaDB

| Critère | SQLite | MySQL/MariaDB |
|---------|--------|---------------|
| **Simplicité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Performance locale** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Performance réseau** | ❌ | ⭐⭐⭐⭐⭐ |
| **Multi-utilisateurs** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Sécurité** | ⭐⭐⭐ (fichier) | ⭐⭐⭐⭐⭐ (utilisateurs) |
| **Maintenance** | ⭐⭐⭐⭐⭐ (aucune) | ⭐⭐⭐ |
| **Portabilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

### SQLite vs PostgreSQL

| Critère | SQLite | PostgreSQL |
|---------|--------|------------|
| **Fonctionnalités SQL** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Types de données** | ⭐⭐⭐ (5 types de base) | ⭐⭐⭐⭐⭐ (types avancés) |
| **Extensions** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Transactions** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **JSON** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Facilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

### SQLite vs MongoDB

| Critère | SQLite | MongoDB |
|---------|--------|---------|
| **Modèle de données** | Relationnel (tables) | Document (JSON) |
| **Schéma** | Fixe (avec migrations) | Flexible |
| **Jointures** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Scalabilité** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Simplicité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Portabilité** | ⭐⭐⭐⭐⭐ (1 fichier) | ⭐⭐⭐ |

---

## 🐳 SQLite et Docker : Pourquoi ?

Vous pourriez vous demander : "Si SQLite est juste un fichier, pourquoi utiliser Docker ?"

### Raisons d'utiliser SQLite avec Docker

#### 1. **Outils Graphiques Sans Installation**

```yaml
# Interface web pour SQLite, sans rien installer
services:
  sqlite-web:
    image: coleifer/sqlite-web
    volumes:
      - ./data:/data
```

#### 2. **Version Spécifique de SQLite**

```bash
# Utiliser une version précise de SQLite
# Sans modifier celle de votre système
docker run -v ./data:/data alpine:3.18 sh
apk add sqlite  # Version contrôlée
```

#### 3. **Isolation et Tests**

```bash
# Tester des migrations sans risque
docker run --rm -v ./test.db:/data/test.db alpine sqlite3 /data/test.db
```

#### 4. **Environnement Reproductible**

```yaml
# Toute l'équipe a le même environnement
# Même version de SQLite, mêmes outils
version: '3.8'
services:
  sqlite:
    image: alpine:latest
    volumes:
      - ./data:/data
```

#### 5. **Faciliter les Backups**

```bash
# Backup automatisé avec un script dans Docker
docker exec sqlite_container sqlite3 /data/ma_base.db ".backup /data/backup.db"
```

---

## 📚 Fonctionnalités SQL de SQLite

### Types de Données (Simplifiés)

SQLite utilise un **système de typage dynamique** avec seulement **5 types de stockage** :

| Type SQLite | Équivalent SQL Standard | Exemples |
|-------------|-------------------------|----------|
| **NULL** | NULL | Valeur absente |
| **INTEGER** | INT, BIGINT, SMALLINT | 42, -1, 1000000 |
| **REAL** | FLOAT, DOUBLE | 3.14, -0.5, 2.718 |
| **TEXT** | VARCHAR, CHAR, TEXT | 'Bonjour', 'SQLite' |
| **BLOB** | BLOB, BYTEA | Images, fichiers binaires |

**Particularité :** SQLite est **flexible** avec les types :

```sql
CREATE TABLE exemple (
    id INTEGER,
    valeur TEXT
);

-- Vous pouvez insérer N'IMPORTE QUEL type dans N'IMPORTE QUELLE colonne
INSERT INTO exemple VALUES (1, 'texte');
INSERT INTO exemple VALUES ('texte', 42);  -- Fonctionne aussi !

-- SQLite convertit automatiquement si possible
```

### Fonctionnalités Supportées

#### ✅ Supporté par SQLite

```sql
-- Tables et contraintes
CREATE TABLE utilisateurs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nom TEXT NOT NULL,
    email TEXT UNIQUE,
    age INTEGER CHECK (age >= 18),
    date_inscription DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Index
CREATE INDEX idx_email ON utilisateurs(email);

-- Vues
CREATE VIEW utilisateurs_actifs AS
SELECT * FROM utilisateurs WHERE actif = 1;

-- Triggers
CREATE TRIGGER avant_suppression
BEFORE DELETE ON utilisateurs
BEGIN
    INSERT INTO audit (action, date) VALUES ('DELETE', datetime('now'));
END;

-- Transactions
BEGIN TRANSACTION;
UPDATE comptes SET solde = solde - 100 WHERE id = 1;
UPDATE comptes SET solde = solde + 100 WHERE id = 2;
COMMIT;

-- Jointures (toutes)
SELECT * FROM commandes c
INNER JOIN clients cl ON c.client_id = cl.id
LEFT JOIN produits p ON c.produit_id = p.id;

-- Sous-requêtes
SELECT nom FROM utilisateurs
WHERE id IN (SELECT DISTINCT user_id FROM commandes);

-- Common Table Expressions (CTE)
WITH ventes_mensuelles AS (
    SELECT strftime('%Y-%m', date) AS mois, SUM(montant) AS total
    FROM ventes
    GROUP BY mois
)
SELECT * FROM ventes_mensuelles WHERE total > 10000;

-- JSON (depuis SQLite 3.38+)
SELECT json_extract(data, '$.nom') AS nom
FROM utilisateurs_json;

-- Full Text Search (FTS5)
CREATE VIRTUAL TABLE articles_fts USING fts5(titre, contenu);
SELECT * FROM articles_fts WHERE articles_fts MATCH 'docker sqlite';
```

#### ❌ Non Supporté par SQLite

```sql
-- Procédures stockées
❌ CREATE PROCEDURE ma_procedure() AS ...

-- RIGHT JOIN / FULL OUTER JOIN
❌ SELECT * FROM t1 RIGHT JOIN t2 ON ...

-- Modifications de colonnes (ALTER TABLE limité)
❌ ALTER TABLE users MODIFY COLUMN age BIGINT;
-- → Il faut recréer la table

-- Utilisateurs et permissions
❌ GRANT SELECT ON table TO user;

-- Réplication
❌ Pas de master-slave natif
```

---

## 📖 Ressources pour Apprendre SQLite

### Documentation Officielle

- 🌐 [Site officiel SQLite](https://www.sqlite.org/)
- 📚 [Documentation complète](https://www.sqlite.org/docs.html)
- 🎓 [Tutoriel SQL SQLite](https://www.sqlite.org/lang.html)

### Tutoriels en Français

- 📹 [SQLite Tutorial (YouTube)](https://www.youtube.com/results?search_query=sqlite+tutoriel+français)
- 📖 [OpenClassrooms - Administrez vos bases de données](https://openclassrooms.com/)
- 📝 [SQLite sur Developpez.com](https://sqlite.developpez.com/)

### Outils de Test en Ligne

- 🌐 [SQLite Online](https://sqliteonline.com/)
- 🧪 [DB Fiddle (SQLite)](https://www.db-fiddle.com/)
- 💻 [SQLime - SQLite Playground](https://sqlime.org/)

---

## 🎯 Prochaines Étapes

Maintenant que vous comprenez ce qu'est SQLite, passons à la pratique !

### Fiches Suivantes

1. **[4.1 - Utilisation de SQLite en conteneur](01-sqlite-conteneur.md)**
   - Lancer SQLite avec Docker
   - Créer vos premières bases
   - Commandes essentielles

2. **[4.2 - SQLite avec volume persistant](02-sqlite-volume-persistant.md)**
   - Sauvegarder vos données
   - Gérer plusieurs bases
   - Backups automatisés

3. **[4.3 - Outils graphiques](03-outils-graphiques.md)**
   - DB Browser for SQLite
   - Interfaces web (sqlite-web, Adminer)
   - DBeaver et autres outils

---

## ✅ Récapitulatif

| Point Clé | Description |
|-----------|-------------|
| 🗄️ **Nature** | Base de données **embarquée** dans un fichier unique |
| ⚡ **Simplicité** | Aucune configuration, zéro maintenance |
| 📦 **Portabilité** | Un fichier `.db` = toute votre base |
| 🚀 **Performance** | Excellente en local, limitée en réseau |
| 👥 **Multi-utilisateurs** | Limité (lecture OK, écriture séquentielle) |
| 💼 **Usage idéal** | Dev, mobile, desktop, petits sites, IoT |
| 🏢 **À éviter pour** | Gros sites web, haute concurrence, big data |
| 🐳 **Avec Docker** | Isolation, outils graphiques, reproductibilité |

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
