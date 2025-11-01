# 11.4 Cluster Elasticsearch simple (3 nœuds)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Un **cluster Elasticsearch** est un ensemble de nœuds (serveurs) qui travaillent ensemble pour stocker et gérer vos données. Contrairement à un nœud unique, un cluster offre :

- 🔄 **Haute disponibilité** : Si un nœud tombe, les autres continuent
- ⚡ **Performance** : Les requêtes sont distribuées sur plusieurs nœuds
- 💾 **Réplication** : Les données sont dupliquées pour éviter les pertes
- 📈 **Scalabilité** : Facile d'ajouter des nœuds pour gérer plus de données

### 🎯 Concepts clés

| Concept | Explication |
|---------|-------------|
| **Cluster** | Ensemble de nœuds travaillant ensemble |
| **Nœud** | Une instance Elasticsearch |
| **Shard** | Morceau d'un index distribué sur les nœuds |
| **Replica** | Copie d'un shard pour la redondance |
| **Master node** | Nœud qui gère le cluster |

### 📦 Ce que vous allez apprendre

Dans cette fiche, nous allons créer un **cluster de 3 nœuds** Elasticsearch avec Docker Compose, parfait pour comprendre le clustering en environnement de développement.

---

## ⚠️ Prérequis Importants

### Configuration système minimale

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ **Au moins 8 GB de RAM** disponibles
  - 2 GB par nœud × 3 nœuds = 6 GB minimum
  - + 2 GB pour le système
- ✅ **Espace disque** : Au moins 10 GB libres

> ⚠️ **Attention** : Un cluster de 3 nœuds consomme beaucoup de ressources. Assurez-vous d'avoir suffisamment de RAM avant de commencer.

### Configuration Linux (obligatoire)

Si vous êtes sur **Linux**, configurez la mémoire virtuelle :

```bash
# Vérifier la valeur actuelle
sysctl vm.max_map_count

# Si < 262144, augmentez-la
sudo sysctl -w vm.max_map_count=262144

# Rendre le changement permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---

## 🚀 Configuration du Cluster 3 Nœuds

### Étape 1 : Créer le fichier `docker-compose.yml`

Créez un nouveau dossier (par exemple `elasticsearch_cluster`) et placez-y ce fichier :

```yaml
version: '3.8'

services:
  # ===== NŒUD 1 : MASTER + DATA =====
  es-node01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es-node01
    restart: unless-stopped

    environment:
      # Nom du nœud
      - node.name=es-node01

      # Nom du cluster (tous les nœuds doivent avoir le même)
      - cluster.name=es-docker-cluster

      # Liste des nœuds éligibles comme master
      # Important : liste les 3 nœuds pour l'élection du master
      - cluster.initial_master_nodes=es-node01,es-node02,es-node03

      # Liste de tous les nœuds du cluster (pour la découverte)
      - discovery.seed_hosts=es-node02,es-node03

      # Désactive la sécurité (développement uniquement)
      - xpack.security.enabled=false

      # Mémoire : 1 GB min, 2 GB max par nœud
      - "ES_JAVA_OPTS=-Xms1g -Xmx2g"

      # Interface d'écoute
      - network.host=0.0.0.0

    ports:
      # API REST du nœud 1
      - "9201:9200"
      # Port de communication inter-nœuds
      - "9301:9300"

    volumes:
      # Données du nœud 1
      - es-data01:/usr/share/elasticsearch/data

    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

    networks:
      - elastic

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ===== NŒUD 2 : MASTER + DATA =====
  es-node02:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es-node02
    restart: unless-stopped

    environment:
      - node.name=es-node02
      - cluster.name=es-docker-cluster
      - cluster.initial_master_nodes=es-node01,es-node02,es-node03
      - discovery.seed_hosts=es-node01,es-node03
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx2g"
      - network.host=0.0.0.0

    ports:
      # API REST du nœud 2 (port différent pour accès externe)
      - "9202:9200"
      - "9302:9300"

    volumes:
      - es-data02:/usr/share/elasticsearch/data

    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

    networks:
      - elastic

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ===== NŒUD 3 : MASTER + DATA =====
  es-node03:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es-node03
    restart: unless-stopped

    environment:
      - node.name=es-node03
      - cluster.name=es-docker-cluster
      - cluster.initial_master_nodes=es-node01,es-node02,es-node03
      - discovery.seed_hosts=es-node01,es-node02
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx2g"
      - network.host=0.0.0.0

    ports:
      # API REST du nœud 3
      - "9203:9200"
      - "9303:9300"

    volumes:
      - es-data03:/usr/share/elasticsearch/data

    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

    networks:
      - elastic

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

