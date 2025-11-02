# Stack ELK avec Docker (Elasticsearch + Logstash + Kibana)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous guide dans la création d'une **stack ELK complète** avec Docker. ELK est l'acronyme de **Elasticsearch + Logstash + Kibana**, une solution puissante et populaire pour la collecte, le stockage, l'analyse et la visualisation de données de logs en temps réel.

**Ce que vous allez apprendre :**
- Comprendre l'architecture et le rôle de chaque composant ELK
- Déployer Elasticsearch, Logstash et Kibana avec Docker Compose
- Configurer la collecte et l'indexation de logs
- Créer des visualisations et des tableaux de bord dans Kibana
- Gérer et interroger vos données avec Elasticsearch
- Suivre les bonnes pratiques de configuration

**Durée estimée :** 40-50 minutes

---

## 🎯 Qu'est-ce que la Stack ELK ?

### Définition

La **stack ELK** est une suite d'outils open source développée par Elastic pour l'analyse centralisée de logs. Elle permet de collecter, stocker, rechercher et visualiser des données de logs provenant de multiples sources (applications, serveurs, containers, etc.).

### Les 3 composants principaux

| Composant | Rôle | Fonction |
|-----------|------|----------|
| **E**lasticsearch | Moteur de recherche | Stocke et indexe les données, permet des recherches ultra-rapides |
| **L**ogstash | Pipeline de traitement | Collecte, transforme et envoie les données vers Elasticsearch |
| **K**ibana | Interface de visualisation | Permet de visualiser et d'explorer les données via des graphiques et dashboards |

### Schéma de fonctionnement

```
┌──────────────────────────────────────────────────────────────┐
│                   SOURCES DE DONNÉES                         │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌────────────┐ │
│  │  Serveur  │  │   Docker  │  │   Apache  │  │Application │ │
│  │   Web     │  │Containers │  │   Logs    │  │   Logs     │ │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └────┬───────┘ │
└────────┼──────────────┼──────────────┼─────────────┼─────────┘
         │              │              │             │
         │              │  Logs        │             │
         └──────────────┼──────────────┼─────────────┘
                        │              │
                        ▼              ▼
         ┌──────────────────────────────────────────┐
         │        LOGSTASH (Port 5000)              │
         │  ┌────────────────────────────────────┐  │
         │  │  1. Collecte (Input)               │  │
         │  │  2. Transformation (Filter)        │  │
         │  │  3. Enrichissement                 │  │
         │  │  4. Envoi vers Elasticsearch       │  │
         │  └────────────────────────────────────┘  │
         └─────────────────┬────────────────────────┘
                           │
                           │ Index JSON
                           │
                           ▼
         ┌──────────────────────────────────────────┐
         │    ELASTICSEARCH (Port 9200)             │
         │  ┌────────────────────────────────────┐  │
         │  │  • Indexation des documents        │  │
         │  │  • Stockage distribué              │  │
         │  │  • Recherche full-text             │  │
         │  │  • Agrégations et analytics        │  │
         │  └────────────────────────────────────┘  │
         └─────────────────┬────────────────────────┘
                           │
                           │ API REST
                           │
                           ▼
         ┌──────────────────────────────────────────┐
         │        KIBANA (Port 5601)                │
         │  ┌────────────────────────────────────┐  │
         │  │  • Dashboards interactifs          │  │
         │  │  • Graphiques et visualisations    │  │
         │  │  • Recherche et exploration        │  │
         │  │  • Alertes et monitoring           │  │
         │  └────────────────────────────────────┘  │
         └──────────────────────────────────────────┘
                           │
                           │ Interface Web
                           ▼
         ┌──────────────────────────────────────────┐
         │         NAVIGATEUR                       │
         │     (http://localhost:5601)              │
         └──────────────────────────────────────────┘
```

### Pourquoi utiliser la stack ELK ?

| Avantage | Explication |
|----------|-------------|
| **Centralisation** | Tous vos logs au même endroit, peu importe leur source |
| **Recherche puissante** | Elasticsearch permet des recherches full-text ultra-rapides |
| **Scalabilité** | Peut gérer des téraoctets de données |
| **Temps réel** | Analyse et visualisation en temps réel |
| **Open source** | Gratuit et personnalisable |
| **Écosystème riche** | Nombreuses intégrations (Beats, plugins...) |
| **Visualisations** | Dashboards interactifs et graphiques variés |

### Cas d'usage typiques

| Use Case | Description |
|----------|-------------|
| **Monitoring applicatif** | Surveiller les erreurs et performances d'applications |
| **Sécurité (SIEM)** | Détecter les tentatives d'intrusion et anomalies |
| **Analyse business** | Analytics sur les comportements utilisateurs |
| **DevOps** | Debugging et troubleshooting d'infrastructures |
| **Conformité** | Audit et traçabilité des événements |
| **IoT** | Analyse de données de capteurs en temps réel |

---

## 🔧 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ Docker installé (version 20.10+)
- ✅ Docker Compose installé (version 2.0+)
- ✅ **Au moins 4 GB de RAM disponible** pour Docker (8 GB recommandé)
- ✅ Un éditeur de texte (VS Code recommandé)
- ✅ Avoir lu les [concepts Docker de base](../00-introduction/02-concepts-docker.md)

**Vérification rapide :**

```bash
docker --version
docker-compose --version

# Vérifier la mémoire allouée à Docker
docker info | grep Memory
```

**⚠️ Important :** Elasticsearch nécessite beaucoup de mémoire. Si Docker n'a que 2 GB, la stack ne démarrera pas correctement.

