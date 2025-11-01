# 9.4 Configuration InfluxDB avec Telegraf

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**Telegraf** est un agent de collecte de métriques développé par InfluxData (les créateurs d'InfluxDB). C'est le **collecteur officiel** pour alimenter InfluxDB en données.

**Le problème qu'il résout :**
Jusqu'à présent, vous deviez **manuellement** insérer des données dans InfluxDB. Telegraf automatise complètement ce processus en collectant automatiquement des métriques système et applicatives.

**Cas d'usage typiques :**
- 📊 **Monitoring système** automatique (CPU, RAM, disque, réseau)
- 🐳 **Monitoring Docker** (conteneurs, images, volumes)
- 🌐 **Monitoring web** (temps de réponse HTTP, statuts)
- 💾 **Monitoring bases de données** (MySQL, PostgreSQL, MongoDB...)
- 🔌 **IoT et capteurs** (via MQTT, HTTP, fichiers...)

**Telegraf en chiffres :**
- ⚡ **300+ plugins** d'entrée (inputs)
- 📤 **60+ plugins** de sortie (outputs)
- 🔧 **30+ plugins** de traitement (processors)
- 🚀 Écrit en **Go** (rapide et léger)

---

## 🎯 Ce que vous allez apprendre

✅ Comprendre l'architecture Telegraf + InfluxDB
✅ Déployer Telegraf avec Docker Compose
✅ Configurer la collecte de métriques système
✅ Connecter Telegraf à InfluxDB
✅ Visualiser les métriques dans InfluxDB et Grafana
✅ Ajouter des plugins personnalisés
✅ Monitorer des conteneurs Docker

---

## 🔑 Qu'est-ce que Telegraf ?

**Telegraf** est un agent qui fonctionne selon un principe simple :

```
┌─────────────────────────────────────────────┐
│              TELEGRAF AGENT                 │
│                                             │
│  📥 INPUTS       🔄 PROCESSORS   📤 OUTPUTS  │
│  (Collecte)      (Traitement)    (Envoi)    │
│                                             │
│  • CPU           • Filtres       • InfluxDB │
│  • RAM           • Agrégations   • Kafka    │
│  • Disque        • Conversions   • MQTT     │
│  • Réseau        • Renommage     • Files    │
│  • Docker        • ...           • ...      │
│  • HTTP          ...                        │
│  • ...                                      │
└─────────────────────────────────────────────┘
```

**Cycle de fonctionnement :**
1. 📥 **Inputs** : Collecte des données depuis diverses sources
2. 🔄 **Processors** : Traite/transforme les données (optionnel)
3. 📤 **Outputs** : Envoie les données vers InfluxDB (ou autres)
4. 🔁 **Répète** toutes les X secondes (intervalle configurable)

---

## 🏗️ Architecture de la stack complète

```
┌──────────────┐
│   Grafana    │ ← Visualisation (Port 3000)
│  (Dashboard) │
└──────┬───────┘
       │ Query
       ↓
┌──────────────┐
│   InfluxDB   │ ← Stockage (Port 8086)
│   (Database) │
└──────┬───────┘
       │ Write
       ↓
┌──────────────┐
│   Telegraf   │ ← Collecte automatique
│    (Agent)   │
└──────────────┘
       ↓
┌──────────────┐
│   Serveur /  │ ← Métriques système
│  Conteneurs  │
└──────────────┘
```

**Flux de données :**
1. **Telegraf** collecte les métriques du système/conteneurs
2. **Telegraf** envoie les données à **InfluxDB**
3. **Grafana** lit les données depuis **InfluxDB**
4. **Vous** visualisez les dashboards dans **Grafana**

---

## 📦 Prérequis

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Avoir suivi les fiches précédentes (recommandé) :
  - [9.1 - Configuration basique InfluxDB](01-config-basique-v2.md)
  - [9.3 - InfluxDB avec Grafana](03-influxdb-grafana.md)

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🚀 Étape 1 : Préparation des fichiers

### A. Créer le dossier du projet

```bash
# Créer et entrer dans le dossier
mkdir influxdb_telegraf_stack
cd influxdb_telegraf_stack
```

### B. Structure des fichiers

Nous allons créer deux fichiers :

```
influxdb_telegraf_stack/
├── docker-compose.yml    # Configuration des services
└── telegraf.conf         # Configuration de Telegraf
```

---

## 🚀 Étape 2 : Configuration de Telegraf

### A. Créer le fichier `telegraf.conf`

Créez un fichier `telegraf.conf` avec le contenu suivant :

```toml
# ============================================
# CONFIGURATION GLOBALE DE TELEGRAF
# ============================================
[agent]
  # Intervalle de collecte (toutes les 10 secondes)
  interval = "10s"

  # Arrondir les timestamps à l'intervalle
  round_interval = true

  # Taille du buffer (nombre de métriques en mémoire)
  metric_batch_size = 1000

  # Taille maximale du buffer
  metric_buffer_limit = 10000

  # Timeout de collecte par plugin
  collection_jitter = "0s"

  # Timeout d'envoi
  flush_interval = "10s"
  flush_jitter = "0s"

  # Niveau de logs (error, warn, info, debug)
  debug = false
  quiet = false

  # Hostname utilisé dans les tags
  hostname = "telegraf-docker"

# ============================================
# OUTPUT : INFLUXDB V2
# ============================================
[[outputs.influxdb_v2]]
  # URL de votre InfluxDB (utiliser le nom du service Docker)
  urls = ["http://influxdb:8086"]

  # Token d'authentification (celui défini dans docker-compose)
  token = "my_super_secret_token_12345"

  # Organisation
  organization = "monitoring_org"

  # Bucket où envoyer les données
  bucket = "telegraf_metrics"

  # Timeout de connexion
  timeout = "5s"

# ============================================
# INPUT : MÉTRIQUES CPU
# ============================================
[[inputs.cpu]]
  # Collecte les stats CPU par core et total
  percpu = true
  totalcpu = true

  # Collecter les temps CPU bruts
  collect_cpu_time = false

  # Rapport en % d'utilisation
  report_active = true

# ============================================
# INPUT : MÉTRIQUES MÉMOIRE
# ============================================
[[inputs.mem]]
  # Pas d'options spécifiques, collecte tout

# ============================================
# INPUT : MÉTRIQUES DISQUE
# ============================================
[[inputs.disk]]
  # Points de montage à ignorer
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

# ============================================
# INPUT : MÉTRIQUES DISQUE I/O
# ============================================
[[inputs.diskio]]
  # Collecte les I/O par disque

# ============================================
# INPUT : MÉTRIQUES RÉSEAU
# ============================================
[[inputs.net]]
  # Interfaces à ignorer
  ignore_protocol_stats = false

# ============================================
# INPUT : MÉTRIQUES SYSTÈME
# ============================================
[[inputs.system]]
  # Collecte : uptime, nombre de CPU, load average

# ============================================
# INPUT : MÉTRIQUES DOCKER (optionnel)
# ============================================
[[inputs.docker]]
  # Socket Docker (accès au daemon Docker)
  endpoint = "unix:///var/run/docker.sock"

  # Timeout de connexion
  timeout = "5s"

  # Collecter les stats de tous les conteneurs
  gather_services = false

  # Collecter les stats détaillées
  container_names = []

  # Métriques à collecter
  total = true
  perdevice = true
```

**🔑 Points importants :**

- **`[agent]`** : Configuration générale (intervalle, buffer, logs)
- **`[[outputs.influxdb_v2]]`** : Où envoyer les données (InfluxDB)
- **`[[inputs.xxx]]`** : Quelles métriques collecter (CPU, RAM, disque...)
- **Token** : Doit correspondre à celui d'InfluxDB
- **Bucket** : Nous utilisons `telegraf_metrics` (à créer dans InfluxDB)

---

## 🚀 Étape 3 : Configuration Docker Compose

### A. Créer le fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  # ============================================
  # SERVICE INFLUXDB
  # ============================================
  influxdb:
    image: influxdb:2.7
    container_name: influxdb_server
    restart: unless-stopped
    ports:
      - "8086:8086"
    volumes:
      - influxdb_data:/var/lib/influxdb2
      - influxdb_config:/etc/influxdb2
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_ORG=monitoring_org
      - DOCKER_INFLUXDB_INIT_BUCKET=telegraf_metrics
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      # ⚠️ CHANGEZ ces mots de passe en production !
      - DOCKER_INFLUXDB_INIT_PASSWORD=admin_password_123
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my_super_secret_token_12345

  # ============================================
  # SERVICE TELEGRAF
  # ============================================
  telegraf:
    image: telegraf:1.28
    container_name: telegraf_agent
    restart: unless-stopped
    volumes:
      # Fichier de configuration
      - ./telegraf.conf:/etc/telegraf/telegraf.conf:ro
      # Accès au socket Docker (pour monitorer les conteneurs)
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # Accès aux informations système (optionnel mais recommandé)
      - /:/hostfs:ro
    environment:
      # Variables d'environnement pour la config
      - HOST_ETC=/hostfs/etc
      - HOST_PROC=/hostfs/proc
      - HOST_SYS=/hostfs/sys
      - HOST_VAR=/hostfs/var
      - HOST_RUN=/hostfs/run
      - HOST_MOUNT_PREFIX=/hostfs
    depends_on:
      - influxdb

  # ============================================
  # SERVICE GRAFANA (optionnel)
  # ============================================
  grafana:
    image: grafana/grafana:10.2.0
    container_name: grafana_server
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - influxdb

volumes:
  influxdb_data:
  influxdb_config:
  grafana_data:
```

**🔑 Points importants :**

- **Telegraf** :
  - Monte `telegraf.conf` en lecture seule (`:ro`)
  - Accède au socket Docker pour monitorer les conteneurs
  - Monte `/` en lecture seule pour accéder aux métriques système

- **InfluxDB** :
  - Bucket par défaut : `telegraf_metrics` (correspond à la config Telegraf)

- **Grafana** :
  - Optionnel, mais pratique pour visualiser les données

---

## ▶️ Étape 4 : Lancement de la stack

### A. Démarrer les services

Dans le dossier contenant vos fichiers, exécutez :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating network "influxdb_telegraf_stack_default" with the default driver
Creating volume "influxdb_telegraf_stack_influxdb_data" with default driver
Creating volume "influxdb_telegraf_stack_influxdb_config" with default driver
Creating volume "influxdb_telegraf_stack_grafana_data" with default driver
Creating influxdb_server ... done
Creating telegraf_agent ... done
Creating grafana_server  ... done
```

### B. Vérifier que les services fonctionnent

```bash
docker-compose ps
```

**Sortie attendue :**
```
      Name                    Command               State           Ports
-----------------------------------------------------------------------------------
grafana_server     /run.sh                          Up      0.0.0.0:3000->3000/tcp
influxdb_server    /entrypoint.sh influxd           Up      0.0.0.0:8086->8086/tcp
telegraf_agent     /entrypoint.sh telegraf          Up
```

Les trois services doivent être `Up`.

### C. Consulter les logs de Telegraf

Pour voir si Telegraf collecte bien les données :

```bash
docker-compose logs -f telegraf
```

**Sortie attendue :**
```
telegraf_agent    | 2024-11-01T10:00:00Z I! Starting Telegraf 1.28.0
telegraf_agent    | 2024-11-01T10:00:00Z I! Loaded inputs: cpu disk diskio mem net system docker
telegraf_agent    | 2024-11-01T10:00:00Z I! Loaded outputs: influxdb_v2
telegraf_agent    | 2024-11-01T10:00:10Z D! Output [influxdb_v2] wrote batch of 45 metrics
```

✅ Si vous voyez `wrote batch of X metrics`, **c'est parfait** ! Telegraf envoie des données à InfluxDB.

Appuyez sur `Ctrl+C` pour quitter les logs.

---

## 🔍 Étape 5 : Vérifier les données dans InfluxDB

### A. Accéder à InfluxDB

Ouvrez votre navigateur et allez à :

```
http://localhost:8086
```

**Connexion :**
- Username : `admin`
- Password : `admin_password_123`

### B. Explorer les données collectées

1. Cliquez sur **"Data Explorer"** (📊) dans la barre latérale
2. Dans **"From"**, sélectionnez le bucket : **`telegraf_metrics`**
3. Dans **"Filter"**, vous devriez voir toutes les mesures collectées :
   - `cpu` (utilisation CPU)
   - `mem` (mémoire)
   - `disk` (espace disque)
   - `diskio` (I/O disque)
   - `net` (réseau)
   - `system` (informations système)
   - `docker` (métriques Docker, si activé)

### C. Créer une requête simple

Dans le **Query Builder** :

1. **From** : `telegraf_metrics`
2. **Measurement** : `cpu`
3. **Field** : `usage_idle` (temps d'inactivité du CPU)
4. Cliquez sur **"Submit"**

Vous devriez voir un graphique avec l'utilisation CPU en temps réel ! 🎉

---

## 📊 Étape 6 : Visualiser dans Grafana

### A. Accéder à Grafana

Ouvrez un nouvel onglet et allez à :

```
http://localhost:3000
```

**Connexion :**
- Username : `admin`
- Password : `admin`

### B. Ajouter la source de données InfluxDB

Si ce n'est pas déjà fait (voir [fiche 9.3](03-influxdb-grafana.md)) :

1. **"Connections"** → **"Data sources"** → **"Add data source"**
2. Sélectionnez **"InfluxDB"**
3. Configurez :
   - **URL** : `http://influxdb:8086`
   - **Query Language** : `Flux`
   - **Organization** : `monitoring_org`
   - **Token** : `my_super_secret_token_12345`
   - **Default Bucket** : `telegraf_metrics`
4. **"Save & Test"**

### C. Créer un dashboard de monitoring système

**Panel 1 : Utilisation CPU**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> filter(fn: (r) => r._field == "usage_system" or r._field == "usage_user")
  |> filter(fn: (r) => r.cpu == "cpu-total")
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)
```

**Panel 2 : Utilisation Mémoire**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "mem")
  |> filter(fn: (r) => r._field == "used_percent")
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)
```

**Panel 3 : Utilisation Disque**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "disk")
  |> filter(fn: (r) => r._field == "used_percent")
  |> filter(fn: (r) => r.path == "/")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