# Volumes (un par nœud)
volumes:
  es-data01:
    driver: local
  es-data02:
    driver: local
  es-data03:
    driver: local

# Réseau partagé
networks:
  elastic:
    driver: bridge
```

### 📝 Explication des paramètres clés

#### Paramètres de clustering

| Paramètre | Explication |
|-----------|-------------|
| `cluster.name` | Nom du cluster (doit être identique sur tous les nœuds) |
| `node.name` | Nom unique de chaque nœud |
| `cluster.initial_master_nodes` | Liste des nœuds éligibles pour devenir master |
| `discovery.seed_hosts` | Liste des autres nœuds pour la découverte |

#### Ports exposés

| Nœud | Port API REST | Port Communication |
|------|---------------|-------------------|
| es-node01 | 9201 → 9200 | 9301 → 9300 |
| es-node02 | 9202 → 9200 | 9302 → 9300 |
| es-node03 | 9203 → 9200 | 9303 → 9300 |

> 💡 **Pourquoi des ports différents ?** Chaque nœud écoute sur le port 9200 *à l'intérieur* du conteneur, mais Docker les expose sur des ports différents (9201, 9202, 9203) pour éviter les conflits.

---

## ▶️ Étape 2 : Lancer le Cluster

Dans le dossier contenant votre `docker-compose.yml` :

```bash
# Démarrer les 3 nœuds
docker-compose up -d

# Voir les logs de tous les nœuds
docker-compose logs -f
```

### Ce qui se passe :

1. ⏳ **Les 3 nœuds démarrent** (chacun prend 30-60 secondes)
2. 🔍 **Découverte** : Les nœuds se trouvent mutuellement
3. 🗳️ **Élection du master** : Un nœud est élu comme master
4. ✅ **Cluster formé** : Les 3 nœuds sont connectés

⏱️ **Temps total** : environ 2-3 minutes pour que le cluster soit complètement opérationnel.

### Logs importants à surveiller

Vous verrez dans les logs :

```
[es-node01] master node changed
[es-node02] added {es-node01} to cluster
[es-node03] added {es-node02} to cluster
cluster health status changed from [YELLOW] to [GREEN]
```

✅ Quand vous voyez `status changed to [GREEN]`, le cluster est prêt !

---

## ✅ Étape 3 : Vérifier le Cluster

### Test 1 : Vérifier l'état du cluster

Vous pouvez interroger **n'importe quel nœud** :

```bash
# Via le nœud 1
curl http://localhost:9201/_cluster/health?pretty

# Via le nœud 2
curl http://localhost:9202/_cluster/health?pretty

# Via le nœud 3
curl http://localhost:9203/_cluster/health?pretty
```

**Réponse attendue** :

```json
{
  "cluster_name" : "es-docker-cluster",
  "status" : "green",
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0
}
```

| Status | Signification |
|--------|---------------|
| 🟢 `green` | Tout va bien, tous les shards sont assignés |
| 🟡 `yellow` | Fonctionnel mais des replicas manquent |
| 🔴 `red` | Problème, des données primaires manquent |

### Test 2 : Lister les nœuds du cluster

```bash
curl http://localhost:9201/_cat/nodes?v
```

**Résultat attendu** :

```
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.18.0.2           45          78   3    0.12    0.23     0.45 cdfhilmrstw *      es-node01
172.18.0.3           42          78   2    0.12    0.23     0.45 cdfhilmrstw -      es-node02
172.18.0.4           40          78   1    0.12    0.23     0.45 cdfhilmrstw -      es-node03
```

✅ Vous devriez voir **3 lignes** (un nœud par ligne).

L'**astérisque (*)** indique quel nœud est le **master** actuel.

### Test 3 : Informations détaillées du cluster

```bash
curl http://localhost:9201/_cluster/stats?human&pretty
```

Vous verrez des statistiques complètes sur le cluster.

---

## 🧪 Tester la Réplication

### Créer un index avec des replicas

```bash
# Créer un index avec 1 shard primaire et 2 replicas
curl -X PUT "http://localhost:9201/test-cluster?pretty" \
     -H 'Content-Type: application/json' \
     -d '{
       "settings": {
         "number_of_shards": 1,
         "number_of_replicas": 2
       }
     }'
