# 7.4 Gestion des keyspaces et tables

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans Cassandra, la modélisation des données est **fondamentalement différente** des bases de données relationnelles (SQL). Cette fiche vous guide dans la création et la gestion des **keyspaces** (équivalent de bases de données) et des **tables**, en comprenant les concepts spécifiques à Cassandra.

### Concepts clés

| Concept Cassandra | Équivalent SQL | Description |
|-------------------|----------------|-------------|
| **Keyspace** | Base de données | Conteneur principal |
| **Table** | Table | Structure de données |
| **Partition Key** | Clé primaire (partie) | Détermine la distribution des données |
| **Clustering Key** | Clé de tri | Ordonne les données dans une partition |
| **Column** | Colonne | Champ de données |

**Principe fondamental :** En Cassandra, vous modélisez **selon vos requêtes**, pas selon vos entités.

---

## 🎯 Ce que vous allez apprendre

À la fin de cette fiche, vous saurez :
- ✅ Créer des keyspaces avec différentes stratégies de réplication
- ✅ Comprendre et choisir le bon Replication Factor
- ✅ Créer des tables avec clés primaires simples et composées
- ✅ Utiliser les Partition Keys et Clustering Keys
- ✅ Modifier la structure des tables (ALTER TABLE)
- ✅ Gérer les types de données Cassandra
- ✅ Comprendre les index secondaires
- ✅ Appliquer les bonnes pratiques de modélisation

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :

- Un cluster Cassandra opérationnel (voir [7.1](01-config-noeud-unique.md) ou [7.3](03-cluster-simple.md))
- Accès à `cqlsh`
- Avoir compris les concepts de base de Cassandra

**Connexion rapide :**
```bash
docker exec -it cassandra-seed cqlsh
```

---

## 🗄️ Partie 1 : Gestion des Keyspaces

### Qu'est-ce qu'un keyspace ?

Un **keyspace** est le conteneur de plus haut niveau dans Cassandra. Il définit :
- La **stratégie de réplication** des données
- Le **nombre de réplicas** (Replication Factor)
- La **topologie** du cluster (datacenters)

### 1.1 Créer un keyspace simple

**Pour un nœud unique ou environnement de développement :**

```sql
CREATE KEYSPACE IF NOT EXISTS app_dev
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

**Explication :**

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `IF NOT EXISTS` | - | Évite une erreur si le keyspace existe |
| `class` | `SimpleStrategy` | Stratégie de réplication basique |
| `replication_factor` | `1` | Une seule copie (développement uniquement) |

⚠️ **Attention :** `SimpleStrategy` avec RF=1 = aucune tolérance de panne !

### 1.2 Créer un keyspace pour cluster (production)

**Pour un cluster de 3 nœuds ou plus :**

```sql
CREATE KEYSPACE IF NOT EXISTS app_prod
WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 3
};
```

**Explication :**

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `class` | `NetworkTopologyStrategy` | Stratégie multi-datacenter (recommandée) |
| `datacenter1` | `3` | 3 réplicas dans le datacenter1 |

**Pourquoi NetworkTopologyStrategy ?**
- Plus flexible (prêt pour multi-datacenter)
- Recommandé même pour un seul datacenter
- Utilisé en production

### 1.3 Créer un keyspace multi-datacenter

**Pour un cluster réparti géographiquement :**

```sql
CREATE KEYSPACE IF NOT EXISTS app_global
WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 3,
  'datacenter2': 3,
  'datacenter3': 2
};
```

**Explication :**
- `datacenter1` : 3 réplicas (ex: Europe)
- `datacenter2` : 3 réplicas (ex: USA)
- `datacenter3` : 2 réplicas (ex: Asie)

**Total : 8 copies des données** (haute disponibilité mondiale).

### 1.4 Choisir le bon Replication Factor

| Environnement | RF recommandé | Raison |
|---------------|---------------|--------|
| **Développement local** | 1 | Économise les ressources |
| **Test/Staging** | 2 | Test de réplication basique |
| **Production (petite)** | 3 | Tolérance à 2 pannes |
| **Production (critique)** | 5 | Tolérance à 4 pannes |

**Formule :** Pour tolérer **N pannes**, utilisez RF = **N + 1**.

### 1.5 Lister les keyspaces

```sql
DESCRIBE KEYSPACES;
```

**Résultat :**
```
system             system_distributed  system_traces
system_auth        system_schema       system_views
system_virtual_schema  app_dev         app_prod
```

### 1.6 Voir la configuration d'un keyspace

```sql
DESCRIBE KEYSPACE app_prod;
```

**Résultat :**
```sql
CREATE KEYSPACE app_prod WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': '3'
} AND durable_writes = true;
```

### 1.7 Modifier un keyspace

**Changer le Replication Factor :**

```sql
ALTER KEYSPACE app_prod
WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 5
};
```

⚠️ **Important :** Après modification, exécutez `nodetool repair` sur tous les nœuds.

### 1.8 Supprimer un keyspace

⚠️ **ATTENTION : Supprime TOUTES les données !**

```sql
DROP KEYSPACE IF EXISTS app_dev;
```

---

## 📊 Partie 2 : Gestion des Tables

### 2.1 Types de clés primaires

Cassandra utilise deux types de clés dans une clé primaire :

#### A. Clé primaire simple (Partition Key uniquement)

```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT
);
```

**Structure :**
```
PRIMARY KEY (user_id)
           └─ Partition Key