```

**Panel 4 : Trafic Réseau**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "net")
  |> filter(fn: (r) => r._field == "bytes_recv" or r._field == "bytes_sent")
  |> derivative(unit: 1s, nonNegative: true)
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)
```

---

## 🐳 Étape 7 : Monitorer les conteneurs Docker

### A. Vérifier que le plugin Docker fonctionne

Dans InfluxDB Data Explorer :

1. **Bucket** : `telegraf_metrics`
2. **Measurement** : `docker` ou `docker_container_cpu`
3. Vous devriez voir les métriques de vos conteneurs

### B. Requête pour voir tous les conteneurs

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "docker_container_cpu")
  |> filter(fn: (r) => r._field == "usage_percent")
  |> group(columns: ["container_name"])
```

### C. Dashboard Docker dans Grafana

**CPU par conteneur :**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "docker_container_cpu")
  |> filter(fn: (r) => r._field == "usage_percent")
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)
```

**Mémoire par conteneur :**

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "docker_container_mem")
  |> filter(fn: (r) => r._field == "usage_percent")
  |> aggregateWindow(every: 10s, fn: mean, createEmpty: false)
```

---

## 🔧 Étape 8 : Ajouter des plugins personnalisés

### A. Plugin HTTP Response (monitoring web)

Ajoutez ceci dans votre `telegraf.conf` :

```toml
# ============================================
# INPUT : MONITORING HTTP
# ============================================
[[inputs.http_response]]
  # URLs à monitorer
  urls = [
    "https://www.google.com",
    "https://www.github.com",
    "http://localhost:8086"
  ]

  # Timeout de requête
  response_timeout = "5s"

  # Méthode HTTP
  method = "GET"

  # Suivre les redirections
  follow_redirects = true

  # Tags additionnels
  [inputs.http_response.tags]
    service = "web_monitoring"