```

**Explication** :

- `number_of_shards: 1` : Un seul shard primaire
- `number_of_replicas: 2` : Deux copies (une sur chaque autre nœud)

### Vérifier la distribution des shards

```bash
curl http://localhost:9201/_cat/shards/test-cluster?v
```

**Résultat attendu** :

```
index        shard prirep state   docs store ip         node
test-cluster 0     p      STARTED    0  208b 172.18.0.2 es-node01
test-cluster 0     r      STARTED    0  208b 172.18.0.3 es-node02
test-cluster 0     r      STARTED    0  208b 172.18.0.4 es-node03
```

✅ Vous voyez :
- **p** (primary) : Le shard primaire sur es-node01
- **r** (replica) : Les replicas sur es-node02 et es-node03

### Ajouter des données

```bash
# Ajouter un document via le nœud 1
curl -X POST "http://localhost:9201/test-cluster/_doc?pretty" \
     -H 'Content-Type: application/json' \
     -d '{
       "message": "Document distribué sur le cluster",
       "timestamp": "2025-11-01T10:00:00"
     }'
```

### Vérifier que les données sont disponibles sur tous les nœuds

```bash
# Recherche via le nœud 1
curl http://localhost:9201/test-cluster/_search?pretty

# Recherche via le nœud 2 (même résultat !)
curl http://localhost:9202/test-cluster/_search?pretty

# Recherche via le nœud 3 (même résultat !)
curl http://localhost:9203/test-cluster/_search?pretty
```

✅ Vous verrez le **même document** depuis n'importe quel nœud !

---

## 🔄 Simuler une Panne de Nœud

### Test de haute disponibilité

```bash
# Arrêter le nœud 2
docker-compose stop es-node02

# Vérifier l'état du cluster
curl http://localhost:9201/_cluster/health?pretty
```

**Ce qui se passe** :

1. ⚠️ Le cluster passe en **YELLOW** (replica manquant)
2. 🔄 Elasticsearch redistribue les shards
3. ✅ Les données restent **accessibles** via les nœuds 1 et 3

### Vérifier que les données sont toujours là

```bash
# Via le nœud 1 (toujours actif)
curl http://localhost:9201/test-cluster/_search?pretty

# Via le nœud 3 (toujours actif)
curl http://localhost:9203/test-cluster/_search?pretty
```

✅ Les données sont **toujours disponibles** !

### Redémarrer le nœud

```bash
# Redémarrer le nœud 2
docker-compose start es-node02

# Attendre 30 secondes, puis vérifier
curl http://localhost:9201/_cluster/health?pretty
```

✅ Le cluster repasse en **GREEN** !

---

## 🔧 Commandes Utiles

### Gestion du cluster

```bash
# Voir l'état de tous les nœuds
docker-compose ps

# Logs de tous les nœuds
docker-compose logs -f

# Logs d'un nœud spécifique
docker-compose logs -f es-node01

# Redémarrer un nœud
docker-compose restart es-node02

# Arrêter tout le cluster
docker-compose stop

# Redémarrer tout le cluster
docker-compose start
```

### Monitoring du cluster

```bash
# État général
curl http://localhost:9201/_cluster/health?pretty

# Liste des nœuds
curl http://localhost:9201/_cat/nodes?v

# Liste des index
curl http://localhost:9201/_cat/indices?v

# Distribution des shards
curl http://localhost:9201/_cat/shards?v

# Allocation des shards (détails)
curl http://localhost:9201/_cat/allocation?v

# Stats du cluster
curl http://localhost:9201/_cluster/stats?human&pretty
```

---

## 📊 Ajouter Kibana au Cluster

Vous pouvez ajouter Kibana pour visualiser votre cluster.

**Ajoutez ce service dans votre `docker-compose.yml`** :

```yaml
  # ===== KIBANA =====
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana_cluster
    restart: unless-stopped

    environment:
      # Se connecte à n'importe quel nœud (ici le nœud 1)
      - ELASTICSEARCH_HOSTS=http://es-node01:9200
      - xpack.security.enabled=false
      - SERVER_HOST=0.0.0.0

    ports:
      - "5601:5601"

    networks:
      - elastic

    depends_on:
      es-node01:
        condition: service_healthy
```

Puis relancez :

```bash
docker-compose up -d kibana
```

Accédez à Kibana : `http://localhost:5601`

Dans **Stack Monitoring**, vous pourrez voir tous vos nœuds visuellement ! 📊

---

## 🛑 Suppression Complète

Pour tout supprimer :

```bash
# 1. Arrêter et supprimer tous les conteneurs
docker-compose down

# 2. Supprimer tous les volumes (⚠️ IRRÉVERSIBLE)
docker volume rm \
  <nom_du_dossier>_es-data01 \
  <nom_du_dossier>_es-data02 \
  <nom_du_dossier>_es-data03

# Pour trouver les noms exacts :
docker volume ls | grep es-data
```

---

## 🐛 Problèmes Courants

### 1. Le cluster reste en YELLOW

**Cause** : Pas assez de nœuds pour tous les replicas.

**Exemple** : Index avec 3 replicas mais seulement 3 nœuds (impossible de placer toutes les copies).