**Sur Windows/macOS (Docker Desktop) :**
1. Ouvrir Docker Desktop → Settings → Resources
2. Augmenter la RAM à au moins 4 GB (idéalement 8 GB)
3. Cliquer sur "Apply & Restart"

---

## 📁 Étape 1 : Structure du projet

### 1.1 Créer l'arborescence

Créez un nouveau dossier pour votre projet ELK :

```bash
# Créer le dossier principal
mkdir elk-stack
cd elk-stack

# Créer les sous-dossiers
mkdir -p elasticsearch/data
mkdir -p logstash/pipeline
mkdir -p logstash/config
mkdir -p kibana/config
mkdir -p logs/sample
```

**Votre structure sera :**

```
elk-stack/
├── docker-compose.yml              # Configuration Docker Compose
├── .env                            # Variables d'environnement
├── elasticsearch/
│   └── data/                       # Données Elasticsearch (généré automatiquement)
├── logstash/
│   ├── config/
│   │   └── logstash.yml           # Configuration Logstash
│   └── pipeline/
│       └── logstash.conf          # Pipeline de traitement
├── kibana/
│   └── config/
│       └── kibana.yml             # Configuration Kibana
└── logs/
    └── sample/
        └── app.log                # Logs d'exemple (à créer)
```

### 1.2 Pourquoi cette organisation ?

| Dossier | Rôle |
|---------|------|
| `elasticsearch/data/` | Stockage persistant des index et données |
| `logstash/pipeline/` | Définition du pipeline de traitement des logs |
| `logstash/config/` | Configuration générale de Logstash |
| `kibana/config/` | Configuration de Kibana |
| `logs/sample/` | Logs d'exemple pour tester la stack |

---

## 🔍 Étape 2 : Configuration d'Elasticsearch

### 2.1 Qu'est-ce qu'Elasticsearch ?

**Elasticsearch** est un moteur de recherche et d'analyse distribué basé sur Apache Lucene. Il stocke les données sous forme de **documents JSON** dans des **index** (équivalent des tables SQL).

**Concepts clés :**

| Concept | Équivalent SQL | Description |
|---------|----------------|-------------|
| **Index** | Base de données | Collection de documents similaires |
| **Document** | Ligne | Unité de données en JSON |
| **Field** | Colonne | Propriété d'un document |
| **Mapping** | Schéma | Définit les types de données |
| **Shard** | Partition | Division d'un index pour la scalabilité |

### 2.2 Configuration dans docker-compose.yml

Nous allons commencer par créer le fichier `docker-compose.yml` avec Elasticsearch :

```yaml
version: '3.8'

services:
  # ==========================================
  # SERVICE 1 : ELASTICSEARCH
  # ==========================================
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    container_name: elk_elasticsearch
    restart: unless-stopped

    environment:
      # Mode single-node (un seul nœud, pour développement)
      - discovery.type=single-node

      # Désactiver la sécurité (pour développement uniquement)
      - xpack.security.enabled=false
      - xpack.security.enrollment.enabled=false

      # Limiter l'utilisation de mémoire
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"

      # Désactiver le machine learning (économise de la RAM)
      - xpack.ml.enabled=false

    ports:
      # API REST d'Elasticsearch
      - "9200:9200"
      # Communication inter-nœuds (non utilisé en single-node)
      - "9300:9300"

    volumes:
      # Données persistantes
      - ./elasticsearch/data:/usr/share/elasticsearch/data

    networks:
      - elk_network

    # Vérification de santé
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

networks:
  elk_network:
    driver: bridge
```

### 2.3 Comprendre la configuration

| Paramètre | Explication |
|-----------|-------------|
| `discovery.type=single-node` | Mode standalone (pas de cluster) |
| `xpack.security.enabled=false` | Désactive l'authentification (développement uniquement) |
| `ES_JAVA_OPTS=-Xms512m -Xmx512m` | Limite la RAM : minimum 512 MB, maximum 512 MB |
| `xpack.ml.enabled=false` | Désactive le machine learning pour économiser des ressources |
| Port `9200` | API REST pour les requêtes HTTP |
| Port `9300` | Communication entre nœuds Elasticsearch (non utilisé ici) |

**💡 Note sur la mémoire :**
- Production : Au moins 2-4 GB de heap JVM
- Développement : 512 MB suffit pour débuter
- La règle : heap JVM = 50% de la RAM disponible (max 32 GB)

---

## 🔄 Étape 3 : Configuration de Logstash

### 3.1 Qu'est-ce que Logstash ?

**Logstash** est un pipeline de traitement de données open source. Il peut ingérer des données de multiples sources, les transformer, et les envoyer vers diverses destinations.

**Architecture d'un pipeline Logstash :**

```
INPUT → FILTER → OUTPUT
  ↓        ↓         ↓
Collecte  Traite  Envoie
```

**Étapes du pipeline :**

| Étape | Rôle | Exemples |
|-------|------|----------|
| **INPUT** | Collecte les données | Files, Syslog, HTTP, Beats, TCP, JDBC |
| **FILTER** | Transforme et enrichit | Parse, Grok, Mutate, Date, GeoIP |
| **OUTPUT** | Envoie vers destination | Elasticsearch, File, Kafka, S3 |

### 3.2 Créer le fichier de configuration principal

Créez le fichier `logstash/config/logstash.yml` :

```yaml
# ==========================================
# Configuration Logstash
# ==========================================

# Hôte HTTP API (pour monitoring)
http.host: "0.0.0.0"

# Elasticsearch output settings
xpack.monitoring.enabled: false

# Pipeline settings
pipeline.workers: 2
pipeline.batch.size: 125
pipeline.batch.delay: 50

# Logging
log.level: info
```

### 3.3 Créer le pipeline de traitement