```

### B. Plugin Ping (monitoring réseau)

```toml
# ============================================
# INPUT : PING
# ============================================
[[inputs.ping]]
  # Hôtes à pinger
  urls = ["8.8.8.8", "1.1.1.1", "google.com"]

  # Nombre de pings
  count = 4

  # Timeout
  timeout = 1.0

  # Intervalle entre les pings
  ping_interval = 1.0
```

### C. Plugin MQTT (IoT)

```toml
# ============================================
# INPUT : MQTT (Capteurs IoT)
# ============================================
[[inputs.mqtt_consumer]]
  # Serveur MQTT
  servers = ["tcp://mosquitto:1883"]

  # Topics à écouter
  topics = [
    "sensors/temperature",
    "sensors/humidity"
  ]

  # Format des données
  data_format = "json"
```

### D. Appliquer les changements

Après modification du `telegraf.conf` :

```bash
# Redémarrer Telegraf
docker-compose restart telegraf

# Vérifier les logs
docker-compose logs -f telegraf
```

---

## 📊 Étape 9 : Dashboard Grafana pré-configuré

### A. Importer un dashboard Telegraf officiel

Grafana propose des dashboards communautaires prêts à l'emploi :

1. Dans Grafana, cliquez sur **"+"** → **"Import dashboard"**
2. Entrez l'**ID** : `928` (Dashboard Telegraf System Metrics)
3. Cliquez sur **"Load"**
4. Sélectionnez votre source de données **InfluxDB**
5. Cliquez sur **"Import"**

🎉 Vous avez maintenant un dashboard professionnel avec :
- Utilisation CPU, RAM, disque
- Trafic réseau
- I/O disque
- Load average
- Uptime

### B. Autres dashboards recommandés

| ID | Nom | Description |
|----|-----|-------------|
| `928` | Telegraf System Metrics | Métriques système complètes |
| `1443` | Telegraf Docker Monitoring | Monitoring de conteneurs |
| `5955` | Telegraf InfluxDB | Vue d'ensemble InfluxDB |

---

## 🔄 Étape 10 : Configuration avancée de Telegraf

### A. Intervalles personnalisés par plugin

Vous pouvez définir des intervalles différents pour chaque input :

```toml
# Collecter le CPU toutes les 5 secondes
[[inputs.cpu]]
  interval = "5s"
  percpu = true
  totalcpu = true

