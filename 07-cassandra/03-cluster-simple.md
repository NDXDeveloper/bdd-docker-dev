# 7.3 Cluster Cassandra simple (3 nœuds)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans les fiches précédentes, nous avons travaillé avec un **nœud unique** de Cassandra. C'est parfait pour le développement, mais Cassandra est conçu pour fonctionner en **cluster** (plusieurs nœuds).

### Qu'est-ce qu'un cluster Cassandra ?

Un **cluster** est un ensemble de **nœuds** (serveurs) Cassandra qui travaillent ensemble pour :
- 🔄 **Répliquer les données** entre plusieurs nœuds
- ⚡ **Haute disponibilité** : Si un nœud tombe, les autres continuent
- 📈 **Scalabilité** : Ajouter des nœuds pour gérer plus de données
- 🚀 **Performances** : Distribuer la charge sur plusieurs serveurs

### Architecture d'un cluster 3 nœuds

```
┌────────────────────────────────────────────┐
│         Cluster "DevCluster"               │
├────────────────────────────────────────────┤
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Nœud 1  │  │  Nœud 2  │  │  Nœud 3  │  │
│  │172.20.0.2│  │172.20.0.3│  │172.20.0.4│  │
│  │   Seed   │  │          │  │          │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
│       │             │             │        │
│       └─────────────┴─────────────┘        │
│          Communication interne             │
└────────────────────────────────────────────┘
```