Créez le fichier `logstash/pipeline/logstash.conf` :

```ruby
# ==========================================
# PIPELINE LOGSTASH - Traitement de logs
# ==========================================

input {
  # ==========================================
  # INPUT 1 : Lecture de fichiers de logs
  # ==========================================
  file {
    # Chemin vers les fichiers de logs à monitorer
    path => "/usr/share/logstash/logs/sample/*.log"

    # Commencer à lire depuis le début du fichier
    start_position => "beginning"

    # Suivre les nouveaux logs en temps réel
    sincedb_path => "/dev/null"

    # Type de log (pour identification)
    type => "application_log"
  }

  # ==========================================
  # INPUT 2 : Écoute TCP (optionnel)
  # ==========================================
  tcp {
    port => 5000
    codec => json_lines
    type => "tcp_json"
  }

  # ==========================================
  # INPUT 3 : HTTP (optionnel)
  # ==========================================
  http {
    port => 8080
    type => "http_json"
  }
}

filter {
  # ==========================================
  # FILTER 1 : Parsing des logs applicatifs
  # ==========================================
  if [type] == "application_log" {
    # Parser les logs au format : [TIMESTAMP] [LEVEL] [COMPONENT] Message
    grok {
      match => {
        "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] \[%{LOGLEVEL:level}\] \[%{DATA:component}\] %{GREEDYDATA:log_message}"
      }
    }

    # Convertir le timestamp en date Elasticsearch
    date {
      match => ["timestamp", "ISO8601"]
      target => "@timestamp"
    }

    # Ajouter des champs supplémentaires
    mutate {
      add_field => {
        "environment" => "development"
        "stack" => "elk"
      }

      # Supprimer le champ brut "message" (déjà parsé)
      remove_field => ["message"]
    }
  }

  # ==========================================
  # FILTER 2 : Enrichissement pour logs TCP/HTTP
  # ==========================================
  if [type] in ["tcp_json", "http_json"] {
    # Les données JSON sont déjà structurées
    # Ajouter juste des métadonnées
    mutate {
      add_field => {
        "source_type" => "%{type}"
      }
    }
  }

  # ==========================================
  # FILTER 3 : Géolocalisation IP (optionnel)
  # ==========================================
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
}

output {
  # ==========================================
  # OUTPUT 1 : Elasticsearch
  # ==========================================
  elasticsearch {
    # Adresse d'Elasticsearch (nom du service Docker)
    hosts => ["http://elasticsearch:9200"]

    # Index dynamique basé sur le type et la date
    # Exemple : logs-application_log-2024.11.02
    index => "logs-%{type}-%{+YYYY.MM.dd}"

    # Créer l'index s'il n'existe pas
    manage_template => true
  }

  # ==========================================
  # OUTPUT 2 : Console (pour debug)
  # ==========================================
  stdout {
    codec => rubydebug
  }
}
```

### 3.4 Comprendre le pipeline

#### Input (Collecte)

```ruby
file {
  path => "/usr/share/logstash/logs/sample/*.log"
  start_position => "beginning"
}
```
- `path` : Chemin des fichiers à monitorer (supporte les wildcards `*`)
- `start_position` : `beginning` (lire depuis le début) ou `end` (nouveaux logs seulement)
- `sincedb_path` : Fichier qui mémorise la position de lecture (ici `/dev/null` pour toujours relire)

#### Filter (Transformation)

**Plugin Grok :**
```ruby
grok {
  match => {
    "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\] \[%{LOGLEVEL:level}\]..."
  }
}
```
- **Grok** : Langage de pattern matching pour extraire des données structurées de texte non structuré
- Patterns prédéfinis : `TIMESTAMP_ISO8601`, `LOGLEVEL`, `IP`, `NUMBER`, etc.

**Plugin Date :**
```ruby
date {
  match => ["timestamp", "ISO8601"]
  target => "@timestamp"
}
```
- Parse une date et l'assigne au champ `@timestamp` (timestamp Elasticsearch)

**Plugin Mutate :**
```ruby
mutate {
  add_field => { "environment" => "development" }
  remove_field => ["message"]
}
```
- Ajoute, supprime ou modifie des champs

#### Output (Destination)

```ruby
elasticsearch {
  hosts => ["http://elasticsearch:9200"]
  index => "logs-%{type}-%{+YYYY.MM.dd}"
}
```
- Envoie vers Elasticsearch
- `index` : Nom de l'index (avec variables dynamiques)
- `%{type}` : Valeur du champ `type`
- `%{+YYYY.MM.dd}` : Date au format année.mois.jour

### 3.5 Ajouter Logstash au docker-compose.yml

Ajoutez ce service dans `docker-compose.yml` :

```yaml
  # ==========================================
  # SERVICE 2 : LOGSTASH
  # ==========================================
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.1
    container_name: elk_logstash
    restart: unless-stopped

    # Attendre qu'Elasticsearch soit prêt
    depends_on:
      elasticsearch:
        condition: service_healthy

    environment:
      # Configuration JVM
      - "LS_JAVA_OPTS=-Xms256m -Xmx256m"

      # Elasticsearch output
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

    ports:
      # HTTP input
      - "8080:8080"
      # TCP input
      - "5000:5000"
      # Monitoring API
      - "9600:9600"

    volumes:
      # Pipeline de traitement
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro

      # Configuration
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro

      # Logs à analyser
      - ./logs/sample:/usr/share/logstash/logs/sample:ro

    networks:
      - elk_network
```

---

## 📊 Étape 4 : Configuration de Kibana

### 4.1 Qu'est-ce que Kibana ?

