# Cassandra - Introduction

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que Apache Cassandra ?

**Apache Cassandra** est une base de données NoSQL distribuée, open-source, conçue pour gérer d'énormes quantités de données sur plusieurs serveurs tout en offrant une haute disponibilité sans point de défaillance unique.

### Caractéristiques principales

| Caractéristique | Description |
|----------------|-------------|
| 🗄️ **Type** | Base de données NoSQL à colonnes larges (wide-column store) |
| 🌐 **Architecture** | Distribuée, décentralisée (peer-to-peer) |
| 📈 **Scalabilité** | Scalabilité linéaire horizontale |
| ⚡ **Performance** | Optimisée pour les écritures massives |
| 🔄 **Disponibilité** | Haute disponibilité (pas de single point of failure) |
| 🌍 **Réplication** | Réplication multi-datacenter native |
| 💪 **Tolérance** | Tolérance aux pannes et aux partitions réseau |

---

## 🎯 Pourquoi choisir Cassandra ?

### Points forts

✅ **Écritures ultra-rapides** : Optimisé pour des millions d'écritures/seconde
✅ **Scalabilité horizontale** : Ajoutez des nœuds pour augmenter les performances
✅ **Haute disponibilité** : Pas de downtime lors des maintenances
✅ **Réplication géographique** : Données répliquées sur plusieurs datacenters
✅ **Pas de point de défaillance** : Tous les nœuds sont égaux (pas de master)
✅ **Tolérance aux pannes** : Continue de fonctionner même si plusieurs nœuds tombent
✅ **Modèle de données flexible** : Schéma défini mais modifiable

### Points faibles

❌ **Pas de JOIN** : Pas de requêtes relationnelles complexes
❌ **Courbe d'apprentissage** : Modélisation différente du SQL classique
❌ **Cohérence éventuelle** : Par défaut, privilégie la disponibilité
❌ **Ressources** : Consomme beaucoup de RAM et CPU
❌ **Transactions limitées** : Pas de transactions ACID multi-partitions
❌ **Pas de recherche full-text** : Nécessite des outils externes (Solr, Elasticsearch)

---

## 🏗️ Architecture de Cassandra

### Concepts fondamentaux

```
┌──────────────────────────────────────────────┐
│              Cluster Cassandra               │
├──────────────────────────────────────────────┤
│                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │  Nœud 1  │───│  Nœud 2  │───│  Nœud 3  │  │
│  │ (Seed)   │   │          │   │          │  │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘  │
│       │              │              │        │
│       └──────────────┴──────────────┘        │
│          Gossip Protocol (P2P)               │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │   Réplication des données            │    │
│  │   (Replication Factor = 3)           │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

### Composants clés

| Composant | Description |
|-----------|-------------|
| **Cluster** | Ensemble de nœuds Cassandra travaillant ensemble |
| **Nœud** | Une instance Cassandra (un serveur) |
| **Datacenter** | Regroupement logique de nœuds (souvent géographique) |
| **Rack** | Subdivision d'un datacenter (pour la tolérance de panne) |
| **Seed Node** | Nœud de référence pour rejoindre le cluster |
| **Keyspace** | Équivalent d'une base de données (conteneur de tables) |
| **Table** | Structure de données (comme en SQL) |
| **Partition** | Unité de distribution des données |

### Architecture peer-to-peer

```
        Tous les nœuds sont égaux
               (No Master)
                    │
        ┌───────────┼───────────┐
        │           │           │
    ┌───▼───┐   ┌───▼───┐   ┌───▼───┐
    │ Nœud  │   │ Nœud  │   │ Nœud  │
    │   1   │◄──┤   2   │──►│   3   │
    └───┬───┘   └───┬───┘   └───┬───┘
        │           │           │
        └───────────┼───────────┘
                    │
         Chaque nœud peut gérer
          des lectures/écritures
```

**Avantages :**
- Pas de goulot d'étranglement (bottleneck)
- Tolérance aux pannes élevée
- Scalabilité linéaire

---

## 📊 Modèle de données

### Structure hiérarchique

```
Cluster
  └── Keyspace (base de données)
      ├── Table 1
      │   ├── Partition 1 (groupe de lignes)
      │   │   ├── Row 1
      │   │   ├── Row 2
      │   │   └── Row 3
      │   └── Partition 2
      │       ├── Row 1
      │       └── Row 2
      └── Table 2
          └── ...