# Collecter le disque toutes les 60 secondes (moins fréquent)
[[inputs.disk]]
  interval = "60s"
```

### B. Filtrer et transformer les données (Processors)

**Renommer des champs :**

```toml
[[processors.rename]]
  [[processors.rename.replace]]
    measurement = "cpu"
    dest = "cpu_metrics"
```

**Ajouter des tags personnalisés :**

```toml
[[processors.enum]]
  [[processors.enum.mapping]]
    tag = "environment"
    dest = "env_type"
    [processors.enum.mapping.value_mappings]
      "prod" = "production"
      "dev" = "development"
```

**Filtrer certaines métriques :**

```toml
[[processors.regex]]
  [[processors.regex.tags]]
    key = "cpu"
    pattern = "^cpu-total$"
    result_key = "filtered"
```

### C. Multiple outputs (envoyer vers plusieurs destinations)

```toml
# Output principal : InfluxDB
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "my_super_secret_token_12345"
  organization = "monitoring_org"
  bucket = "telegraf_metrics"

# Output secondaire : Fichier (backup)
[[outputs.file]]
  files = ["/tmp/telegraf_backup.out"]
  data_format = "json"
```

---

## 🛡️ Étape 11 : Sécurité et bonnes pratiques

### A. Utiliser des variables d'environnement

Au lieu de mettre le token en dur dans `telegraf.conf`, utilisez des variables :

**Dans `telegraf.conf` :**
```toml
[[outputs.influxdb_v2]]
  urls = ["$INFLUX_URL"]
  token = "$INFLUX_TOKEN"
  organization = "$INFLUX_ORG"
  bucket = "$INFLUX_BUCKET"