**Pourquoi 3 nœuds ?**
- Minimum recommandé pour la réplication (Replication Factor = 3)
- Assure la haute disponibilité (perte d'un nœud tolérée)
- Bon équilibre pour l'apprentissage (pas trop lourd)

---

## 🎯 Ce que vous allez apprendre

À la fin de cette fiche, vous saurez :
- ✅ Créer un cluster Cassandra de 3 nœuds avec Docker Compose
- ✅ Configurer les nœuds seeds (points d'entrée du cluster)
- ✅ Assigner des IPs fixes à chaque nœud
- ✅ Vérifier l'état du cluster avec `nodetool`
- ✅ Créer un keyspace avec réplication sur 3 nœuds
- ✅ Tester la résilience (arrêt d'un nœud)
- ✅ Comprendre les concepts de réplication et de consistance

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :

- **Docker** installé (version 20.10+)
- **Docker Compose** installé (version 2.0+)
- Au moins **4 GB de RAM disponible** (3 nœuds = ressources × 3)
- Avoir lu les fiches précédentes :
  - [7.1 - Configuration basique](01-config-noeud-unique.md)
  - [7.2 - Configuration avec IP fixe](02-config-ip-fixe.md)

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🌐 Étape 1 : Création du réseau Docker

Créez un réseau dédié pour le cluster :

```bash
docker network create --subnet=172.20.0.0/16 cassandra_cluster_net
```

**Vérification :**
```bash
docker network ls | grep cassandra_cluster_net
```

---

## 📝 Étape 2 : Configuration du `docker-compose.yml`

Créez un nouveau dossier pour ce projet :

```bash
mkdir cassandra-cluster
cd cassandra-cluster
```

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  # ========================================
  # NŒUD 1 - Seed Node (Point d'entrée)
  # ========================================
  cassandra-seed:
    image: cassandra:4.1
    container_name: cassandra-seed
    hostname: cassandra-seed
    restart: unless-stopped

    environment:
      # Configuration du cluster
      CASSANDRA_CLUSTER_NAME: 'DevCluster'
      CASSANDRA_DC: 'datacenter1'
      CASSANDRA_RACK: 'rack1'
      CASSANDRA_ENDPOINT_SNITCH: 'GossipingPropertyFileSnitch'

      # Configuration réseau
      CASSANDRA_LISTEN_ADDRESS: 'auto'
      CASSANDRA_RPC_ADDRESS: '0.0.0.0'
      CASSANDRA_BROADCAST_ADDRESS: '172.20.0.2'
      CASSANDRA_BROADCAST_RPC_ADDRESS: '172.20.0.2'

      # SEED NODE : Ce nœud est le point d'entrée du cluster
      CASSANDRA_SEEDS: '172.20.0.2'

      # Optimisations pour le développement
      MAX_HEAP_SIZE: '512M'
      HEAP_NEWSIZE: '100M'

    ports:
      # Port CQL pour connexion externe
      - "9042:9042"

    volumes:
      - cassandra_seed_data:/var/lib/cassandra

    networks:
      cassandra_cluster_net:
        ipv4_address: 172.20.0.2

    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 90s

  # ========================================
  # NŒUD 2
  # ========================================
  cassandra-node2:
    image: cassandra:4.1
    container_name: cassandra-node2
    hostname: cassandra-node2
    restart: unless-stopped

    environment:
      CASSANDRA_CLUSTER_NAME: 'DevCluster'
      CASSANDRA_DC: 'datacenter1'
      CASSANDRA_RACK: 'rack1'
      CASSANDRA_ENDPOINT_SNITCH: 'GossipingPropertyFileSnitch'

      CASSANDRA_LISTEN_ADDRESS: 'auto'
      CASSANDRA_RPC_ADDRESS: '0.0.0.0'
      CASSANDRA_BROADCAST_ADDRESS: '172.20.0.3'
      CASSANDRA_BROADCAST_RPC_ADDRESS: '172.20.0.3'

      # Ce nœud rejoint via le seed
      CASSANDRA_SEEDS: '172.20.0.2'

      MAX_HEAP_SIZE: '512M'
      HEAP_NEWSIZE: '100M'

    volumes:
      - cassandra_node2_data:/var/lib/cassandra

    networks:
      cassandra_cluster_net:
        ipv4_address: 172.20.0.3

    # Attend que le seed soit prêt avant de démarrer
    depends_on:
      cassandra-seed:
        condition: service_healthy

    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 90s

  # ========================================
  # NŒUD 3
  # ========================================
  cassandra-node3:
    image: cassandra:4.1
    container_name: cassandra-node3
    hostname: cassandra-node3
    restart: unless-stopped

    environment:
      CASSANDRA_CLUSTER_NAME: 'DevCluster'
      CASSANDRA_DC: 'datacenter1'
      CASSANDRA_RACK: 'rack1'
      CASSANDRA_ENDPOINT_SNITCH: 'GossipingPropertyFileSnitch'

      CASSANDRA_LISTEN_ADDRESS: 'auto'
      CASSANDRA_RPC_ADDRESS: '0.0.0.0'
      CASSANDRA_BROADCAST_ADDRESS: '172.20.0.4'
      CASSANDRA_BROADCAST_RPC_ADDRESS: '172.20.0.4'

      # Ce nœud rejoint via le seed
      CASSANDRA_SEEDS: '172.20.0.2'

      MAX_HEAP_SIZE: '512M'
      HEAP_NEWSIZE: '100M'

    volumes:
      - cassandra_node3_data:/var/lib/cassandra

    networks:
      cassandra_cluster_net:
        ipv4_address: 172.20.0.4

    # Attend que le seed soit prêt avant de démarrer
    depends_on:
      cassandra-seed:
        condition: service_healthy

    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 90s

# ========================================
# VOLUMES (un par nœud)
# ========================================
volumes:
  cassandra_seed_data:
    driver: local
  cassandra_node2_data:
    driver: local
  cassandra_node3_data:
    driver: local

# ========================================
# RÉSEAU
# ========================================
networks:
  cassandra_cluster_net:
    external: true
```

---

## 🔍 Explications détaillées

### Structure du cluster

| Nœud | IP | Rôle | Description |
|------|-----|------|-------------|
| **cassandra-seed** | `172.20.0.2` | Seed Node | Point d'entrée, premier à démarrer |
| **cassandra-node2** | `172.20.0.3` | Node | Rejoint le cluster via le seed |
| **cassandra-node3** | `172.20.0.4` | Node | Rejoint le cluster via le seed |

### Variables d'environnement importantes

| Variable | Valeur | Description |
|----------|--------|-------------|
| `CASSANDRA_SEEDS` | `172.20.0.2` | Adresse du(des) nœud(s) seed |
| `CASSANDRA_ENDPOINT_SNITCH` | `GossipingPropertyFileSnitch` | Stratégie de topologie du cluster |
| `MAX_HEAP_SIZE` | `512M` | Mémoire JVM (réduite pour le dev) |
| `HEAP_NEWSIZE` | `100M` | Mémoire pour les nouveaux objets |

### Ordre de démarrage

```yaml
depends_on:
  cassandra-seed:
    condition: service_healthy
```

- Le **seed** démarre en premier
- Les **nœuds 2 et 3** attendent que le seed soit "healthy"
- Cela garantit un démarrage ordonné du cluster

### Volumes séparés

Chaque nœud a son propre volume :
- `cassandra_seed_data`
- `cassandra_node2_data`
- `cassandra_node3_data`

**Pourquoi ?** Chaque nœud stocke une partie différente des données (distribution).

---

## ▶️ Étape 3 : Lancement du cluster

### Démarrer les 3 nœuds

```bash
docker-compose up -d
```

**⏳ Patience :** Le démarrage complet d'un cluster prend **3 à 5 minutes**.

### Suivre les logs

**Logs du seed :**
```bash
docker-compose logs -f cassandra-seed
```

**Attendez de voir :**
```
Starting listening for CQL clients on /0.0.0.0:9042
Node is now in NORMAL state
```

**Logs des autres nœuds :**
```bash
docker-compose logs -f cassandra-node2
docker-compose logs -f cassandra-node3
```

**Appuyez sur Ctrl+C** pour sortir des logs.

---

## ✅ Étape 4 : Vérification du cluster

### Méthode 1 : État des conteneurs

```bash
docker-compose ps
```

**Résultat attendu :**
```
NAME                STATE      STATUS
cassandra-seed      running    healthy
cassandra-node2     running    healthy
cassandra-node3     running    healthy
```

✅ Tous doivent être `running` et `healthy`.

### Méthode 2 : Commande `nodetool status`

C'est **LA** commande pour vérifier l'état du cluster :

```bash
docker exec -it cassandra-seed nodetool status
```

**Résultat attendu :**
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address      Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.20.0.2   108.5 KiB  16      66.7%             abc123...                             rack1
UN  172.20.0.3   104.2 KiB  16      66.7%             def456...                             rack1
UN  172.20.0.4   107.8 KiB  16      66.6%             ghi789...                             rack1
```

**Interprétation :**

| Colonne | Description | Valeur attendue |
|---------|-------------|-----------------|
| **Status** | U = Up (actif) | `U` pour tous |
| **State** | N = Normal | `N` pour tous |
| **Address** | IP du nœud | 172.20.0.2/3/4 |
| **Tokens** | Nombre de tokens | 16 (par défaut) |
| **Owns** | % des données | ~33% chacun |

✅ **Cluster opérationnel** si tous les nœuds sont `UN` (Up Normal).

### Méthode 3 : Information du cluster

```bash
docker exec -it cassandra-seed nodetool describecluster
```

**Résultat :**
```
Cluster Information:
    Name: DevCluster
    Snitch: org.apache.cassandra.locator.GossipingPropertyFileSnitch
    DynamicEndPointSnitch: enabled
    Partitioner: org.apache.cassandra.dht.Murmur3Partitioner
    Schema versions:
        abc123: [172.20.0.2, 172.20.0.3, 172.20.0.4]
```

✅ Les **3 nœuds** doivent apparaître dans `Schema versions`.

---

## 🔌 Étape 5 : Connexion au cluster

### Se connecter au seed node

```bash
docker exec -it cassandra-seed cqlsh
```

**Résultat :**
```
Connected to DevCluster at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.x | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh>
```

### Vérifier la topologie depuis CQL

```sql
SELECT * FROM system.peers;
```

**Résultat :**
```
 peer         | data_center  | host_id                              | rack
--------------+--------------+--------------------------------------+-------
 172.20.0.3   | datacenter1  | def456...                            | rack1
 172.20.0.4   | datacenter1  | ghi789...                            | rack1
```

✅ Les **2 autres nœuds** sont bien reconnus.

---

## 📊 Étape 6 : Créer un keyspace avec réplication

Maintenant, créons un keyspace qui **réplique les données sur les 3 nœuds**.

### Créer le keyspace

```sql
CREATE KEYSPACE IF NOT EXISTS demo_cluster
WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 3
};
```

**Explication :**

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `class` | `NetworkTopologyStrategy` | Stratégie de réplication multi-datacenter |
| `datacenter1` | `3` | **Replication Factor** = 3 (copie sur 3 nœuds) |

**Pourquoi `NetworkTopologyStrategy` ?**
- Plus flexible que `SimpleStrategy`
- Recommandé même pour un seul datacenter
- Prêt pour l'ajout de datacenters futurs

### Utiliser le keyspace

```sql
USE demo_cluster;
```

### Créer une table

```sql
CREATE TABLE IF NOT EXISTS users (
  user_id UUID PRIMARY KEY,
  username TEXT,
  email TEXT,
  created_at TIMESTAMP
);
```

### Insérer des données

```sql
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'alice', 'alice@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'bob', 'bob@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'charlie', 'charlie@example.com', toTimestamp(now()));
```

### Vérifier les données

```sql
SELECT * FROM users;
```

---

## 🔄 Étape 7 : Tester la réplication

Les données sont-elles vraiment répliquées sur les 3 nœuds ?

### Vérifier sur le nœud 2

```bash
docker exec -it cassandra-node2 cqlsh
```

```sql
USE demo_cluster;
SELECT * FROM users;
```

**Résultat :**
```
 user_id                              | username | email               | created_at
--------------------------------------+----------+---------------------+---------------------------
 a1b2c3d4-...                         | alice    | alice@example.com   | 2024-10-31 10:30:00+0000
 e5f6g7h8-...                         | bob      | bob@example.com     | 2024-10-31 10:30:05+0000
 i9j0k1l2-...                         | charlie  | charlie@example.com | 2024-10-31 10:30:10+0000
```

✅ **Les mêmes données** apparaissent !

### Vérifier sur le nœud 3

```bash
docker exec -it cassandra-node3 cqlsh
```

```sql
USE demo_cluster;
SELECT COUNT(*) FROM users;
```

**Résultat :**
```
 count
-------
     3
```

✅ Les **3 enregistrements** sont présents sur tous les nœuds.

---

## 🛡️ Étape 8 : Tester la haute disponibilité

Que se passe-t-il si un nœud tombe ?

### Arrêter le nœud 3

```bash
docker-compose stop cassandra-node3
```

### Vérifier l'état du cluster

```bash
docker exec -it cassandra-seed nodetool status
```

**Résultat :**
```
--  Address      Load       Tokens  Owns (effective)  Host ID      Rack
UN  172.20.0.2   108.5 KiB  16      100.0%            abc123...    rack1
UN  172.20.0.3   104.2 KiB  16      100.0%            def456...    rack1
DN  172.20.0.4   107.8 KiB  16      100.0%            ghi789...    rack1
```

- `DN` = **Down Normal** (nœud arrêté mais reconnu)

### Les données sont-elles toujours accessibles ?

```bash
docker exec -it cassandra-seed cqlsh
```

```sql
USE demo_cluster;
SELECT * FROM users;
```

**Résultat :**
```
 user_id                              | username | email               | created_at
--------------------------------------+----------+---------------------+---------------------------
 a1b2c3d4-...                         | alice    | alice@example.com   | 2024-10-31 10:30:00+0000
 e5f6g7h8-...                         | bob      | bob@example.com     | 2024-10-31 10:30:05+0000
 i9j0k1l2-...                         | charlie  | charlie@example.com | 2024-10-31 10:30:10+0000
```

✅ **Toutes les données sont accessibles !** Grâce au Replication Factor = 3.

### Redémarrer le nœud 3

```bash
docker-compose start cassandra-node3
```

Attendez quelques secondes, puis vérifiez :

```bash
docker exec -it cassandra-seed nodetool status
```

**Résultat :**
```
UN  172.20.0.2   ...
UN  172.20.0.3   ...
UN  172.20.0.4   ...
```

✅ Le nœud 3 est de retour (`UN`).

---

## 📐 Concepts importants

### 1. Replication Factor (RF)

**Définition :** Nombre de copies de chaque donnée dans le cluster.

| RF | Description | Tolérance de panne |
|----|-------------|--------------------|
| 1 | Une seule copie | Aucune (perte de nœud = perte de données) |
| 2 | Deux copies | Perte d'un nœud tolérée |
| **3** | Trois copies | Perte de deux nœuds tolérée |

**Recommandation :** RF = 3 minimum en production.

### 2. Consistency Level (CL)

**Définition :** Nombre de réplicas qui doivent répondre pour une lecture/écriture.

| CL | Description | Garantie |
|----|-------------|----------|
| `ONE` | 1 nœud répond | Rapide, peu fiable |
| `QUORUM` | Majorité répond (RF/2 + 1) | Équilibré (recommandé) |
| `ALL` | Tous les nœuds répondent | Très fiable, lent |

**Par défaut :** `ONE` (pour les tests).

**Exemple avec QUORUM :**
```sql
-- Lecture avec QUORUM
CONSISTENCY QUORUM;
SELECT * FROM users;

-- Écriture avec QUORUM
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'david', 'david@example.com', toTimestamp(now()));
```

### 3. Seed Nodes

**Définition :** Nœuds de référence pour rejoindre le cluster.

- Le premier nœud à démarrer
- Les autres nœuds le contactent pour rejoindre le cluster
- **Ne pas confondre** avec "master" (Cassandra est décentralisé)

**Recommandation :** Au moins 2 seeds en production.

### 4. Tokens et Partitionnement

**Token :** Hash qui détermine sur quel nœud une donnée est stockée.

- Par défaut : 16 tokens par nœud (num_tokens=16)
- Cassandra distribue automatiquement les données

**Exemple :**
```
Donnée avec clé primaire "alice"
→ Hash = 12345678
→ Token dans la plage du nœud 2
→ Stockée sur nœud 2 (et répliquée sur nœuds 1 et 3)
```

---

## 🛠️ Étape 9 : Commandes utiles

### État détaillé d'un nœud

```bash
docker exec -it cassandra-seed nodetool info
```

### Statistiques du cluster

```bash
docker exec -it cassandra-seed nodetool status --resolve-ip
```

### Voir les keyspaces

```bash
docker exec -it cassandra-seed cqlsh -e "DESCRIBE KEYSPACES;"
```

### Distribution des données

```bash
docker exec -it cassandra-seed nodetool getendpoints demo_cluster users <user_id>
```

Remplacez `<user_id>` par un UUID réel de votre table.

**Résultat :**
```
172.20.0.2
172.20.0.3
172.20.0.4
```

✅ La donnée est bien présente sur les 3 nœuds.

### Réparer un nœud

Si un nœud a été arrêté longtemps :

```bash
docker exec -it cassandra-node3 nodetool repair
```

---

## 🧹 Étape 10 : Nettoyage complet

### Arrêter le cluster

```bash
docker-compose stop
```

### Supprimer le cluster (conteneurs + données)

```bash
docker-compose down -v
```

### Supprimer le réseau

```bash
docker network rm cassandra_cluster_net
```

### Vérification

```bash
docker ps -a | grep cassandra
docker volume ls | grep cassandra
docker network ls | grep cassandra
```

Tout doit être vide.

---

## 🚨 Troubleshooting

### Problème : Un nœud reste en état "Joining"

**Symptôme :**
```
UJ  172.20.0.3   ...
```

**Causes possibles :**
- Le seed n'est pas complètement démarré
- Problème de connectivité réseau

**Solution :**
```bash
# Vérifier les logs
docker-compose logs cassandra-node2

# Redémarrer le nœud
docker-compose restart cassandra-node2
```

### Problème : "Schema disagreement"

**Symptôme :**
```
Cluster Information:
    Schema versions:
        abc123: [172.20.0.2]
        def456: [172.20.0.3, 172.20.0.4]
```

**Solution :**
```bash
# Sur chaque nœud
docker exec -it cassandra-seed nodetool describecluster
docker exec -it cassandra-node2 nodetool describecluster
docker exec -it cassandra-node3 nodetool describecluster

# Attendre quelques minutes (propagation automatique)
# Ou forcer la synchronisation
docker exec -it cassandra-seed nodetool repair
```

### Problème : Ressources insuffisantes

**Symptôme :** Les conteneurs redémarrent en boucle.

**Solution :**
- Augmenter la RAM Docker (dans les paramètres Docker Desktop)
- Ou réduire `MAX_HEAP_SIZE` à `256M`

### Problème : Port 9042 déjà utilisé

**Solution :**
Changez le port dans `docker-compose.yml` :
```yaml
ports:
  - "9043:9042"  # Port externe différent
```

---

## 💡 Conseils pour débutants

### 1. Toujours attendre le healthcheck

Ne vous connectez pas avant que tous les nœuds soient `healthy`.

### 2. Utiliser `nodetool status` en premier

C'est votre meilleur ami pour diagnostiquer l'état du cluster.

### 3. Commencer par RF=3 et CL=QUORUM

Configuration équilibrée entre performance et fiabilité.

### 4. Monitoring des logs

Gardez un terminal avec `docker-compose logs -f` ouvert pendant le démarrage.

### 5. Documentez vos keyspaces

Créez un fichier `KEYSPACES.md` :
```markdown
# Keyspaces

## demo_cluster
- RF: 3
- Tables: users
- Usage: Démo du tutoriel
```

---

## 📚 Prochaines étapes

Maintenant que vous maîtrisez les clusters, explorez :

- **[7.4 Gestion des keyspaces](04-gestion-keyspaces.md)** : Stratégies de réplication avancées
- **[Annexe G - Comparaison des BDD](/annexes/G-comparaison-bdd.md)** : Quand utiliser Cassandra
- **[Cas pratique - Environnement de dev complet](/cas-pratiques/04-env-dev-complet.md)** : Cassandra avec applications

---

## ❓ Questions fréquentes (FAQ)

**Q : Combien de nœuds seeds recommandez-vous ?**
R : 2-3 seeds suffisent, même pour de gros clusters. Pas besoin que tous les nœuds soient seeds.

**Q : Puis-je ajouter un 4ème nœud au cluster ?**
R : Oui ! Copiez la config de `cassandra-node3`, changez l'IP (`172.20.0.5`) et lancez `docker-compose up -d`.

**Q : Quelle est la différence entre RF et nombre de nœuds ?**
R : RF = nombre de copies. Vous pouvez avoir RF=3 sur un cluster de 10 nœuds.

**Q : Les performances sont-elles multipliées par 3 ?**
R : Pas exactement. Les écritures sont plus rapides (distribution), les lectures dépendent du CL.

**Q : Cassandra est-il mieux que MongoDB pour un cluster ?**
R : Cassandra excelle pour les écritures massives et la haute disponibilité. MongoDB est plus flexible pour les requêtes complexes.

**Q : Comment migrer d'un nœud unique vers un cluster ?**
R : Exportez les données (cqlsh COPY), créez le cluster, importez les données.

---

## 🔗 Ressources complémentaires

- [Documentation Cassandra - Architecture](https://cassandra.apache.org/doc/latest/cassandra/architecture/)
- [DataStax - Understanding Replication](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/architecture/archDataDistributeReplication.html)
- [Cassandra Best Practices](https://cassandra.apache.org/doc/latest/cassandra/operating/index.html)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