```

### Comparaison avec SQL

| Cassandra | SQL | Description |
|-----------|-----|-------------|
| **Keyspace** | Database | Conteneur principal |
| **Table** | Table | Structure de données |
| **Row** | Row | Enregistrement |
| **Column** | Column | Champ |
| **Partition Key** | Primary Key (partie) | Clé de distribution |
| **Clustering Key** | - | Clé de tri dans une partition |
| **CQL** | SQL | Langage de requête |

### Exemple de table

```sql
CREATE TABLE users (
  user_id UUID,           -- Partition Key
  country TEXT,
  created_at TIMESTAMP,   -- Clustering Key
  username TEXT,
  email TEXT,
  PRIMARY KEY (user_id, created_at)
);
```

**Explications :**
- `user_id` : **Partition Key** (détermine sur quel nœud stocker)
- `created_at` : **Clustering Key** (trie les données dans la partition)
- Les lignes avec le même `user_id` sont stockées ensemble

---

## 🔄 Réplication et cohérence

### Replication Factor (RF)

**Définition :** Nombre de copies de chaque donnée dans le cluster.

```
Replication Factor = 3

     Donnée X
        │
    ┌───┼───┐
    │   │   │
┌───▼─┐ │ ┌─▼───┐
│Nœud1│ │ │Nœud3│
│ X   │ │ │ X   │
└─────┘ │ └─────┘
        │
    ┌───▼───┐
    │ Nœud2 │
    │   X   │
    └───────┘