```

**Dans `docker-compose.yml` :**
```yaml
telegraf:
  environment:
    - INFLUX_URL=http://influxdb:8086
    - INFLUX_TOKEN=my_super_secret_token_12345
    - INFLUX_ORG=monitoring_org
    - INFLUX_BUCKET=telegraf_metrics
```

### B. Fichier `.env` (recommandé)

Créez un fichier `.env` :

```env
INFLUX_TOKEN=my_super_secret_token_12345
INFLUX_PASSWORD=admin_password_123
```

**Dans `docker-compose.yml` :**
```yaml
services:
  influxdb:
    environment:
      - DOCKER_INFLUXDB_INIT_PASSWORD=${INFLUX_PASSWORD}
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=${INFLUX_TOKEN}

  telegraf:
    env_file:
      - .env
```

**⚠️ N'oubliez pas d'ajouter `.env` dans `.gitignore` !**

---

## 🛑 Étape 12 : Gestion de la stack

### A. Commandes essentielles

```bash
# Démarrer tous les services
docker-compose up -d

# Arrêter tous les services (sans supprimer)
docker-compose stop

# Redémarrer tous les services
docker-compose restart

# Redémarrer uniquement Telegraf (après modification de telegraf.conf)
docker-compose restart telegraf

# Voir les logs en temps réel
docker-compose logs -f telegraf

# Voir l'état des services
docker-compose ps

# Supprimer tout (⚠️ supprime les données)
docker-compose down -v
```

### B. Recharger la configuration Telegraf

Après modification de `telegraf.conf` :

```bash
# Méthode 1 : Redémarrage complet
docker-compose restart telegraf

# Méthode 2 : Envoyer un signal HUP (reload config sans restart)
docker exec telegraf_agent killall -HUP telegraf
```

---

## 📈 Étape 13 : Monitoring des performances de Telegraf

### A. Métriques internes de Telegraf

Telegraf peut monitorer ses propres performances :

```toml
# Ajouter dans telegraf.conf
[[inputs.internal]]
  # Collecte les métriques internes de Telegraf
  collect_memstats = true
```

**Métriques disponibles :**
- Nombre de métriques collectées/envoyées
- Utilisation mémoire de Telegraf
- Erreurs de collecte
- Temps d'exécution des plugins

### B. Visualiser dans Grafana

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "internal_agent")
  |> filter(fn: (r) => r._field == "metrics_gathered")
```

---

## ✅ Récapitulatif

**Ce que vous avez appris :**

✅ Comprendre le rôle de Telegraf dans la stack de monitoring
✅ Configurer Telegraf pour collecter des métriques système
✅ Connecter Telegraf à InfluxDB avec authentification
✅ Déployer une stack complète (Telegraf + InfluxDB + Grafana)
✅ Monitorer le système (CPU, RAM, disque, réseau)
✅ Monitorer les conteneurs Docker
✅ Ajouter des plugins personnalisés (HTTP, Ping, MQTT...)
✅ Créer des dashboards Grafana avec les données Telegraf
✅ Utiliser les processors pour transformer les données
✅ Gérer la configuration et la sécurité

---

## 💡 Plugins Telegraf les plus utiles

| Plugin | Usage | Configuration |
|--------|-------|---------------|
| **cpu** | Utilisation CPU | `[[inputs.cpu]]` |
| **mem** | Utilisation RAM | `[[inputs.mem]]` |
| **disk** | Espace disque | `[[inputs.disk]]` |
| **docker** | Stats conteneurs | `[[inputs.docker]]` |
| **http_response** | Monitoring HTTP | `[[inputs.http_response]]` |
| **mysql** | Stats MySQL | `[[inputs.mysql]]` |
| **postgresql** | Stats PostgreSQL | `[[inputs.postgresql]]` |
| **redis** | Stats Redis | `[[inputs.redis]]` |
| **nginx** | Stats Nginx | `[[inputs.nginx]]` |
| **ping** | Test de latence | `[[inputs.ping]]` |
| **mqtt_consumer** | Capteurs IoT | `[[inputs.mqtt_consumer]]` |