```

**Utilisation :** Accès direct par `user_id`.

#### B. Clé primaire composée (Partition Key + Clustering Key)

```sql
CREATE TABLE posts (
  user_id UUID,
  post_id TIMEUUID,
  title TEXT,
  content TEXT,
  PRIMARY KEY (user_id, post_id)
);
```

**Structure :**
```
PRIMARY KEY (user_id, post_id)
             └─ Partition Key
                      └─ Clustering Key
```

**Comportement :**
- Les posts sont **partitionnés** par `user_id`
- Dans chaque partition, ils sont **triés** par `post_id`
- Tous les posts d'un utilisateur sont sur le même nœud

#### C. Partition Key composite (plusieurs colonnes)

```sql
CREATE TABLE sessions (
  app_name TEXT,
  user_id UUID,
  session_id TIMEUUID,
  last_activity TIMESTAMP,
  PRIMARY KEY ((app_name, user_id), session_id)
);
```

**Structure :**
```
PRIMARY KEY ((app_name, user_id), session_id)
             └──────────┬─────────┘     └─ Clustering Key
                   Partition Key
```

**Comportement :**
- Les sessions sont partitionnées par **combinaison** de `app_name` ET `user_id`
- Meilleure distribution des données

### 2.2 Créer des tables : Exemples pratiques

#### Exemple 1 : Table simple d'utilisateurs

```sql
USE app_prod;

CREATE TABLE IF NOT EXISTS users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT,
  first_name TEXT,
  last_name TEXT,
  created_at TIMESTAMP,
  last_login TIMESTAMP
);
```

**Requêtes possibles :**
```sql
-- ✅ Efficace (requête par partition key)
SELECT * FROM users WHERE user_id = 123e4567-e89b-12d3-a456-426614174000;