```

**Impact :**
- RF = 1 : Aucune tolérance de panne
- RF = 2 : Tolère la perte d'un nœud
- RF = 3 : Tolère la perte de deux nœuds (recommandé)

### Consistency Level (CL)

**Définition :** Nombre de réplicas qui doivent répondre pour une opération.

| Niveau | Description | Usage |
|--------|-------------|-------|
| **ONE** | 1 réplica répond | Rapide, peu fiable |
| **QUORUM** | Majorité répond (RF/2 + 1) | **Recommandé** (équilibré) |
| **ALL** | Tous les réplicas répondent | Très fiable, lent |
| **LOCAL_QUORUM** | Majorité dans le datacenter local | Multi-datacenter |

**Formule de cohérence forte :**
```
CL_read + CL_write > RF = Cohérence forte
Exemple : QUORUM + QUORUM > 3 = OK
```

---

## 🎮 CQL : Cassandra Query Language

### Syntaxe similaire à SQL

**Création d'un keyspace :**
```sql
CREATE KEYSPACE demo
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};
```

**Création d'une table :**
```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT
);
```

**Insertion :**
```sql
INSERT INTO users (user_id, username, email)
VALUES (uuid(), 'alice', 'alice@example.com');
```

**Sélection :**
```sql
SELECT * FROM users WHERE user_id = <uuid>;
```

### Différences importantes avec SQL

| SQL | CQL | Remarque |
|-----|-----|----------|
| `JOIN` | ❌ Pas supporté | Dénormaliser les données |
| `GROUP BY` | ❌ Limité | Faire dans l'application |
| `ORDER BY` | ✅ Sur Clustering Key uniquement | Tri prédéfini |
| `WHERE` | ✅ Sur clés primaires | Ou avec `ALLOW FILTERING` (lent) |
| `UPDATE` | ✅ Upsert | Insert si n'existe pas |
| `DELETE` | ✅ Avec tombstones | Marquage pour suppression |

---

## 📈 Cas d'usage typiques

### ✅ Cassandra est excellent pour :

| Cas d'usage | Raison |
|-------------|--------|
| **IoT et capteurs** | Millions d'écritures/seconde |
| **Logs d'applications** | Grande volumétrie, écritures continues |
| **Historique d'événements** | Append-only, pas de mises à jour |
| **Messagerie/Chat** | Distribution géographique |
| **Systèmes de recommandation** | Grandes quantités de données |
| **Time-series data** | Séries temporelles (métriques, analytics) |
| **Catalogues produits** | Lecture rapide, haute disponibilité |
| **Systèmes de paiement** | Résilience, pas de downtime |

### ❌ Cassandra est moins adapté pour :

| Cas d'usage | Raison |
|-------------|--------|
| **Transactions ACID complexes** | Pas de transactions multi-partitions |
| **Requêtes ad-hoc complexes** | Modélisation rigide, pas de JOIN |
| **Agrégations en temps réel** | Pas de GROUP BY efficace |
| **Petites bases de données** | Overhead trop important |
| **Données fortement relationnelles** | Dénormalisation nécessaire |
| **Budget limité** | Nécessite beaucoup de ressources |

---

## 🏢 Qui utilise Cassandra ?

### Entreprises majeures

| Entreprise | Usage |
|------------|-------|
| **Netflix** | Données de visionnage (millions d'écritures/sec) |
| **Apple** | iCloud, plus de 75 000 nœuds |
| **Uber** | Données de géolocalisation en temps réel |
| **Instagram** | Stockage des photos et métadonnées |
| **Spotify** | Historique d'écoute et recommandations |
| **Discord** | Messages et historique des serveurs |
| **Reddit** | Système de vote et commentaires |

### Statistiques impressionnantes

- 🌍 **Apple** : Plus de 75 000 nœuds en production
- 📊 **Netflix** : Plus de 1 trillion de requêtes/jour
- ⚡ **Discord** : Plus de 4 milliards de messages/jour
- 💾 **Uber** : Plus de 100 PB de données

---

## 🆚 Cassandra vs autres bases NoSQL

### Comparaison rapide

| Base | Type | Forces | Faiblesses |
|------|------|--------|------------|
| **Cassandra** | Wide-column | Écritures massives, HA | Pas de JOIN, complexe |
| **MongoDB** | Document | Flexibilité, requêtes riches | Scalabilité limitée |
| **Redis** | Clé-valeur | Ultra-rapide, cache | Volatile, volumétrie limitée |
| **Neo4j** | Graphe | Relations complexes | Scalabilité horizontale limitée |
| **Elasticsearch** | Recherche | Full-text search | Pas pour stockage primaire |

### Quand choisir Cassandra plutôt que...

**MongoDB :**
- Si vous avez besoin de scalabilité linéaire
- Si vous privilégiez la disponibilité à la cohérence
- Si vous avez beaucoup d'écritures

**PostgreSQL :**
- Si vous n'avez pas besoin de JOIN complexes
- Si vous gérez des pétaoctets de données
- Si vous ne pouvez pas vous permettre de downtime

**Redis :**
- Si vous avez besoin de persistance garantie
- Si vos données dépassent la RAM disponible
- Si vous avez besoin de réplication géographique

---

## 🛠️ Outils et écosystème

### Outils de gestion

| Outil | Description |
|-------|-------------|
| **cqlsh** | Shell interactif (inclus) |
| **nodetool** | Commandes d'administration (inclus) |
| **DataStax Studio** | IDE graphique |
| **DataStax DevCenter** | Client SQL-like |
| **DBeaver** | Client universel (supporte Cassandra) |
| **TablePlus** | Client multi-DB moderne |

### Monitoring

| Outil | Description |
|-------|-------------|
| **Prometheus + Grafana** | Métriques et dashboards |
| **DataStax OpsCenter** | Monitoring complet (commercial) |
| **Cassandra Reaper** | Gestion des réparations |
| **JConsole / JVisualVM** | Monitoring JVM |

### Drivers officiels

| Langage | Driver |
|---------|--------|
| **Java** | DataStax Java Driver |
| **Python** | cassandra-driver |
| **Node.js** | cassandra-driver |
| **Go** | gocql |
| **C#** | DataStax C# Driver |
| **PHP** | php-driver |

---

## 📚 Versions de Cassandra

### Historique des versions majeures

| Version | Date | Nouveautés |
|---------|------|------------|
| **1.0** | 2011 | Première version stable |
| **2.0** | 2013 | CQL amélioré, triggers |
| **3.0** | 2015 | Materialized Views, amélioration performances |
| **4.0** | 2021 | Virtual tables, audit logging, amélioration stabilité |
| **4.1** | 2022 | Amélioration performances, bug fixes |
| **5.0** | 2023+ | En développement (support Java 17) |

### Version recommandée pour Docker

**Cassandra 4.1** : Stable, performant, bien supporté.

```yaml
services:
  cassandra:
    image: cassandra:4.1  # Version recommandée