---

## 🐛 Étape 14 : Dépannage

### A. Telegraf ne démarre pas

**Erreur :** `Error in config file: ...`

**Causes possibles :**
1. Erreur de syntaxe dans `telegraf.conf`
2. Plugin mal configuré

**Solutions :**
```bash
# Vérifier la syntaxe du fichier de config
docker run --rm -v $(pwd)/telegraf.conf:/telegraf.conf:ro telegraf:1.28 telegraf --config /telegraf.conf --test

# Voir les logs détaillés
docker-compose logs telegraf
```

### B. Pas de données dans InfluxDB

**Causes possibles :**
1. Mauvais token/bucket/organization
2. InfluxDB n'est pas démarré
3. Problème réseau entre Telegraf et InfluxDB

**Solutions :**
```bash
# 1. Vérifier que InfluxDB est accessible
docker exec telegraf_agent curl http://influxdb:8086/ping

# 2. Vérifier les logs de Telegraf
docker-compose logs telegraf | grep "error"

# 3. Tester l'écriture manuellement
docker exec influxdb_server influx write \
  --bucket telegraf_metrics \
  --org monitoring_org \
  --token my_super_secret_token_12345 \
  "test_metric value=1"
```

### C. Erreur "permission denied" sur Docker socket

**Erreur :** `Cannot connect to Docker daemon`

**Solution :** Vérifier que le socket Docker est bien monté :

```yaml
telegraf:
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
```

Sur Linux, vous devrez peut-être ajouter l'utilisateur au groupe docker :

```bash
# Alternative : Donner les permissions au socket (temporaire)
sudo chmod 666 /var/run/docker.sock
```

### D. Métriques système incorrectes

**Cause :** Le conteneur Telegraf n'a pas accès au système hôte.

**Solution :** Vérifier les montages de volumes :

```yaml
telegraf:
  volumes:
    - /:/hostfs:ro
  environment:
    - HOST_PROC=/hostfs/proc
    - HOST_SYS=/hostfs/sys
```

---

## 📚 Étape 15 : Exemples de configurations avancées

### A. Monitoring d'une base de données MySQL

```toml
[[inputs.mysql]]
  # Chaîne de connexion
  servers = ["user:password@tcp(mysql_host:3306)/"]

  # Métriques à collecter
  metric_version = 2

  # Collecter les stats détaillées
  gather_table_schema = true
  gather_process_list = true
  gather_info_schema_auto_inc = true
  gather_slave_status = true

  # Timeout
  timeout = "5s"
```

### B. Monitoring d'une API REST personnalisée

```toml
[[inputs.http]]
  # URL de votre API
  urls = ["http://api.example.com/metrics"]

  # Headers d'authentification
  headers = {"Authorization" = "Bearer YOUR_TOKEN"}

  # Format des données (json, xml, csv...)
  data_format = "json"

  # Parser JSON
  json_query = "metrics"

  # Tag à ajouter
  [inputs.http.tags]
    source = "api"
```

### C. Monitoring de fichiers logs

```toml
[[inputs.tail]]
  # Fichiers à surveiller
  files = ["/var/log/app/*.log"]

  # Parser les logs
  data_format = "grok"

  # Pattern Grok
  grok_patterns = ["%{COMBINED_LOG_FORMAT}"]

  # Tags
  [inputs.tail.tags]
    log_type = "application"
```

### D. Collecte depuis un fichier CSV

```toml
[[inputs.file]]
  # Fichiers à lire
  files = ["/data/sensors/*.csv"]

  # Format
  data_format = "csv"

  # Colonnes
  csv_column_names = ["timestamp", "temperature", "humidity"]
  csv_timestamp_column = "timestamp"
  csv_timestamp_format = "2006-01-02 15:04:05"
```

---

## 🎓 Étape 16 : Cas d'usage réels

### A. Stack IoT complète (Capteurs → Telegraf → InfluxDB → Grafana)

**Architecture :**
```
[Capteurs] → MQTT → [Telegraf] → [InfluxDB] → [Grafana]
                          ↓
                     [Alertes]
```

**Configuration Telegraf pour IoT :**
```toml
[[inputs.mqtt_consumer]]
  servers = ["tcp://mosquitto:1883"]
  topics = ["sensors/#"]
  data_format = "json"

  json_string_fields = ["sensor_id", "location"]

  [inputs.mqtt_consumer.tags]
    source = "iot"
```

### B. Monitoring d'infrastructure multi-serveurs