**Kibana** est l'interface de visualisation de la stack Elastic. C'est une application web qui permet de :
- Créer des dashboards interactifs
- Effectuer des recherches dans Elasticsearch
- Créer des graphiques et visualisations
- Configurer des alertes
- Gérer les index Elasticsearch

**Fonctionnalités principales :**

| Feature | Description |
|---------|-------------|
| **Discover** | Explorer et rechercher dans les données |
| **Visualize** | Créer des graphiques (barres, lignes, camemberts, etc.) |
| **Dashboard** | Assembler plusieurs visualisations |
| **Canvas** | Créer des présentations personnalisées |
| **Maps** | Visualiser des données géographiques |
| **Alerts** | Configurer des notifications |

### 4.2 Créer le fichier de configuration

Créez le fichier `kibana/config/kibana.yml` :

```yaml
# ==========================================
# Configuration Kibana
# ==========================================

# Nom du serveur (visible dans l'interface)
server.name: "kibana-elk"

# Hôte sur lequel Kibana écoute
server.host: "0.0.0.0"

# URL d'Elasticsearch
elasticsearch.hosts: ["http://elasticsearch:9200"]

# Désactiver la sécurité (développement uniquement)
xpack.security.enabled: false
xpack.encryptedSavedObjects.encryptionKey: "fhjskloppd678ehkdfdlliverpoolfcr"

# Langue de l'interface
i18n.locale: "fr"

# Monitoring
monitoring.ui.enabled: true
monitoring.kibana.collection.enabled: false

# Logs
logging.dest: stdout
logging.verbose: false
```

### 4.3 Ajouter Kibana au docker-compose.yml

Ajoutez ce service dans `docker-compose.yml` :

```yaml
  # ==========================================
  # SERVICE 3 : KIBANA
  # ==========================================
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.1
    container_name: elk_kibana
    restart: unless-stopped

    # Attendre qu'Elasticsearch soit prêt
    depends_on:
      elasticsearch:
        condition: service_healthy

    ports:
      # Interface web Kibana
      - "5601:5601"

    volumes:
      # Configuration personnalisée
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro

    networks:
      - elk_network

    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5601/api/status || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
```

---

## 📝 Étape 5 : Créer des logs d'exemple

### 5.1 Créer un fichier de logs

Créez le fichier `logs/sample/app.log` avec du contenu d'exemple :

```log
[2024-11-02T10:00:00.123Z] [INFO] [AuthService] User login successful - userId: 12345, ip: 192.168.1.100
[2024-11-02T10:00:05.456Z] [INFO] [PaymentService] Payment processed - orderId: ORD-98765, amount: 99.99
[2024-11-02T10:00:10.789Z] [WARN] [DatabaseService] Slow query detected - duration: 3500ms, query: SELECT * FROM users
[2024-11-02T10:00:15.234Z] [ERROR] [EmailService] Failed to send email - recipient: user@example.com, error: SMTP timeout
[2024-11-02T10:00:20.567Z] [INFO] [CacheService] Cache cleared - keys: 150, duration: 250ms
[2024-11-02T10:00:25.890Z] [DEBUG] [APIGateway] Request received - method: POST, endpoint: /api/users, responseTime: 45ms
[2024-11-02T10:00:30.123Z] [ERROR] [FileService] File not found - path: /uploads/image.jpg, userId: 67890
[2024-11-02T10:00:35.456Z] [INFO] [NotificationService] Push notification sent - userId: 54321, deviceId: DEV-ABC123
[2024-11-02T10:00:40.789Z] [WARN] [SecurityService] Suspicious activity detected - ip: 203.0.113.50, attempts: 5
[2024-11-02T10:00:45.012Z] [INFO] [BackupService] Backup completed successfully - size: 2.5GB, duration: 180s
[2024-11-02T10:00:50.345Z] [ERROR] [APIGateway] Rate limit exceeded - ip: 198.51.100.25, endpoint: /api/search
[2024-11-02T10:00:55.678Z] [INFO] [SchedulerService] Cron job executed - job: clean-temp-files, status: success
[2024-11-02T10:01:00.901Z] [DEBUG] [ValidationService] Input validation passed - field: email, value: test@example.com
[2024-11-02T10:01:05.234Z] [WARN] [MemoryMonitor] High memory usage detected - usage: 85%, threshold: 80%
[2024-11-02T10:01:10.567Z] [INFO] [UserService] User profile updated - userId: 11111, fields: [name, avatar]
[2024-11-02T10:01:15.890Z] [ERROR] [DatabaseService] Connection pool exhausted - active: 100, max: 100
[2024-11-02T10:01:20.123Z] [INFO] [SearchService] Search query executed - term: "docker tutorial", results: 1250, duration: 85ms
[2024-11-02T10:01:25.456Z] [WARN] [SessionService] Session expired - sessionId: SESS-XYZ789, userId: 22222
[2024-11-02T10:01:30.789Z] [INFO] [AnalyticsService] Event tracked - event: page_view, page: /products, userId: 33333
[2024-11-02T10:01:35.012Z] [DEBUG] [CacheService] Cache hit - key: user:12345:profile, ttl: 3600s
```

### 5.2 Comprendre le format des logs

**Format utilisé :**
```
[TIMESTAMP] [LEVEL] [COMPONENT] Message détaillé
```

**Éléments :**
- `[TIMESTAMP]` : Date et heure au format ISO8601
- `[LEVEL]` : Niveau de log (INFO, WARN, ERROR, DEBUG)
- `[COMPONENT]` : Service ou composant qui génère le log
- `Message` : Description de l'événement

Ce format correspond au pattern Grok défini dans notre pipeline Logstash.

---