```

---

## 🎓 Concepts avancés (aperçu)

### 1. Compaction

Processus de fusion des SSTables (fichiers sur disque) pour optimiser les lectures.

**Stratégies :**
- **SizeTieredCompactionStrategy** : Par défaut
- **LeveledCompactionStrategy** : Pour lectures fréquentes
- **TimeWindowCompactionStrategy** : Pour time-series data

### 2. Gossip Protocol

Protocole de communication peer-to-peer entre nœuds.
- Découverte des nœuds
- Détection de pannes
- Échange d'informations sur l'état du cluster

### 3. Hinted Handoff

Mécanisme pour gérer les nœuds temporairement hors ligne.
- Les écritures sont stockées sur d'autres nœuds
- Rejouées quand le nœud revient

### 4. Read Repair

Mécanisme pour garantir la cohérence des données.
- Détecte les incohérences lors des lectures
- Répare automatiquement

---

## 💡 Bonnes pratiques (résumé)

### Modélisation

✅ **Modéliser selon les requêtes**, pas selon les entités
✅ **Dénormaliser** les données (dupliquer pour optimiser)
✅ **Une table par pattern de requête**
✅ **Limiter la taille des partitions** (< 100 MB)
✅ **Utiliser des UUID/TIMEUUID** pour les clés

### Configuration

✅ **Replication Factor ≥ 3** en production
✅ **Consistency Level = QUORUM** (équilibré)
✅ **NetworkTopologyStrategy** (pas SimpleStrategy)
✅ **Monitoring actif** (métriques, alertes)
✅ **Backups réguliers** (snapshots)

### Développement

✅ **Tester avec Docker** avant production
✅ **Commencer avec un nœud unique** pour apprendre
✅ **Utiliser cqlsh** pour les tests
✅ **Documenter les keyspaces** et tables
✅ **Éviter ALLOW FILTERING** en production

---

## 🎬 Prochaines étapes

Maintenant que vous comprenez les concepts de Cassandra, explorez les fiches pratiques :

1. **[7.1 Configuration basique d'un nœud unique](01-config-noeud-unique.md)**
   - Démarrer votre premier conteneur Cassandra
   - Se connecter avec cqlsh
   - Créer vos premières tables

2. **[7.2 Configuration avec IP fixe](02-config-ip-fixe.md)**
   - Assigner une IP statique
   - Configurer le réseau Docker

3. **[7.3 Cluster simple (3 nœuds)](03-cluster-simple.md)**
   - Créer un cluster de développement
   - Tester la réplication
   - Vérifier la haute disponibilité

4. **[7.4 Gestion des keyspaces et tables](04-gestion-keyspaces.md)**
   - Stratégies de réplication avancées
   - Modélisation de données
   - Bonnes pratiques

---

## 📖 Glossaire

| Terme | Définition |
|-------|------------|
| **Cluster** | Ensemble de nœuds Cassandra |
| **Nœud** | Une instance Cassandra |
| **Keyspace** | Équivalent d'une base de données |
| **Partition Key** | Clé qui détermine la distribution des données |
| **Clustering Key** | Clé qui trie les données dans une partition |
| **Replication Factor (RF)** | Nombre de copies des données |
| **Consistency Level (CL)** | Nombre de réplicas qui doivent répondre |
| **Seed Node** | Nœud de référence pour rejoindre le cluster |
| **Gossip** | Protocole de communication peer-to-peer |
| **SSTable** | Fichier de stockage sur disque (Sorted String Table) |
| **Compaction** | Fusion des SSTables pour optimisation |
| **Tombstone** | Marqueur de suppression |

---

## 🔗 Ressources complémentaires

### Documentation officielle

- [Site officiel Apache Cassandra](https://cassandra.apache.org/)
- [Documentation Cassandra 4.1](https://cassandra.apache.org/doc/latest/)
- [CQL Reference](https://cassandra.apache.org/doc/latest/cassandra/cql/)

### Apprentissage

- [DataStax Academy](https://www.datastax.com/learn) - Cours gratuits
- [Cassandra: The Definitive Guide](https://www.oreilly.com/library/view/cassandra-the-definitive/9781491933657/) - Livre de référence
- [Docker Hub - Image Cassandra](https://hub.docker.com/_/cassandra)

### Communauté

- [Cassandra Mailing Lists](https://cassandra.apache.org/community/)
- [Stack Overflow - Tag Cassandra](https://stackoverflow.com/questions/tagged/cassandra)
- [Reddit - r/cassandra](https://www.reddit.com/r/cassandra/)

---

## ❓ Questions fréquentes (FAQ)

**Q : Cassandra est-il difficile à apprendre ?**
R : La courbe d'apprentissage est raide si vous venez du SQL, mais les concepts sont logiques une fois compris.

**Q : Puis-je utiliser Cassandra pour une petite application ?**
R : Techniquement oui, mais c'est du "over-engineering". Utilisez PostgreSQL ou MongoDB à la place.

**Q : Cassandra consomme-t-il beaucoup de ressources ?**
R : Oui, surtout en RAM (minimum 4 GB par nœud, 16+ GB en production).

**Q : Puis-je migrer de MongoDB vers Cassandra ?**
R : Oui, mais vous devrez remodéliser vos données (pas de migration directe).

**Q : Cassandra est-il gratuit ?**
R : Oui, Apache Cassandra est open-source (licence Apache 2.0). DataStax propose une version commerciale avec support.

**Q : Docker est-il adapté pour Cassandra en production ?**
R : Possible avec Kubernetes (StatefulSets), mais les déploiements bare-metal sont plus courants en production.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