**Sur chaque serveur, installer Telegraf avec :**
```toml
[agent]
  hostname = "server-paris-01"

[[outputs.influxdb_v2]]
  urls = ["http://influxdb-central.company.com:8086"]

[[inputs.cpu]]
[[inputs.mem]]
[[inputs.disk]]
[[inputs.net]]

[global_tags]
  datacenter = "paris"
  environment = "production"
```

### C. Monitoring d'application web (temps de réponse)

```toml
[[inputs.http_response]]
  urls = [
    "https://app.example.com/",
    "https://app.example.com/api/health",
    "https://app.example.com/login"
  ]

  response_timeout = "10s"

  # Extraire des métriques du body
  response_string_match = "\"status\":\"ok\""

  [inputs.http_response.tags]
    application = "web_app"
```

---

## 🔄 Étape 17 : Rotation des données et rétention

### A. Politique de rétention dans InfluxDB

Les buckets InfluxDB v2 ont une durée de rétention configurable.

**Créer un bucket avec rétention de 7 jours :**

Via l'interface InfluxDB :
1. **Settings** → **Buckets** → **Create Bucket**
2. **Name** : `telegraf_short_term`
3. **Retention** : `7 days`

Via CLI :
```bash
docker exec influxdb_server influx bucket create \
  --name telegraf_short_term \
  --org monitoring_org \
  --retention 168h \
  --token my_super_secret_token_12345
```

### B. Downsampling (agrégation des données anciennes)

Pour économiser l'espace, créez des tâches qui agrègent les vieilles données :

```flux
option task = {
  name: "downsample_cpu",
  every: 1h
}

from(bucket: "telegraf_metrics")
  |> range(start: -2h, stop: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> aggregateWindow(every: 5m, fn: mean)
  |> to(bucket: "telegraf_downsampled")
```

---

## 📊 Étape 18 : Alertes avancées avec Grafana

### A. Alerte sur utilisation CPU élevée

**Condition :** CPU > 80% pendant 5 minutes

1. Éditez le panel CPU
2. Onglet **Alert**
3. **Create alert rule**
4. **Condition** :
   ```
   WHEN avg() OF query(A, 5m, now) IS ABOVE 80
   ```
5. **Evaluate** : `every 1m` for `5m`
6. **Notification** : Email/Slack

### B. Alerte sur perte de métriques

**Condition :** Pas de données reçues depuis 2 minutes

```flux
from(bucket: "telegraf_metrics")
  |> range(start: -2m)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> count()
  |> map(fn: (r) => ({r with _value: if r._value == 0 then 1 else 0}))
```

Alert si `_value == 1` (aucune donnée).

---

## 🌐 Étape 19 : Accès depuis le réseau local

Si vous voulez accéder à votre stack depuis d'autres machines du réseau :

### A. Exposer les ports sur toutes les interfaces

**Dans `docker-compose.yml` :**
```yaml
services:
  influxdb:
    ports:
      - "0.0.0.0:8086:8086"  # Accessible depuis le réseau

  grafana:
    ports:
      - "0.0.0.0:3000:3000"  # Accessible depuis le réseau
```

### B. Trouver l'IP de votre machine

**Linux/macOS :**
```bash
ip addr show | grep inet
# ou
ifconfig | grep inet
```

**Windows :**
```cmd
ipconfig
```

**Exemple :** Si votre IP est `192.168.1.50`, accédez via :
- Grafana : `http://192.168.1.50:3000`
- InfluxDB : `http://192.168.1.50:8086`

### C. Configurer le pare-feu

**Linux (ufw) :**
```bash
sudo ufw allow 3000/tcp
sudo ufw allow 8086/tcp
```

**Windows :** Voir [fiche MariaDB - Accès réseau](../01-mariadb/05-acces-reseau-local.md)

---

## 🚀 Étape 20 : Optimisation des performances

### A. Augmenter le buffer de Telegraf

Pour des environnements à fort trafic :

```toml
[agent]
  # Augmenter la taille du buffer
  metric_batch_size = 5000
  metric_buffer_limit = 50000

  # Envoyer plus souvent
  flush_interval = "5s"
```

### B. Paralléliser les inputs

```toml
[agent]
  # Nombre de goroutines pour la collecte
  collection_jitter = "1s"

  # Paralléliser l'envoi
  flush_jitter = "1s"
```

### C. Optimiser InfluxDB

**Dans `docker-compose.yml` :**
```yaml
influxdb:
  environment:
    # Augmenter la cache
    - INFLUXDB_DATA_CACHE_MAX_MEMORY_SIZE=1073741824  # 1GB

    # Réduire la compression (plus rapide, mais plus d'espace)
    - INFLUXDB_DATA_CACHE_SNAPSHOT_WRITE_COLD_DURATION=10m
```

---

## 📦 Étape 21 : Sauvegarde et restauration

### A. Sauvegarder les données InfluxDB