## 🐳 Étape 6 : Fichier docker-compose.yml complet

Voici le fichier `docker-compose.yml` complet avec les 3 services :

```yaml
version: '3.8'

services:
  # ==========================================
  # SERVICE 1 : ELASTICSEARCH
  # ==========================================
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    container_name: elk_elasticsearch
    restart: unless-stopped

    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - xpack.security.enrollment.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.ml.enabled=false

    ports:
      - "9200:9200"
      - "9300:9300"

    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data

    networks:
      - elk_network

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ==========================================
  # SERVICE 2 : LOGSTASH
  # ==========================================
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.1
    container_name: elk_logstash
    restart: unless-stopped

    depends_on:
      elasticsearch:
        condition: service_healthy

    environment:
      - "LS_JAVA_OPTS=-Xms256m -Xmx256m"
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

    ports:
      - "8080:8080"
      - "5000:5000"
      - "9600:9600"

    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
      - ./logs/sample:/usr/share/logstash/logs/sample:ro

    networks:
      - elk_network

  # ==========================================
  # SERVICE 3 : KIBANA
  # ==========================================
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.1
    container_name: elk_kibana
    restart: unless-stopped

    depends_on:
      elasticsearch:
        condition: service_healthy

    ports:
      - "5601:5601"

    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro

    networks:
      - elk_network

    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5601/api/status || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

# ==========================================
# RÉSEAU PARTAGÉ
# ==========================================
networks:
  elk_network:
    driver: bridge
```

---

## ▶️ Étape 7 : Démarrer la stack

### 7.1 Permissions (Linux uniquement)

Sur Linux, Elasticsearch nécessite des permissions spécifiques :

```bash
# Donner les droits au dossier de données
sudo chown -R 1000:1000 elasticsearch/data

# Ou autoriser tout le monde (moins sécurisé)
chmod -R 777 elasticsearch/data
```

### 7.2 Démarrage de la stack

```bash
# Depuis le dossier elk-stack
docker-compose up -d
```

**Ce qui se passe :**
1. Téléchargement des images (première fois uniquement, ~2 GB)
2. Création du réseau `elk_network`
3. Démarrage d'Elasticsearch
4. Attente que le healthcheck d'Elasticsearch passe au vert
5. Démarrage de Logstash (commence à ingérer les logs)
6. Démarrage de Kibana

**⏳ Patience :** Le premier démarrage peut prendre 3-5 minutes.

### 7.3 Vérifier que tout fonctionne

```bash
# Voir les conteneurs actifs
docker-compose ps

# Résultat attendu :
# NAME               STATE           PORTS
# elk_elasticsearch  Up (healthy)    9200/tcp, 9300/tcp
# elk_logstash       Up              5000/tcp, 8080/tcp, 9600/tcp
# elk_kibana         Up (healthy)    5601/tcp
```

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Logs d'un service spécifique
docker-compose logs -f elasticsearch
docker-compose logs -f logstash
docker-compose logs -f kibana
```

### 7.4 Vérifier Elasticsearch

```bash
# Health check
curl http://localhost:9200/_cluster/health?pretty

# Informations sur le cluster
curl http://localhost:9200