**Solution** : Réduire le nombre de replicas :

```bash
curl -X PUT "http://localhost:9201/mon-index/_settings?pretty" \
     -H 'Content-Type: application/json' \
     -d '{
       "number_of_replicas": 2
     }'
```

### 2. Les nœuds ne se trouvent pas

**Symptôme** : `number_of_nodes` reste à 1.

**Solutions** :

```bash
# Vérifier que tous les conteneurs tournent
docker-compose ps

# Vérifier les logs
docker-compose logs | grep "discovery"

# Vérifier qu'ils sont sur le même réseau
docker network inspect <nom_du_dossier>_elastic
```

### 3. "master not discovered yet"

**Cause** : Les nœuds ne peuvent pas élire un master.

**Solution** : Vérifier la configuration `cluster.initial_master_nodes` (doit être identique sur tous les nœuds).

### 4. Manque de mémoire

**Symptôme** : Les conteneurs redémarrent en boucle.

**Solution** : Réduire la mémoire par nœud :

```yaml
- "ES_JAVA_OPTS=-Xms512m -Xmx1g"
```

### 5. "max virtual memory areas too low" (Linux)

**Solution** :

```bash
sudo sysctl -w vm.max_map_count=262144
```

---

## 📊 Tableau Récapitulatif

### Configuration du cluster

| Élément | Valeur |
|---------|--------|
| **Nombre de nœuds** | 3 |
| **Nom du cluster** | es-docker-cluster |
| **Nœuds master** | Les 3 nœuds |
| **Réplication** | Possible (jusqu'à 2 replicas) |
| **RAM totale** | ~6 GB (2 GB par nœud) |

### Ports d'accès

| Nœud | URL API REST |
|------|--------------|
| es-node01 | `http://localhost:9201` |
| es-node02 | `http://localhost:9202` |
| es-node03 | `http://localhost:9203` |

---

## 💡 Bonnes Pratiques

### Pour le développement

✅ **3 nœuds minimum** : Pour tester la haute disponibilité
✅ **Noms explicites** : `es-node01`, `es-node02`, etc.
✅ **Volumes séparés** : Un volume par nœud
✅ **Surveillez la RAM** : 8 GB minimum recommandés

### Pour la production

🔒 **Activez la sécurité** : Authentification obligatoire
🔐 **Utilisez HTTPS** : Chiffrement des communications
🎯 **Rôles dédiés** : Nœuds master / data / coordination séparés
🌐 **Plusieurs zones** : Pour la résilience géographique
📊 **Monitoring** : Kibana Stack Monitoring ou Prometheus
💾 **Snapshots** : Backups réguliers

---

## 🔍 Comprendre les Rôles de Nœuds

Dans notre configuration, tous les nœuds ont **tous les rôles** (master + data). En production, on sépare souvent :

| Rôle | Fonction |
|------|----------|
| **Master** | Gère le cluster (élections, création d'index, etc.) |
| **Data** | Stocke les données et exécute les requêtes |
| **Ingest** | Traite les données avant indexation |
| **Coordination** | Distribue les requêtes (pas de données) |

### Configuration avec rôles séparés (avancé)

```yaml
# Nœud master uniquement
- node.roles=["master"]

# Nœud data uniquement
- node.roles=["data"]

# Nœud coordination uniquement
- node.roles=[]
```

---

## 📚 Ressources Complémentaires

- 📖 [Elasticsearch Cluster Architecture](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)
- 🎓 [Clustering and Node Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html)
- 🔍 [Shard Allocation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)
- 📊 [Cluster Health](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html)

---

## 🎯 Prochaines Étapes

Maintenant que vous avez un cluster Elasticsearch, vous pouvez :

- 📊 Explorer les [cas pratiques - Stack ELK](/cas-pratiques/03-stack-elk.md)
- 🖥️ Retour à [Elasticsearch avec Kibana](03-elasticsearch-kibana.md)
- 🌐 Voir la [configuration avec IP fixe](02-config-ip-fixe.md)
- 🏠 Retour à la [configuration basique](01-config-noeud-unique.md)

---

## 💡 Points Clés à Retenir

✅ Un **cluster** améliore la disponibilité et les performances
✅ **3 nœuds minimum** pour une vraie haute disponibilité
✅ Les **replicas** protègent contre la perte de données
✅ Un cluster peut fonctionner même si **1 nœud tombe**
✅ Tous les nœuds peuvent recevoir des **requêtes API**
✅ Le **master node** est élu automatiquement
✅ **8 GB de RAM minimum** pour un cluster de 3 nœuds
✅ Le statut **GREEN** signifie que tout va bien

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)