**Méthode 1 : Sauvegarde du volume Docker**
```bash
# Arrêter les services
docker-compose stop

# Créer une archive du volume
docker run --rm \
  -v influxdb_telegraf_stack_influxdb_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/influxdb_backup_$(date +%Y%m%d).tar.gz -C /data .

# Redémarrer
docker-compose start
```

**Méthode 2 : Backup natif InfluxDB**
```bash
# Créer un backup
docker exec influxdb_server influx backup /tmp/backup \
  --token my_super_secret_token_12345

# Copier le backup hors du conteneur
docker cp influxdb_server:/tmp/backup ./influxdb_backup_$(date +%Y%m%d)
```

### B. Restaurer les données

```bash
# Copier le backup dans le conteneur
docker cp ./influxdb_backup_20241101 influxdb_server:/tmp/restore

# Restaurer
docker exec influxdb_server influx restore /tmp/restore \
  --token my_super_secret_token_12345
```

---

## ✅ Checklist de production

Avant de déployer en production, vérifiez :

- [ ] **Sécurité**
  - [ ] Changer tous les mots de passe par défaut
  - [ ] Utiliser des tokens forts et uniques
  - [ ] Activer HTTPS sur Grafana
  - [ ] Restreindre l'accès réseau (pare-feu)

- [ ] **Performance**
  - [ ] Configurer la rétention des données
  - [ ] Ajuster les buffers Telegraf
  - [ ] Optimiser les requêtes Flux

- [ ] **Monitoring**
  - [ ] Configurer les alertes critiques
  - [ ] Monitorer Telegraf lui-même
  - [ ] Mettre en place des sauvegardes automatiques

- [ ] **Documentation**
  - [ ] Documenter la configuration
  - [ ] Lister les métriques collectées
  - [ ] Créer un runbook pour les incidents

---

## 📖 Ressources utiles

### Documentation officielle
- 📘 [Telegraf Documentation](https://docs.influxdata.com/telegraf/)
- 📖 [Telegraf Plugins](https://docs.influxdata.com/telegraf/latest/plugins/)
- 🔧 [Telegraf Configuration](https://github.com/influxdata/telegraf/blob/master/etc/telegraf.conf)
- 🐙 [Image Docker Telegraf](https://hub.docker.com/_/telegraf)

### Communauté
- 💬 [InfluxData Community](https://community.influxdata.com/)
- 🐙 [Telegraf GitHub](https://github.com/influxdata/telegraf)
- 📚 [Exemples de configurations](https://github.com/influxdata/telegraf/tree/master/plugins)

### Tutoriels vidéo
- 🎥 [InfluxData YouTube](https://www.youtube.com/c/InfluxData)
- 🎓 [InfluxDB University](https://university.influxdata.com/)

---

## 🎯 Récapitulatif final

**Ce que vous maîtrisez maintenant :**

✅ **Architecture complète** Telegraf + InfluxDB + Grafana
✅ **Collecte automatique** de métriques système et applicatives
✅ **Configuration avancée** avec multiples plugins
✅ **Visualisation professionnelle** dans Grafana
✅ **Monitoring Docker** et conteneurs
✅ **Alertes** configurées et notifications
✅ **Optimisation** des performances
✅ **Sauvegarde** et restauration des données
✅ **Sécurité** et bonnes pratiques de production

**Vous êtes prêt pour :**
- Déployer une stack de monitoring en production
- Monitorer des infrastructures complexes
- Collecter des données IoT en temps réel
- Créer des dashboards professionnels
- Résoudre les problèmes de performance

---

## 🚀 Pour aller plus loin

Explorez d'autres sections du guide :

- **9.1** [Configuration basique InfluxDB](01-config-basique-v2.md) - Les fondamentaux
- **9.2** [Configuration avec IP fixe](02-config-ip-fixe.md) - Réseau stable
- **9.3** [InfluxDB avec Grafana](03-influxdb-grafana.md) - Visualisation avancée

**Autres bases de données :**
- **[MongoDB](../05-mongodb/README.md)** - Base NoSQL document
- **[Redis](../06-redis/README.md)** - Cache et temps réel
- **[Elasticsearch](../11-elasticsearch/README.md)** - Recherche et logs

**Cas pratiques :**
- **[Stack ELK](../cas-pratiques/03-stack-elk.md)** - Elasticsearch + Logstash + Kibana
- **[Environnement dev complet](../cas-pratiques/04-env-dev-complet.md)** - Multi-BDD

---

## 💬 Besoin d'aide ?

**Problème avec la configuration ?**
1. Consultez la section [Dépannage](#-étape-14--dépannage)
2. Vérifiez les logs : `docker-compose logs -f telegraf`
3. Testez la config : `telegraf --test`

**Voulez contribuer ?**
Ce guide est open source ! Proposez des améliorations sur GitHub.

---

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Novembre 2025*