# Lister les index créés
curl http://localhost:9200/_cat/indices?v
```

**Réponse attendue pour les index :**
```
health status index                          docs.count
yellow open   logs-application_log-2024.11.02    20
```

---

## 📊 Étape 8 : Utiliser Kibana

### 8.1 Accéder à Kibana

Ouvrez votre navigateur et allez sur :

```
http://localhost:5601
```

**Au premier démarrage :**
- Kibana peut mettre 1-2 minutes à être complètement prêt
- Si vous voyez "Kibana server is not ready yet", patientez et rechargez

### 8.2 Créer un Index Pattern

Un **Index Pattern** indique à Kibana quels index Elasticsearch interroger.

**Étapes :**

1. **Cliquer sur le menu hamburger (☰)** en haut à gauche
2. **Naviguer vers** : Management > Stack Management
3. **Dans le menu de gauche** : Kibana > Index Patterns (ou Data Views dans les versions récentes)
4. **Cliquer sur** "Create index pattern" ou "Create data view"
5. **Définir le pattern** :
   - Name : `logs-*`
   - Index pattern : `logs-*` (correspond à tous les index commençant par "logs-")
6. **Cliquer sur** "Next step"
7. **Time field** : Sélectionner `@timestamp`
8. **Cliquer sur** "Create index pattern"

**✅ Votre index pattern est créé !** Kibana peut maintenant interroger vos logs.

### 8.3 Explorer les données (Discover)

1. **Menu (☰)** → Analytics → **Discover**
2. **Sélectionner l'index pattern** "logs-*" dans le menu déroulant en haut
3. **Vous devriez voir vos logs** apparaître dans la timeline et la liste

**Interface Discover :**

```
┌─────────────────────────────────────────────────────────┐
│ [Filtres]  [Période]  [Rafraîchir]                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📊 Timeline (graphique temporel)                       │
│     ▁▂▃▄▅▆▇█                                            │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ Champs disponibles:                 Table des logs      │
│ • @timestamp                        ┌──────────────────┐│
│ • level                             │ @timestamp       ││
│ • component                         │ 2024-11-02 10:00 ││
│ • log_message                       │ [INFO]...        ││
│ • ...                               │                  ││
│                                     └──────────────────┘│
└─────────────────────────────────────────────────────────┘
```

**Actions utiles :**

- **Filtrer par niveau** : Cliquer sur un champ (ex: `level`) dans la colonne de gauche → `+` (ajouter filtre)
- **Recherche** : Utiliser la barre de recherche en haut (syntaxe Lucene ou KQL)
  - Exemple : `level: ERROR`
  - Exemple : `component: "EmailService"`
- **Période** : Ajuster la période en haut à droite (Last 15 minutes, Today, Last 7 days...)
- **Rafraîchir** : Cliquer sur "Refresh" pour voir les nouveaux logs

### 8.4 Créer des visualisations

**Exemple : Graphique des niveaux de log**

1. **Menu (☰)** → Analytics → **Visualize Library**
2. **Cliquer sur** "Create visualization"
3. **Choisir le type** : "Vertical bar" (graphique à barres verticales)
4. **Sélectionner la source** : "logs-*"
5. **Configuration** :
   - **Horizontal axis (Axe X)** :
     - Aggregation : "Terms"
     - Field : "level.keyword"
     - Order by : "Metric: Count"
   - **Vertical axis (Axe Y)** :
     - Aggregation : "Count"
6. **Cliquer sur** "Update" (bouton ▶️ en haut)
7. **Voir le graphique** : Barres représentant le nombre de logs par niveau (INFO, WARN, ERROR, DEBUG)
8. **Sauvegarder** : Cliquer sur "Save" en haut à droite, donner un nom ("Logs par niveau")

**Autres visualisations intéressantes :**

| Type | Utilité | Configuration |
|------|---------|---------------|
| **Pie Chart** | Répartition des composants | Terms sur `component.keyword` |
| **Line Chart** | Évolution temporelle | Date histogram sur `@timestamp` |
| **Metric** | Compteur simple | Count des documents |
| **Data Table** | Tableau détaillé | Multiple Metrics et Buckets |

### 8.5 Créer un Dashboard

Un **Dashboard** assemble plusieurs visualisations sur une seule page.

**Étapes :**

1. **Menu (☰)** → Analytics → **Dashboard**
2. **Cliquer sur** "Create new dashboard"
3. **Cliquer sur** "Add" (ou "Add from library")
4. **Sélectionner vos visualisations** créées précédemment
5. **Organiser** : Glisser-déposer et redimensionner les panneaux
6. **Sauvegarder** : Cliquer sur "Save" en haut à droite, donner un nom ("Dashboard Logs Application")

**Exemple de layout :**

```
┌──────────────────────────────────────────────────┐
│  Dashboard : Logs Application                    │
├─────────────────┬────────────────────────────────┤
│                 │                                │
│  📊 Logs par    │  🥧 Logs par                   │
│     niveau      │     composant                  │
│                 │                                │
├─────────────────┴────────────────────────────────┤
│                                                  │
│  📈 Évolution temporelle des logs                │
│                                                  │
├──────────────────────────────────────────────────┤
│  📋 Tableau des derniers logs ERROR              │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 🔍 Étape 9 : Requêtes et recherches

### 9.1 Requêtes Elasticsearch (API REST)

Elasticsearch expose une API REST puissante. Voici des exemples de requêtes :

**1. Rechercher tous les logs ERROR :**

```bash
curl -X GET "http://localhost:9200/logs-*/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}'
```

**2. Compter les logs par niveau :**

```bash
curl -X GET "http://localhost:9200/logs-*/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "by_level": {
      "terms": {
        "field": "level.keyword"
      }
    }
  }
}'
```

**3. Logs des dernières 24 heures :**

```bash
curl -X GET "http://localhost:9200/logs-*/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-24h"
      }
    }
  }
}'
```

### 9.2 KQL (Kibana Query Language)

Dans Kibana (Discover), vous pouvez utiliser **KQL** pour des recherches simples :

**Exemples de requêtes KQL :**

| Requête | Résultat |
|---------|----------|
| `level: ERROR` | Tous les logs de niveau ERROR |
| `component: "EmailService"` | Logs du composant EmailService |
| `level: ERROR AND component: "DatabaseService"` | Logs ERROR du DatabaseService |
| `level: (ERROR OR WARN)` | Logs ERROR ou WARN |
| `log_message: *timeout*` | Logs contenant le mot "timeout" |
| `NOT level: DEBUG` | Tous les logs sauf DEBUG |

### 9.3 Lucene Query Syntax

Si vous désactivez KQL dans Kibana, vous pouvez utiliser la syntaxe Lucene :

```
level:ERROR AND component:EmailService
level:(ERROR OR WARN)
log_message:timeout AND level:ERROR
@timestamp:[now-1h TO now]
```

---

## 📊 Étape 10 : Envoyer des logs en temps réel

### 10.1 Via TCP (JSON)

Logstash écoute sur le port 5000 (TCP) :

```bash
# Envoyer un log JSON via TCP
echo '{"level":"INFO","component":"TestService","message":"Test depuis TCP"}' | nc localhost 5000
```

### 10.2 Via HTTP (JSON)

Logstash écoute sur le port 8080 (HTTP) :

```bash
# Envoyer un log JSON via HTTP
curl -X POST "http://localhost:8080" -H 'Content-Type: application/json' -d'
{
  "level": "WARN",
  "component": "MonitoringService",
  "message": "CPU usage above 80%",
  "cpu_usage": 85.5,
  "timestamp": "2024-11-02T12:00:00.000Z"
}'
```

### 10.3 Ajouter des logs au fichier

Vous pouvez continuer à ajouter des lignes au fichier `logs/sample/app.log` :

```bash
# Ajouter une nouvelle ligne
echo "[$(date -Iseconds)] [INFO] [TestService] Test log entry" >> logs/sample/app.log
```

Logstash détectera automatiquement le nouveau log et l'indexera dans Elasticsearch.

---

## 🛠️ Étape 11 : Gestion et maintenance

### 11.1 Commandes utiles

