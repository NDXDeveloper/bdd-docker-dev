# 7.1 Configuration basique d'un nœud unique Cassandra

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction à Cassandra

**Apache Cassandra** est une base de données NoSQL distribuée, conçue pour gérer de très grandes quantités de données sur plusieurs serveurs, sans point de défaillance unique. C'est une base de données de type **colonnes larges** (wide-column store).

**Cas d'usage typiques :**
- Applications nécessitant une **haute disponibilité**
- Gestion de **gros volumes de données** (Big Data)
- Applications IoT avec beaucoup d'écritures
- Systèmes de messagerie, logs, données de séries temporelles

**Pourquoi commencer avec Docker ?**
- Installation simple et rapide
- Environnement isolé
- Idéal pour le développement et l'apprentissage
- Pas de pollution de votre système

⚠️ **Note importante :** Cette fiche couvre un **nœud unique** pour le développement. En production, Cassandra est utilisé en cluster (plusieurs nœuds).

---

## 🎯 Ce que vous allez apprendre

À la fin de cette fiche, vous saurez :
- ✅ Démarrer un conteneur Cassandra avec Docker Compose
- ✅ Vous connecter à Cassandra avec `cqlsh` (le shell CQL)
- ✅ Créer votre premier keyspace (équivalent d'une base de données)
- ✅ Créer une table et insérer des données
- ✅ Exécuter des requêtes CQL basiques
- ✅ Arrêter et supprimer proprement le conteneur

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :

- **Docker** installé (version 20.10+)
- **Docker Compose** installé (version 2.0+)

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

Si vous n'avez pas Docker, consultez le fichier [01-prerequis.md](/00-introduction/01-prerequis.md).

---

## 🚀 Étape 1 : Création du fichier `docker-compose.yml`

Créez un nouveau dossier pour votre projet Cassandra :

```bash
mkdir cassandra-dev
cd cassandra-dev
```

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  cassandra:
    # Image officielle Cassandra (version 4.1)
    image: cassandra:4.1
    container_name: cassandra_dev
    # Redémarre automatiquement sauf si arrêté manuellement
    restart: unless-stopped

    # Variables d'environnement
    environment:
      # Nom du cluster (important pour la configuration)
      CASSANDRA_CLUSTER_NAME: 'DevCluster'
      # Nom du datacenter (utile pour la réplication)
      CASSANDRA_DC: 'datacenter1'
      # Nom du rack (topology)
      CASSANDRA_RACK: 'rack1'
      # Adresse d'écoute (0.0.0.0 = toutes les interfaces)
      CASSANDRA_LISTEN_ADDRESS: 'auto'
      # Nœud seed (pour rejoindre le cluster)
      CASSANDRA_SEEDS: 'cassandra'

    # Ports exposés
    ports:
      # Port CQL (Cassandra Query Language)
      - "9042:9042"
      # Port JMX (monitoring, optionnel)
      - "7199:7199"

    # Volume pour persister les données
    volumes:
      - cassandra_data:/var/lib/cassandra

    # Configuration de santé (healthcheck)
    healthcheck:
      # Vérifie que Cassandra répond
      test: ["CMD-SHELL", "cqlsh -e 'describe cluster'"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s

# Déclaration du volume pour la persistance
volumes:
  cassandra_data:
    driver: local
```

### 📝 Explications des paramètres

| Paramètre | Description |
|-----------|-------------|
| `image: cassandra:4.1` | Version de Cassandra à utiliser (4.1 est stable) |
| `container_name` | Nom du conteneur (pour le retrouver facilement) |
| `CASSANDRA_CLUSTER_NAME` | Nom de votre cluster (regroupement logique) |
| `CASSANDRA_DC` | Nom du datacenter (important pour la réplication) |
| `9042` | Port CQL pour se connecter avec des clients |
| `7199` | Port JMX pour le monitoring (optionnel) |
| `cassandra_data` | Volume Docker pour persister les données |
| `healthcheck` | Vérifie automatiquement que Cassandra fonctionne |

---

## ▶️ Étape 2 : Lancement du conteneur

Lancez Cassandra avec Docker Compose :

```bash
docker-compose up -d
```

**Explication de la commande :**
- `up` : Démarre les services définis dans le docker-compose.yml
- `-d` : Mode "detached" (en arrière-plan)

### ⏳ Patience : Cassandra prend du temps à démarrer

Cassandra peut prendre **30 secondes à 2 minutes** pour démarrer complètement. C'est normal !

**Suivez les logs en temps réel :**
```bash
docker-compose logs -f cassandra
```

**Attendez de voir ces lignes :**
```
Starting listening for CQL clients...
Node is now in NORMAL state
```

**Appuyez sur `Ctrl+C`** pour sortir des logs (le conteneur continue de tourner).

### ✅ Vérifier que Cassandra est prêt

Utilisez le healthcheck :

```bash
docker-compose ps
```

Vous devriez voir :
```
NAME              STATE     STATUS
cassandra_dev     running   healthy
```

Si le statut est `starting` ou `unhealthy`, attendez encore un peu.

---

## 🔌 Étape 3 : Connexion à Cassandra avec `cqlsh`

**cqlsh** est le shell interactif de Cassandra (comme `mysql` pour MariaDB ou `psql` pour PostgreSQL).

### Se connecter au conteneur

```bash
docker exec -it cassandra_dev cqlsh
```

**Explication :**
- `docker exec` : Exécute une commande dans un conteneur
- `-it` : Mode interactif avec terminal
- `cassandra_dev` : Nom du conteneur
- `cqlsh` : Le shell CQL

**Vous devriez voir :**
```
Connected to DevCluster at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.x | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh>
```

🎉 **Félicitations !** Vous êtes connecté à Cassandra.

---

## 📊 Étape 4 : Premiers pas avec CQL

### Comprendre les concepts de base

| Concept Cassandra | Équivalent SQL | Description |
|-------------------|----------------|-------------|
| **Keyspace** | Base de données | Conteneur de tables |
| **Table** | Table | Structure de données |
| **Row** | Ligne | Enregistrement |
| **Column** | Colonne | Champ de données |
| **Partition Key** | Clé primaire | Détermine sur quel nœud les données sont stockées |

### 1️⃣ Créer un keyspace

Un **keyspace** est l'équivalent d'une base de données en SQL.

```sql
CREATE KEYSPACE IF NOT EXISTS demo
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

**Explication :**
- `CREATE KEYSPACE` : Crée un nouveau keyspace
- `IF NOT EXISTS` : Évite une erreur si le keyspace existe déjà
- `SimpleStrategy` : Stratégie de réplication (pour un nœud unique)
- `replication_factor: 1` : Une seule copie des données (pour le dev)

### 2️⃣ Utiliser le keyspace

```sql
USE demo;
```

Maintenant, toutes vos commandes s'exécutent dans le keyspace `demo`.

### 3️⃣ Créer une table

Créons une table simple pour stocker des utilisateurs :

```sql
CREATE TABLE IF NOT EXISTS users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT,
  created_at TIMESTAMP
);
```

**Explication :**
- `UUID` : Identifiant unique universel (type courant dans Cassandra)
- `PRIMARY KEY` : La clé primaire (aussi appelée partition key ici)
- `TEXT` : Type chaîne de caractères
- `TIMESTAMP` : Date et heure

### 4️⃣ Insérer des données

```sql
-- Insertion avec UUID généré
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'alice', 'alice@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'bob', 'bob@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'charlie', 'charlie@example.com', toTimestamp(now()));
```

**Fonctions utiles :**
- `uuid()` : Génère un UUID aléatoire
- `now()` : Date/heure actuelle
- `toTimestamp()` : Convertit en timestamp

### 5️⃣ Requêter les données

**Afficher toutes les lignes :**
```sql
SELECT * FROM users;
```

**Résultat :**
```
 user_id                              | username | email               | created_at
--------------------------------------+----------+---------------------+---------------------------
 a1b2c3d4-...                         | alice    | alice@example.com   | 2024-10-31 10:30:00+0000
 e5f6g7h8-...                         | bob      | bob@example.com     | 2024-10-31 10:30:05+0000
 i9j0k1l2-...                         | charlie  | charlie@example.com | 2024-10-31 10:30:10+0000

(3 rows)
```

**Rechercher un utilisateur spécifique (par PRIMARY KEY) :**
```sql
-- Remplacez par un UUID réel de votre table
SELECT * FROM users WHERE user_id = a1b2c3d4-...;
```

⚠️ **Important :** Dans Cassandra, les requêtes sont optimisées pour les recherches par clé primaire. Les requêtes sans clé primaire nécessitent `ALLOW FILTERING` (à éviter en production).

### 6️⃣ Mettre à jour des données

```sql
-- Mise à jour de l'email d'un utilisateur
UPDATE users SET email = 'alice.updated@example.com'
WHERE user_id = a1b2c3d4-...;
```

### 7️⃣ Supprimer des données

```sql
-- Supprimer un utilisateur
DELETE FROM users WHERE user_id = a1b2c3d4-...;
```

### 8️⃣ Quitter cqlsh

```sql
EXIT;
```

Ou appuyez sur `Ctrl+D`.

---

## 🔍 Étape 5 : Commandes utiles

### Lister les keyspaces

```sql
DESCRIBE KEYSPACES;
```

**Résultat :**
```
system       system_distributed  system_traces  system_virtual_schema
system_auth  system_schema       system_views   demo
```

### Voir la structure d'un keyspace

```sql
DESCRIBE KEYSPACE demo;
```

### Lister les tables d'un keyspace

```sql
USE demo;
DESCRIBE TABLES;
```

### Voir la structure d'une table

```sql
DESCRIBE TABLE users;
```

### Vérifier l'état du cluster

```sql
-- Depuis cqlsh
DESCRIBE CLUSTER;
```

**Ou depuis le terminal :**
```bash
docker exec -it cassandra_dev nodetool status
```

**Résultat attendu :**
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns    Host ID                               Rack
UN  172.18.0.2  108.5 KiB  16      100.0%  abc123...                             rack1
```

- `UN` = Up and Normal (tout va bien !)

---

## 📈 Étape 6 : Vérifications et monitoring

### Voir les logs en direct

```bash
docker-compose logs -f cassandra
```

### Vérifier l'utilisation des ressources

```bash
docker stats cassandra_dev
```

### Accéder au conteneur en bash

```bash
docker exec -it cassandra_dev bash
```

**Commandes utiles dans le conteneur :**
```bash
# Version de Cassandra
cassandra -v

# Outils nodetool
nodetool status
nodetool info
nodetool describecluster

# Sortir du conteneur
exit
```

---

## 🛑 Étape 7 : Arrêt et suppression

### Arrêter le conteneur (sans supprimer les données)

```bash
docker-compose stop
```

### Redémarrer le conteneur

```bash
docker-compose start
```

### Arrêter ET supprimer le conteneur (données conservées)

```bash
docker-compose down
```

Les données restent dans le volume Docker `cassandra_data`.

### Suppression COMPLÈTE (conteneur + données)

⚠️ **ATTENTION : Cette commande supprime TOUTES vos données !**

```bash
# Supprimer conteneur et volumes
docker-compose down -v

# Vérifier que le volume est supprimé
docker volume ls
```

---

## 💡 Astuces pour débutants

### 1. Cassandra est lent au démarrage
C'est normal ! Cassandra est conçu pour de gros clusters. Attendez 1-2 minutes après `docker-compose up -d`.

### 2. Erreur "Connection refused"
Vérifiez avec `docker-compose ps` que le statut est `healthy` avant de vous connecter.

### 3. Les requêtes sans clé primaire
Cassandra est optimisé pour les requêtes avec clé primaire. Évitez `ALLOW FILTERING` en production.

### 4. UUID vs INT
Cassandra utilise beaucoup les UUID pour les clés primaires (distribution optimale sur les nœuds).

### 5. Pas de jointures !
Cassandra ne supporte pas les jointures SQL classiques. Pensez "dénormalisation" et "tables par requête".

---

## 🎓 Concepts clés à retenir

| Concept | Description |
|---------|-------------|
| **Keyspace** | Conteneur de tables (= base de données) |
| **Partition Key** | Détermine où les données sont stockées |
| **CQL** | Cassandra Query Language (ressemble à SQL mais différent) |
| **Nœud** | Une instance Cassandra (ici, un seul) |
| **Cluster** | Ensemble de nœuds (ici, cluster d'un nœud) |
| **Replication Factor** | Nombre de copies des données (ici, 1) |

---

## 📚 Prochaines étapes

Maintenant que vous maîtrisez la configuration de base, explorez :

- **[7.2 Configuration avec IP fixe](02-config-ip-fixe.md)** : Assigner une adresse IP statique
- **[7.3 Cluster simple](03-cluster-simple.md)** : Créer un cluster à 3 nœuds
- **[7.4 Gestion des keyspaces](04-gestion-keyspaces.md)** : Stratégies de réplication avancées

---

## ❓ Questions fréquentes (FAQ)

**Q : Puis-je utiliser cette config en production ?**
R : Non, c'est pour le développement. Production = cluster multi-nœuds avec réplication.

**Q : Cassandra vs MongoDB ?**
R : Cassandra = haute disponibilité, écritures massives. MongoDB = flexibilité, requêtes complexes.

**Q : Pourquoi Cassandra est-il si lent à démarrer ?**
R : Il initialise beaucoup de composants (JVM, compaction, etc.). C'est normal.

**Q : Puis-je utiliser un client GUI ?**
R : Oui ! DataGrip, DBeaver et TablePlus supportent Cassandra (via driver CQL).

**Q : Comment sauvegarder mes données ?**
R : Utilisez `nodetool snapshot` ou sauvegardez le volume Docker. Voir [Annexe C](/annexes/C-gestion-volumes.md).

---

## 🔗 Ressources complémentaires

- [Documentation officielle Cassandra](https://cassandra.apache.org/doc/latest/)
- [CQL Reference](https://cassandra.apache.org/doc/latest/cassandra/cql/)
- [Image Docker Cassandra](https://hub.docker.com/_/cassandra)
- [DataStax Academy](https://www.datastax.com/learn) (cours gratuits)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