-- ❌ Inefficace (scan complet de la table)
SELECT * FROM users WHERE username = 'alice';
```

#### Exemple 2 : Table de messages (modèle par requête)

**Objectif :** Récupérer rapidement tous les messages d'un utilisateur, triés par date.

```sql
CREATE TABLE IF NOT EXISTS messages_by_user (
  user_id UUID,
  message_id TIMEUUID,
  sender_id UUID,
  content TEXT,
  sent_at TIMESTAMP,
  is_read BOOLEAN,
  PRIMARY KEY (user_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

**Points clés :**
- `user_id` : Partition Key (tous les messages d'un user sur le même nœud)
- `message_id` : Clustering Key (tri des messages)
- `DESC` : Ordre décroissant (messages récents en premier)

**Requêtes efficaces :**
```sql
-- ✅ Tous les messages d'un user
SELECT * FROM messages_by_user WHERE user_id = <uuid>;

-- ✅ Les 10 derniers messages
SELECT * FROM messages_by_user WHERE user_id = <uuid> LIMIT 10;

-- ✅ Messages dans une période
SELECT * FROM messages_by_user
WHERE user_id = <uuid>
AND message_id > minTimeuuid('2024-01-01')
AND message_id < maxTimeuuid('2024-01-31');
```

#### Exemple 3 : Table de compteurs

```sql
CREATE TABLE IF NOT EXISTS page_views (
  page_url TEXT,
  date DATE,
  view_count COUNTER,
  PRIMARY KEY (page_url, date)
);
```

**Utilisation des compteurs :**
```sql
-- Incrémenter
UPDATE page_views SET view_count = view_count + 1
WHERE page_url = '/home' AND date = '2024-10-31';

-- Décrémenter
UPDATE page_views SET view_count = view_count - 5
WHERE page_url = '/home' AND date = '2024-10-31';

-- Lire
SELECT view_count FROM page_views
WHERE page_url = '/home' AND date = '2024-10-31';
```

⚠️ **Limitation :** Les colonnes COUNTER ne peuvent pas être mélangées avec d'autres types de colonnes.

### 2.3 Types de données Cassandra

| Type | Description | Exemple |
|------|-------------|---------|
| `TEXT` | Chaîne de caractères | `'Alice'` |
| `VARCHAR` | Alias de TEXT | `'Bob'` |
| `INT` | Entier 32 bits | `42` |
| `BIGINT` | Entier 64 bits | `9223372036854775807` |
| `FLOAT` | Nombre flottant 32 bits | `3.14` |
| `DOUBLE` | Nombre flottant 64 bits | `3.14159265359` |
| `DECIMAL` | Nombre décimal précis | `19.99` |
| `BOOLEAN` | Vrai/Faux | `true`, `false` |
| `UUID` | Identifiant universel | `123e4567-e89b-12d3-...` |
| `TIMEUUID` | UUID basé sur le temps | `50554d6e-29bb-11e5-...` |
| `TIMESTAMP` | Date et heure | `2024-10-31 10:30:00+0000` |
| `DATE` | Date uniquement | `2024-10-31` |
| `TIME` | Heure uniquement | `10:30:00` |
| `BLOB` | Données binaires | `0x48656c6c6f` |
| `INET` | Adresse IP | `192.168.1.1` |
| `COUNTER` | Compteur distribué | `42` |

#### Types de collections

```sql
CREATE TABLE users_with_collections (
  user_id UUID PRIMARY KEY,
  emails SET<TEXT>,           -- Ensemble (pas de doublons)
  phone_numbers LIST<TEXT>,   -- Liste (ordre préservé)
  preferences MAP<TEXT, TEXT> -- Dictionnaire clé-valeur
);
```

**Insertion :**
```sql
INSERT INTO users_with_collections (user_id, emails, phone_numbers, preferences)
VALUES (
  uuid(),
  {'alice@example.com', 'alice@work.com'},
  ['+33123456789', '+33987654321'],
  {'theme': 'dark', 'language': 'fr'}
);
```

**Mise à jour :**
```sql
-- Ajouter à un SET
UPDATE users_with_collections
SET emails = emails + {'alice@personal.com'}
WHERE user_id = <uuid>;

-- Ajouter à une LIST
UPDATE users_with_collections
SET phone_numbers = phone_numbers + ['+33111222333']
WHERE user_id = <uuid>;

-- Mettre à jour une MAP
UPDATE users_with_collections
SET preferences['notifications'] = 'enabled'
WHERE user_id = <uuid>;
```

### 2.4 Modifier la structure d'une table

#### Ajouter une colonne

```sql
ALTER TABLE users ADD middle_name TEXT;
```

#### Renommer une colonne

```sql
ALTER TABLE users RENAME email TO email_address;
```

⚠️ **Limitation :** Ne fonctionne que pour les colonnes non-clés.

#### Supprimer une colonne

```sql
ALTER TABLE users DROP middle_name;
```

⚠️ **Attention :** Les données de cette colonne seront perdues.

#### Modifier le type d'une colonne

**Impossible !** Cassandra ne permet pas de changer le type d'une colonne existante.

**Solution :** Créer une nouvelle colonne et migrer les données.

### 2.5 Lister les tables

```sql
USE app_prod;
DESCRIBE TABLES;
```

**Résultat :**
```
users  messages_by_user  page_views
```

### 2.6 Voir la structure d'une table

```sql
DESCRIBE TABLE users;
```

**Résultat :**
```sql
CREATE TABLE app_prod.users (
    user_id uuid PRIMARY KEY,
    created_at timestamp,
    email text,
    first_name text,
    last_name text,
    last_login timestamp,
    username text
) WITH bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy'}
    AND compression = {'chunk_length_in_kb': '16', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND min_index_interval = 128
    AND speculative_retry = '99PERCENTILE';
```

### 2.7 Options de création de table

#### Time To Live (TTL)

**Au niveau de la table :**
```sql
CREATE TABLE sessions (
  session_id UUID PRIMARY KEY,
  user_id UUID,
  data TEXT
) WITH default_time_to_live = 3600;  -- 1 heure
```

**Au niveau de l'insertion :**
```sql
INSERT INTO sessions (session_id, user_id, data)
VALUES (uuid(), <user_uuid>, 'session_data')
USING TTL 7200;  -- 2 heures
```

**Vérifier le TTL restant :**
```sql
SELECT session_id, TTL(data) FROM sessions;
```

#### Ordre de tri (Clustering Order)

```sql
CREATE TABLE posts (
  user_id UUID,
  post_id TIMEUUID,
  title TEXT,
  PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

- `ASC` : Ordre croissant (par défaut)
- `DESC` : Ordre décroissant

#### Compactage (Compaction Strategy)

```sql
CREATE TABLE logs (
  log_id TIMEUUID PRIMARY KEY,
  message TEXT
) WITH compaction = {
  'class': 'TimeWindowCompactionStrategy',
  'compaction_window_unit': 'DAYS',
  'compaction_window_size': 1
};
```

**Stratégies de compactage :**
- `SizeTieredCompactionStrategy` : Par défaut, général
- `LeveledCompactionStrategy` : Pour lectures fréquentes
- `TimeWindowCompactionStrategy` : Pour données temporelles

#### Compression

```sql
CREATE TABLE images (
  image_id UUID PRIMARY KEY,
  data BLOB
) WITH compression = {
  'class': 'org.apache.cassandra.io.compress.LZ4Compressor',
  'chunk_length_in_kb': 64
};
```

### 2.8 Supprimer une table

⚠️ **ATTENTION : Supprime TOUTES les données de la table !**

```sql
DROP TABLE IF EXISTS users;
```

---

## 🔍 Partie 3 : Index secondaires

### Qu'est-ce qu'un index secondaire ?

Un **index secondaire** permet de requêter une table par une colonne qui n'est pas la clé primaire.

⚠️ **Attention :** À utiliser avec parcimonie ! Performances limitées.

### Créer un index secondaire

```sql
CREATE INDEX IF NOT EXISTS users_username_idx
ON users (username);
```

**Maintenant, cette requête est possible :**
```sql
SELECT * FROM users WHERE username = 'alice';
```

### Quand utiliser les index secondaires ?

| ✅ Bon usage | ❌ Mauvais usage |
|-------------|-----------------|
| Faible cardinalité (peu de valeurs distinctes) | Haute cardinalité (beaucoup de valeurs uniques) |
| Requêtes occasionnelles | Requêtes fréquentes |
| Tables de petite taille | Tables massives |

**Exemple bon usage :** Index sur `country` (200 pays).
**Exemple mauvais usage :** Index sur `email` (millions de valeurs uniques).

### Alternative : Modélisation par requête

**Au lieu d'un index secondaire, créez une table dédiée :**

```sql
-- Table principale
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT
);

-- Table pour recherche par username
CREATE TABLE users_by_username (
  username TEXT PRIMARY KEY,
  user_id UUID
);
```

**Lors de l'insertion :**
```sql
-- Insérer dans les deux tables (batch pour atomicité)
BEGIN BATCH
  INSERT INTO users (user_id, username, email)
  VALUES (<uuid>, 'alice', 'alice@example.com');

  INSERT INTO users_by_username (username, user_id)
  VALUES ('alice', <uuid>);
APPLY BATCH;
```

### Lister les index

```sql
DESCRIBE INDEX users_username_idx;
```

### Supprimer un index

```sql
DROP INDEX IF EXISTS users_username_idx;
```

---

## 🎨 Partie 4 : Bonnes pratiques de modélisation

### Règle d'or : Modéliser selon les requêtes

**En SQL :** Normaliser les données, puis écrire les requêtes.
**En Cassandra :** Écrire les requêtes, puis créer les tables.

### Principe 1 : Dénormalisation

**Dupliquer les données** pour optimiser les lectures.

**Exemple :**

```sql
-- Table 1 : Posts par utilisateur
CREATE TABLE posts_by_user (
  user_id UUID,
  post_id TIMEUUID,
  title TEXT,
  content TEXT,
  PRIMARY KEY (user_id, post_id)
);

-- Table 2 : Posts par tag
CREATE TABLE posts_by_tag (
  tag TEXT,
  post_id TIMEUUID,
  user_id UUID,
  title TEXT,
  content TEXT,
  PRIMARY KEY (tag, post_id)
);
```

**Résultat :** Les mêmes posts sont stockés dans **deux tables** différentes.

### Principe 2 : Une table par pattern de requête

**Requêtes souhaitées :**
- Récupérer tous les posts d'un user
- Récupérer tous les posts avec un tag

**Solution :** Deux tables dédiées (ci-dessus).

### Principe 3 : Limiter la taille des partitions

**Problème :** Une partition trop grande (> 100 MB) dégrade les performances.

**Solution :** Utiliser un "bucketing" :

```sql
-- Mauvais : tous les posts d'un user dans une partition
CREATE TABLE posts_by_user (
  user_id UUID,
  post_id TIMEUUID,
  title TEXT,
  PRIMARY KEY (user_id, post_id)
);

-- Bon : posts groupés par mois
CREATE TABLE posts_by_user_monthly (
  user_id UUID,
  year_month TEXT,  -- '2024-10'
  post_id TIMEUUID,
  title TEXT,
  PRIMARY KEY ((user_id, year_month), post_id)
);
```

### Principe 4 : Éviter ALLOW FILTERING

**Mauvaise pratique :**
```sql
SELECT * FROM users WHERE last_name = 'Dupont' ALLOW FILTERING;
```

**Pourquoi ?** Scan complet de la table (très lent).

**Solution :** Créer une table dédiée ou un index secondaire (si approprié).

### Principe 5 : Utiliser les bons types de clés

| Type de clé | Usage |
|-------------|-------|
| `UUID` | Identifiants généraux |
| `TIMEUUID` | Identifiants avec tri temporel |
| `TEXT` | Clés naturelles (username, email) |

---

## 📐 Partie 5 : Exemples de modélisation

### Exemple 1 : Application de messagerie

**Requêtes :**
1. Récupérer toutes les conversations d'un user
2. Récupérer tous les messages d'une conversation

**Modèle :**

```sql
-- Table 1 : Conversations par user
CREATE TABLE conversations_by_user (
  user_id UUID,
  conversation_id UUID,
  other_user_id UUID,
  last_message_at TIMESTAMP,
  PRIMARY KEY (user_id, conversation_id)
) WITH CLUSTERING ORDER BY (conversation_id DESC);

-- Table 2 : Messages par conversation
CREATE TABLE messages_by_conversation (
  conversation_id UUID,
  message_id TIMEUUID,
  sender_id UUID,
  content TEXT,
  sent_at TIMESTAMP,
  PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

### Exemple 2 : Système de logs

**Requêtes :**
1. Récupérer les logs d'un serveur sur une période
2. Récupérer les logs par niveau de sévérité

**Modèle :**

```sql
-- Table 1 : Logs par serveur et heure
CREATE TABLE logs_by_server (
  server_id TEXT,
  hour TIMESTAMP,
  log_id TIMEUUID,
  level TEXT,
  message TEXT,
  PRIMARY KEY ((server_id, hour), log_id)
) WITH CLUSTERING ORDER BY (log_id DESC);

-- Table 2 : Logs par niveau
CREATE TABLE logs_by_level (
  level TEXT,
  hour TIMESTAMP,
  log_id TIMEUUID,
  server_id TEXT,
  message TEXT,
  PRIMARY KEY ((level, hour), log_id)
) WITH CLUSTERING ORDER BY (log_id DESC);
```

### Exemple 3 : Réseau social (timeline)

**Requêtes :**
1. Récupérer le feed d'un user (posts de ses amis)

**Modèle :**

```sql
-- Table : Timeline par user
CREATE TABLE timeline (
  user_id UUID,
  post_id TIMEUUID,
  author_id UUID,
  content TEXT,
  posted_at TIMESTAMP,
  PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

**Logique applicative :**
- Lors de la création d'un post, l'insérer dans la timeline de tous les followers
- Dénormalisation extrême, mais lectures ultra-rapides

---

## 🛠️ Commandes utiles

### Vider une table (supprimer toutes les données)

```sql
TRUNCATE users;
```

### Exporter une table en CSV

```bash
docker exec -it cassandra-seed cqlsh -e "COPY app_prod.users TO '/tmp/users.csv' WITH HEADER = true;"
docker cp cassandra-seed:/tmp/users.csv ./users.csv
```

### Importer des données depuis CSV

```sql
COPY users (user_id, username, email)
FROM '/path/to/users.csv'
WITH HEADER = true;
```

### Compter les lignes d'une table

```sql
SELECT COUNT(*) FROM users;
```

⚠️ **Attention :** Lent sur de grandes tables (scan complet).

### Voir la taille d'une table

```bash
docker exec -it cassandra-seed nodetool tablestats app_prod.users
```

---

## 🚨 Erreurs courantes

### Erreur 1 : "Cannot execute this query as it might involve data filtering"

**Cause :** Requête sans clé primaire et sans `ALLOW FILTERING`.

**Solution :** Ajouter `ALLOW FILTERING` (déconseillé) ou remodéliser.

### Erreur 2 : "Partition key parts: ... must be restricted"

**Cause :** Requête sur Clustering Key sans spécifier la Partition Key.

**Exemple incorrect :**
```sql
SELECT * FROM messages_by_user WHERE message_id = <timeuuid>;
```

**Solution :** Toujours spécifier la Partition Key :
```sql
SELECT * FROM messages_by_user
WHERE user_id = <uuid> AND message_id = <timeuuid>;
```

### Erreur 3 : "Undefined column name"

**Cause :** La colonne n'existe pas ou faute de frappe.

**Vérification :**
```sql
DESCRIBE TABLE users;
```

---

## 💡 Conseils pour débutants

### 1. Commencez simple

Créez d'abord un keyspace avec RF=1 et des tables basiques.

### 2. Dessinez votre modèle

Avant de coder, dessinez :
- Vos requêtes
- Vos tables
- Vos clés primaires

### 3. Testez avec des données

Insérez des données de test pour valider votre modèle.

### 4. Documentez vos keyspaces

Créez un fichier `SCHEMA.md` :
```markdown
# Schéma de données

## Keyspace: app_prod
- RF: 3
- Stratégie: NetworkTopologyStrategy

### Table: users
- Description: Utilisateurs de l'application
- Partition Key: user_id
- Requêtes: Accès par ID

### Table: messages_by_user
- Description: Messages par utilisateur
- Partition Key: user_id
- Clustering Key: message_id
- Requêtes: Tous les messages d'un user
```

### 5. Utilisez des noms clairs

**Mauvais :** `data`, `info`, `tmp`
**Bon :** `user_profiles`, `order_history`, `session_tokens`

---

## 📚 Récapitulatif

| Concept | Description | Commande |
|---------|-------------|----------|
| **Keyspace** | Conteneur de tables | `CREATE KEYSPACE` |
| **Replication Factor** | Nombre de copies | `'replication_factor': 3` |
| **Table** | Structure de données | `CREATE TABLE` |
| **Partition Key** | Distribution des données | `PRIMARY KEY (partition_key)` |
| **Clustering Key** | Tri dans une partition | `PRIMARY KEY (pk, ck)` |
| **Index secondaire** | Requête sur colonne non-clé | `CREATE INDEX` |
| **TTL** | Expiration automatique | `WITH default_time_to_live` |

---

## ❓ Questions fréquentes (FAQ)

**Q : Quelle est la différence entre SimpleStrategy et NetworkTopologyStrategy ?**
R : `SimpleStrategy` = basique, un seul datacenter. `NetworkTopologyStrategy` = recommandé, multi-datacenter.

**Q : Puis-je ajouter une colonne à une clé primaire existante ?**
R : Non. Vous devez créer une nouvelle table et migrer les données.

**Q : Combien de tables puis-je créer dans un keyspace ?**
R : Techniquement illimité, mais limité par les ressources. Recommandation : < 200 tables/keyspace.

**Q : Dois-je créer un index secondaire ou une table dédiée ?**
R : Préférez une table dédiée pour les requêtes fréquentes. Index secondaire pour les requêtes occasionnelles.

**Q : Comment gérer les relations "one-to-many" ?**
R : Utilisez une Partition Key + Clustering Key. Exemple : `PRIMARY KEY (user_id, order_id)`.

**Q : Cassandra supporte-t-il les JOIN ?**
R : Non. Dénormalisez vos données et créez des tables par pattern de requête.

---

## 🔗 Ressources complémentaires

- [DataStax - Data Modeling](https://www.datastax.com/learn/data-modeling-by-example)
- [Cassandra Data Modeling Best Practices](https://cassandra.apache.org/doc/latest/cassandra/data_modeling/)
- [CQL Reference](https://cassandra.apache.org/doc/latest/cassandra/cql/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