```bash
# Voir les conteneurs actifs
docker-compose ps

# Logs de tous les services
docker-compose logs

# Logs en temps réel
docker-compose logs -f

# Logs d'un service spécifique
docker-compose logs -f logstash

# Statistiques de ressources
docker stats elk_elasticsearch elk_logstash elk_kibana

# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer tous les services
docker-compose start

# Redémarrer un service spécifique
docker-compose restart logstash
```

### 11.2 Gestion des index Elasticsearch

**Lister les index :**
```bash
curl -X GET "http://localhost:9200/_cat/indices?v&s=index"
```

**Supprimer un index :**
```bash
curl -X DELETE "http://localhost:9200/logs-application_log-2024.11.01"
```

**Supprimer tous les index logs-* :**
```bash
curl -X DELETE "http://localhost:9200/logs-*"
```

**Voir l'espace disque utilisé :**
```bash
curl -X GET "http://localhost:9200/_cat/allocation?v"
```

### 11.3 Réindexation

Si vous modifiez le pipeline Logstash, vous devrez peut-être réindexer :

```bash
# 1. Supprimer les anciens index
curl -X DELETE "http://localhost:9200/logs-*"

# 2. Redémarrer Logstash
docker-compose restart logstash

# Logstash va relire les fichiers de logs et réindexer
```

---

## 🛑 Étape 12 : Arrêt et nettoyage

### 12.1 Arrêter proprement

```bash
# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer plus tard
docker-compose start
```

### 12.2 Suppression complète

```bash
# Arrêter et supprimer les conteneurs
docker-compose down

# Supprimer les données Elasticsearch (⚠️ IRRÉVERSIBLE)
rm -rf elasticsearch/data/*

# Supprimer les images Docker (optionnel)
docker rmi docker.elastic.co/elasticsearch/elasticsearch:8.11.1
docker rmi docker.elastic.co/logstash/logstash:8.11.1
docker rmi docker.elastic.co/kibana/kibana:8.11.1

# Supprimer le réseau
docker network rm elk-stack_elk_network
```

---

## 🐛 Dépannage

### Problème 1 : Elasticsearch ne démarre pas

**Symptôme :** `elk_elasticsearch` se redémarre en boucle ou s'arrête immédiatement.

**Solutions :**

1. **Vérifier la mémoire allouée à Docker :**
   ```bash
   docker info | grep Memory
   # Doit afficher au moins 4 GB
   ```
   - Windows/macOS : Docker Desktop → Settings → Resources → Augmenter la RAM

2. **Vérifier les logs Elasticsearch :**
   ```bash
   docker-compose logs elasticsearch
   ```
   - Cherchez des erreurs comme "OutOfMemoryError" ou "vm.max_map_count"

3. **Linux : Augmenter vm.max_map_count :**
   ```bash
   sudo sysctl -w vm.max_map_count=262144

   # Permanent (survit aux redémarrages)
   echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
   ```

4. **Vérifier les permissions du dossier data :**
   ```bash
   sudo chown -R 1000:1000 elasticsearch/data
   ```

### Problème 2 : Logstash ne traite pas les logs

**Symptôme :** Les logs ne s'affichent pas dans Kibana.

**Solutions :**

1. **Vérifier que Logstash est démarré :**
   ```bash
   docker-compose ps
   # elk_logstash doit être "Up"
   ```

2. **Vérifier les logs Logstash :**
   ```bash
   docker-compose logs logstash | grep -i error
   ```

3. **Vérifier que le fichier de logs existe et est accessible :**
   ```bash
   docker exec -it elk_logstash ls -la /usr/share/logstash/logs/sample/
   ```

4. **Tester le pipeline manuellement :**
   ```bash
   # Ajouter une nouvelle ligne au fichier
   echo "[$(date -Iseconds)] [ERROR] [TestService] Manual test log" >> logs/sample/app.log

   # Voir les logs Logstash en temps réel
   docker-compose logs -f logstash
   ```

5. **Vérifier qu'Elasticsearch reçoit les données :**
   ```bash
   curl -X GET "http://localhost:9200/_cat/indices?v"
   # Vérifiez que les index logs-* existent
   ```

### Problème 3 : Kibana "Unable to connect to Elasticsearch"

**Symptôme :** Kibana affiche un message d'erreur de connexion.

**Solutions :**

1. **Vérifier qu'Elasticsearch est accessible :**
   ```bash
   curl http://localhost:9200
   # Doit retourner un JSON avec des infos sur le cluster
   ```

2. **Vérifier la configuration de Kibana :**
   ```bash
   cat kibana/config/kibana.yml | grep elasticsearch.hosts
   # Doit être : http://elasticsearch:9200
   ```

3. **Redémarrer Kibana :**
   ```bash
   docker-compose restart kibana
   ```

4. **Vérifier le réseau Docker :**
   ```bash
   docker network inspect elk-stack_elk_network
   # Les 3 conteneurs doivent être listés
   ```

### Problème 4 : Index pattern ne trouve pas d'index

**Symptôme :** "No matching indices found" lors de la création de l'index pattern.

**Solutions :**

1. **Vérifier que des index existent dans Elasticsearch :**
   ```bash
   curl -X GET "http://localhost:9200/_cat/indices?v"
   ```
   - Si aucun index `logs-*` n'apparaît, Logstash n'a pas encore traité de logs

2. **Vérifier que le fichier app.log contient des données :**
   ```bash
   cat logs/sample/app.log
   ```

3. **Redémarrer Logstash pour forcer le retraitement :**
   ```bash
   docker-compose restart logstash
   ```

4. **Attendre quelques secondes** et rafraîchir Kibana

