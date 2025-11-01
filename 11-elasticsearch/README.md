# Elasticsearch avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**Elasticsearch** est un moteur de recherche et d'analyse distribué, open source, basé sur Apache Lucene. Il permet de stocker, rechercher et analyser de grandes quantités de données en temps réel avec une vitesse et une pertinence exceptionnelles.

### 🎯 Pourquoi Elasticsearch ?

Elasticsearch se distingue par :

- ⚡ **Recherche rapide** : Résultats en millisecondes même sur des téraoctets de données
- 🔍 **Recherche full-text** : Analyse linguistique avancée, recherche floue, autocomplete
- 📊 **Analyse en temps réel** : Agrégations complexes et statistiques instantanées
- 🌐 **Distribué** : Scalabilité horizontale automatique
- 🔄 **Résilient** : Réplication et haute disponibilité intégrées
- 🛠️ **RESTful API** : Interface HTTP simple et universelle

---

## 🎨 Cas d'Usage Typiques

### 1. 🔍 Moteur de Recherche

**Applications web, e-commerce, sites de contenu**

- Recherche de produits avec filtres multiples
- Suggestions automatiques (autocomplete)
- Recherche floue et tolérante aux fautes
- Recherche multi-langue

**Exemple** : Amazon, Wikipedia, GitHub utilisent Elasticsearch pour leur recherche.

### 2. 📊 Analyse de Logs (Stack ELK)

**Centralisation et analyse de logs d'applications**

- Collecte de logs de multiples sources
- Analyse en temps réel des erreurs
- Détection d'anomalies
- Tableaux de bord de monitoring

**Stack ELK** = **E**lasticsearch + **L**ogstash + **K**ibana

### 3. 📈 Analytics et Business Intelligence

**Analyse de données métier en temps réel**

- Métriques d'utilisation d'applications
- Analyse de comportements utilisateurs
- Tableaux de bord temps réel
- Rapports et statistiques

### 4. 🛡️ Monitoring et Observabilité

**Surveillance de systèmes et applications**

- Métriques système (CPU, RAM, réseau)
- APM (Application Performance Monitoring)
- Alertes en temps réel
- Traçage distribué

### 5. 🔐 Sécurité et SIEM

**Security Information and Event Management**

- Analyse de logs de sécurité
- Détection d'intrusions
- Corrélation d'événements
- Audit et conformité

---

## 🗄️ Architecture et Concepts Clés

### Concepts Fondamentaux

| Concept | Explication | Équivalent SQL |
|---------|-------------|----------------|
| **Index** | Collection de documents similaires | Base de données |
| **Document** | Unité de base des données (JSON) | Ligne / Enregistrement |
| **Field** | Attribut d'un document | Colonne |
| **Mapping** | Définition de la structure des documents | Schéma |
| **Shard** | Fragment d'un index distribué | Partition |
| **Replica** | Copie d'un shard pour la redondance | Réplication |

### Architecture d'un Cluster

```
┌─────────────────────────────────────────────────────┐
│                    CLUSTER                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Nœud 1    │  │   Nœud 2    │  │   Nœud 3    │ │
│  │   (Master)  │  │             │  │             │ │
│  │             │  │             │  │             │ │
│  │  Shard P1   │  │  Shard R1   │  │  Shard R1   │ │
│  │  Shard P2   │  │  Shard P3   │  │  Shard R2   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────┘
         P = Primary (principal)
         R = Replica (copie)
```

### Types de Nœuds