### Problème 5 : "Disk watermark exceeded"

**Symptôme :** Elasticsearch refuse d'indexer de nouvelles données.

**Cause :** Le disque est plein à plus de 90%.

**Solutions :**

1. **Vérifier l'espace disque :**
   ```bash
   df -h
   ```

2. **Supprimer les vieux index :**
   ```bash
   # Supprimer les index de plus de 7 jours
   curl -X DELETE "http://localhost:9200/logs-*-2024.10.*"
   ```

3. **Augmenter le seuil (temporaire, pour développement) :**
   ```bash
   curl -X PUT "http://localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
   {
     "transient": {
       "cluster.routing.allocation.disk.watermark.low": "95%",
       "cluster.routing.allocation.disk.watermark.high": "97%"
     }
   }'
   ```

---

## ✅ Récapitulatif

### Ce que vous avez appris

- ✅ Comprendre l'architecture et le rôle de chaque composant ELK
- ✅ Déployer Elasticsearch, Logstash et Kibana avec Docker Compose
- ✅ Créer un pipeline Logstash pour collecter et transformer des logs
- ✅ Utiliser Grok pour parser des logs non structurés
- ✅ Indexer des données dans Elasticsearch
- ✅ Créer des index patterns dans Kibana
- ✅ Explorer des logs avec Discover
- ✅ Créer des visualisations et des dashboards
- ✅ Effectuer des recherches avec KQL et l'API Elasticsearch

### Technologies maîtrisées

| Technologie | Version | Rôle |
|-------------|---------|------|
| Elasticsearch | 8.11.1 | Moteur de recherche et stockage |
| Logstash | 8.11.1 | Pipeline de traitement de données |
| Kibana | 8.11.1 | Interface de visualisation |
| Docker | 20.10+ | Conteneurisation |

### Fichiers créés

```
elk-stack/
├── docker-compose.yml              # Orchestration des 3 services
├── elasticsearch/
│   └── data/                       # Données indexées (persistantes)
├── logstash/
│   ├── config/
│   │   └── logstash.yml           # Config Logstash
│   └── pipeline/
│       └── logstash.conf          # Pipeline de traitement
├── kibana/
│   └── config/
│       └── kibana.yml             # Config Kibana
└── logs/
    └── sample/
        └── app.log                # Logs d'exemple
```

### Ports utilisés

| Service | Port | Usage |
|---------|------|-------|
| Elasticsearch | 9200 | API REST |
| Elasticsearch | 9300 | Communication inter-nœuds |
| Logstash | 5000 | Input TCP |
| Logstash | 8080 | Input HTTP |
| Logstash | 9600 | Monitoring API |
| Kibana | 5601 | Interface web |

### Commandes essentielles

```bash
# Démarrer la stack
docker-compose up -d

# Voir les logs
docker-compose logs -f

# Arrêter (données conservées)
docker-compose stop

# Redémarrer
docker-compose start

# Supprimer tout
docker-compose down
rm -rf elasticsearch/data

# API Elasticsearch
curl http://localhost:9200/_cat/indices?v

# Accès Kibana
http://localhost:5601
```

---

## 🚀 Pour aller plus loin

### Extensions possibles

1. **Ajouter Beats** (collecteurs légers)
   - **Filebeat** : Collecter des logs de fichiers
   - **Metricbeat** : Métriques système et applicatives
   - **Packetbeat** : Analyse réseau
   - **Heartbeat** : Monitoring de disponibilité

2. **Activer la sécurité** (production)
   - Authentification (users/passwords)
   - Chiffrement TLS/SSL
   - API keys

3. **Configurer des alertes**
   - Détection d'anomalies
   - Notifications (email, Slack, webhook)
   - Watcher (alerting d'Elastic)

4. **Optimiser les performances**
   - Sharding et replicas
   - Index Lifecycle Management (ILM)
   - Index templates
   - Compression

5. **Ajouter Grafana**
   - Alternative à Kibana pour les visualisations
   - Intégration avec Elasticsearch

6. **Cluster multi-nœuds**
   - Haute disponibilité
   - Load balancing
   - Failover automatique

### Scénarios avancés

| Scénario | Description |
|----------|-------------|
| **Log aggregation** | Collecter les logs de 100+ serveurs |
| **SIEM** | Security Information and Event Management |
| **APM** | Application Performance Monitoring |
| **Observability** | Logs + Metrics + Traces (Elastic Observability) |
| **Business analytics** | Analyser les comportements utilisateurs |

### Ressources complémentaires

- 📖 [Documentation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- 📖 [Documentation Logstash](https://www.elastic.co/guide/en/logstash/current/index.html)
- 📖 [Documentation Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)
- 📖 [Grok patterns](https://github.com/elastic/logstash/blob/main/patterns/grok-patterns)
- 🎓 [Elastic training](https://www.elastic.co/training/)
- 💬 [Discuss forum](https://discuss.elastic.co/)

---

## 🎉 Félicitations !

Vous avez maintenant une **stack ELK complète et fonctionnelle** ! Cette base vous permet de :

- 📊 Centraliser et analyser vos logs
- 🔍 Effectuer des recherches ultra-rapides
- 📈 Créer des dashboards interactifs
- 🚨 Détecter des anomalies et erreurs
- 📉 Suivre des KPIs en temps réel
- 🐳 Déployer facilement avec Docker

**Prochain pas :** Explorez d'autres configurations du guide !

➡️ [Stack LAMP (Apache + MariaDB + PHP)](01-stack-lamp.md)
➡️ [Stack MEAN (MongoDB + Express + Angular + Node)](02-stack-mean.md)
➡️ [Environnement de développement complet](04-env-dev-complet.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