| Type | Rôle | Usage |
|------|------|-------|
| **Master** | Gestion du cluster (création d'index, allocation de shards) | Léger en ressources |
| **Data** | Stockage des données et exécution des requêtes | Intensif (RAM, CPU, I/O) |
| **Ingest** | Traitement des données avant indexation | CPU intensif |
| **Coordination** | Distribution des requêtes (pas de données) | Réseau intensif |

---

## 📦 Versions d'Elasticsearch

### Versions Majeures

| Version | Date | Nouveautés Principales |
|---------|------|------------------------|
| **8.x** | 2022+ | Sécurité par défaut, amélioration des performances |
| **7.x** | 2019-2022 | Nouvelle architecture de recherche, SQL |
| **6.x** | 2017-2019 | Types dépréciés, amélioration clustering |
| **5.x** | 2016-2017 | Ingest nodes, Painless scripting |

> 💡 **Recommandation** : Utilisez toujours la dernière version de la branche 8.x pour bénéficier des dernières fonctionnalités et corrections de sécurité.

---

## 🚀 Elasticsearch avec Docker

### Avantages de Docker pour Elasticsearch

✅ **Installation rapide** : Pas de dépendances Java à gérer
✅ **Isolation** : Pas de conflit avec d'autres services
✅ **Reproductibilité** : Même environnement partout
✅ **Clustering facile** : Plusieurs nœuds sur une seule machine
✅ **Nettoyage simple** : Suppression propre sans traces

### Prérequis Techniques

#### Configuration Minimale

- **RAM** : 4 GB minimum (2 GB pour Elasticsearch)
- **CPU** : 2 cœurs minimum
- **Disque** : 10 GB d'espace libre
- **OS** : Linux, macOS, Windows avec Docker Desktop

#### Configuration Recommandée

- **RAM** : 8 GB ou plus
- **CPU** : 4 cœurs ou plus
- **Disque** : SSD avec 50+ GB libres
- **OS** : Linux (meilleures performances)

### Configuration Système Requise

#### Linux : Mémoire Virtuelle

Elasticsearch nécessite une augmentation de la limite de mémoire virtuelle :

```bash
# Vérifier la valeur actuelle
sysctl vm.max_map_count

# Augmenter temporairement
sudo sysctl -w vm.max_map_count=262144

# Rendre permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

#### macOS / Windows avec Docker Desktop

Docker Desktop gère automatiquement cette configuration. Assurez-vous simplement d'allouer suffisamment de RAM à Docker (minimum 4 GB).

---

## 📚 Fiches Disponibles

Ce dossier contient **4 fiches pratiques** pour maîtriser Elasticsearch avec Docker :

### 🔰 Niveau Débutant

#### [11.1 - Configuration basique d'un nœud unique](01-config-noeud-unique.md)

**Ce que vous apprendrez :**
- Démarrer Elasticsearch en moins de 5 minutes
- Configuration Docker Compose minimale
- Vérifications de base
- Premières requêtes HTTP

**Idéal pour :** Premiers pas, développement simple, tests rapides

---

#### [11.2 - Configuration avec IP fixe](02-config-ip-fixe.md)

**Ce que vous apprendrez :**
- Créer un réseau Docker dédié
- Assigner une IP statique au conteneur
- Communication inter-conteneurs stable
- Configuration réseau avancée

**Idéal pour :** Applications multi-conteneurs, environnements de test prévisibles

---

### 🔧 Niveau Intermédiaire

#### [11.3 - Elasticsearch avec Kibana](03-elasticsearch-kibana.md)

**Ce que vous apprendrez :**
- Déployer Elasticsearch + Kibana ensemble
- Utiliser l'interface graphique Kibana
- Dev Tools : console pour requêtes
- Discover : exploration visuelle des données
- Créer des visualisations

**Idéal pour :** Visualisation de données, analyse, tableaux de bord

---

### 🎓 Niveau Avancé

#### [11.4 - Cluster simple (3 nœuds)](04-cluster-simple.md)

**Ce que vous apprendrez :**
- Créer un cluster de 3 nœuds
- Réplication et haute disponibilité
- Tester la tolérance aux pannes
- Distribution des shards
- Monitoring de cluster

**Idéal pour :** Comprendre le clustering, tests de résilience, environnements avancés

---

## 🛠️ L'Écosystème Elastic Stack

Elasticsearch fait partie d'un écosystème plus large appelé **Elastic Stack** (anciennement ELK Stack) :

### Composants Principaux

#### 1. 🔍 Elasticsearch
Le moteur de recherche et d'analyse (cœur du stack)

#### 2. 📊 Kibana
Interface de visualisation et d'exploration

- Tableaux de bord interactifs
- Visualisations (graphiques, cartes, etc.)
- Dev Tools pour requêtes
- Management du cluster

#### 3. 📥 Logstash
Pipeline de traitement de données

- Collecte de logs depuis multiples sources
- Transformation et enrichissement
- Envoi vers Elasticsearch

#### 4. 📡 Beats
Collecteurs légers de données

- **Filebeat** : Logs de fichiers
- **Metricbeat** : Métriques système
- **Packetbeat** : Analyse réseau
- **Heartbeat** : Monitoring de disponibilité

### Architecture Typique (Stack ELK)

```
┌─────────────┐
│ Application │
│   (Logs)    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Filebeat   │ ← Collecte les logs
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Logstash   │ ← Traite et enrichit
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Elasticsearch│ ← Indexe et stocke
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Kibana    │ ← Visualise
└─────────────┘
```

---

## 💡 Quand Utiliser Elasticsearch ?

### ✅ Elasticsearch est IDÉAL pour :

- 🔍 Recherche full-text complexe
- 📊 Analyse en temps réel de logs
- 📈 Agrégations statistiques sur gros volumes
- 🗺️ Recherche géospatiale
- 📝 Données semi-structurées (JSON)
- ⚡ Besoin de performances sur des requêtes complexes

### ❌ Elasticsearch N'EST PAS recommandé pour :

- 💾 Stockage primaire de données critiques (utiliser PostgreSQL, MongoDB)
- 🔐 Transactions ACID strictes
- 📊 Relations complexes entre données (préférer SQL)
- 💰 Petits volumes de données (< 10 000 documents)
- 🎯 Données hautement structurées sans recherche (utiliser SQL)

### 🤝 Utilisez AVEC une autre base de données

**Pattern recommandé** : Base de données primaire + Elasticsearch pour la recherche

```
┌──────────────┐         ┌──────────────┐
│  PostgreSQL  │ ──sync──│Elasticsearch │
│   (Source)   │         │  (Recherche) │
└──────────────┘         └──────────────┘
        │                        │
        ▼                        ▼
   CRUD complet           Recherche rapide
   Transactions           Analyse
   Relations              Full-text
```

---

## 📖 Ressources Complémentaires

### Documentation Officielle

- 📚 [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- 🎓 [Getting Started](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
- 🔍 [Search APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html)
- 📊 [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)

### Images Docker Officielles

- 🐳 [Elasticsearch sur Docker Hub](https://www.docker.elastic.co/r/elasticsearch)
- 📦 [Kibana sur Docker Hub](https://www.docker.elastic.co/r/kibana)

### Communauté et Support

- 💬 [Forum Elastic](https://discuss.elastic.co/)
- 🐙 [GitHub Elasticsearch](https://github.com/elastic/elasticsearch)
- 📺 [Elastic YouTube](https://www.youtube.com/c/Elastic)

### Cours et Tutoriels

- 🎓 [Elastic Certified Engineer](https://www.elastic.co/training/certification)
- 📖 [Awesome Elasticsearch](https://github.com/dzharii/awesome-elasticsearch)

---

## ⚙️ Commandes Essentielles

### Vérifications Rapides

```bash
# État du cluster
curl http://localhost:9200/_cluster/health?pretty

# Informations générales
curl http://localhost:9200/

# Liste des nœuds
curl http://localhost:9200/_cat/nodes?v

# Liste des index
curl http://localhost:9200/_cat/indices?v

# Statistiques du cluster
curl http://localhost:9200/_cluster/stats?pretty
```

### Opérations de Base

```bash
# Créer un index
curl -X PUT "http://localhost:9200/mon-index"

# Ajouter un document
curl -X POST "http://localhost:9200/mon-index/_doc" \
     -H 'Content-Type: application/json' \
     -d '{"titre": "Mon document", "contenu": "Contenu ici"}'

# Rechercher
curl http://localhost:9200/mon-index/_search?q=titre:Mon

# Supprimer un index
curl -X DELETE "http://localhost:9200/mon-index"
```

---

## 🔐 Sécurité

### ⚠️ IMPORTANT pour le Développement

Les configurations présentées dans ce dossier ont la **sécurité désactivée** (`xpack.security.enabled=false`) pour simplifier l'apprentissage.

**Cela signifie :**
- ❌ Pas d'authentification
- ❌ Pas de chiffrement
- ❌ Accès libre à toutes les données

### ✅ Pour la Production

**TOUJOURS activer :**

1. **Authentification** : Utilisateurs et mots de passe
2. **Chiffrement TLS/SSL** : Communications sécurisées
3. **Contrôle d'accès** : Permissions par utilisateur/rôle
4. **Audit logging** : Traçabilité des actions
5. **Pare-feu** : Restrictions réseau

**Configuration sécurisée minimale** :

```yaml
environment:
  - xpack.security.enabled=true
  - xpack.security.transport.ssl.enabled=true
  - xpack.security.http.ssl.enabled=true
  - ELASTIC_PASSWORD=VotreMotDePasseRobuste123!
```

> 📖 Consultez l'[Annexe D - Sécurité](/annexes/D-securite-bonnes-pratiques.md) pour plus de détails.

---

## 🎯 Par Où Commencer ?

### 1. 🔰 Vous Débutez Complètement

➡️ Commencez par **[11.1 - Configuration basique](01-config-noeud-unique.md)**

Vous apprendrez à démarrer votre premier nœud Elasticsearch et à effectuer vos premières requêtes.

### 2. 🧪 Vous Voulez Tester avec une Interface Graphique

➡️ Allez directement à **[11.3 - Elasticsearch avec Kibana](03-elasticsearch-kibana.md)**

Vous aurez Elasticsearch + une interface web pour visualiser vos données.

### 3. 🎓 Vous Voulez Comprendre le Clustering

➡️ Suivez **toutes les fiches dans l'ordre** (11.1 → 11.2 → 11.3 → 11.4)

Vous aurez une compréhension complète d'Elasticsearch du déploiement simple au cluster.

### 4. 🚀 Vous Voulez une Stack Complète

➡️ Consultez le **[Cas Pratique - Stack ELK](/cas-pratiques/03-stack-elk.md)**

Vous déploierez Elasticsearch + Logstash + Kibana pour une solution complète.

---

## 💡 Conseils pour Bien Démarrer

### ✅ Bonnes Pratiques

1. **Commencez simple** : Un nœud unique suffit pour apprendre
2. **Utilisez Kibana** : L'interface graphique aide énormément
3. **Testez avec de vraies données** : Plus instructif que des données fictives
4. **Surveillez la RAM** : Elasticsearch est gourmand en mémoire
5. **Lisez les logs** : Ils sont très informatifs en cas de problème

### 🎓 Progression Recommandée

```
Semaine 1 : Nœud unique + premières requêtes
            ↓
Semaine 2 : Kibana + visualisations
            ↓
Semaine 3 : Index, mapping, recherche avancée
            ↓
Semaine 4 : Clustering et haute disponibilité
```

---

## 📊 Comparaison avec d'Autres Solutions

| Fonctionnalité | Elasticsearch | PostgreSQL | MongoDB | Solr |
|----------------|---------------|------------|---------|------|
| **Recherche full-text** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Analyse temps réel** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Scalabilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Facilité d'utilisation** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Transactions ACID** | ❌ | ✅ | ⚠️ | ❌ |
| **Requêtes SQL** | ⚠️ (basique) | ✅ | ❌ | ⚠️ |

---

## 🎉 Prêt à Commencer ?

**Choisissez votre fiche et lancez-vous !**

1. 🔰 [Configuration basique d'un nœud unique](01-config-noeud-unique.md)
2. 🌐 [Configuration avec IP fixe](02-config-ip-fixe.md)
3. 🖥️ [Elasticsearch avec Kibana](03-elasticsearch-kibana.md)
4. 🔗 [Cluster simple (3 nœuds)](04-cluster-simple.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)


