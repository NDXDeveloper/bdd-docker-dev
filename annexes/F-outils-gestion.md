# Annexe F - Outils de Gestion

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Gérer des bases de données via la ligne de commande est puissant, mais parfois fastidieux. Les **outils graphiques** (GUI) et de **monitoring** facilitent grandement le travail quotidien : visualiser les données, exécuter des requêtes, surveiller les performances, créer des backups automatiques.

**Ce que vous allez apprendre :**
- 🖥️ Choisir et utiliser des clients GUI pour chaque base de données
- 📊 Mettre en place des outils de monitoring
- 💾 Automatiser les sauvegardes
- 🔧 Intégrer ces outils avec Docker

**Pourquoi utiliser ces outils ?**
- ✅ **Productivité** : Visualiser et manipuler les données plus rapidement
- ✅ **Confort** : Interface intuitive vs lignes de commande
- ✅ **Sécurité** : Backups automatisés et monitoring des erreurs
- ✅ **Apprentissage** : Comprendre la structure des données visuellement

**Niveau :** 🟢 Débutant à 🟡 Intermédiaire

**Durée de lecture :** 50 minutes

---

## 📑 Table des Matières

1. [Clients GUI pour Bases de Données](#-1-clients-gui-pour-bases-de-données)
2. [Clients Universels](#-2-clients-universels)
3. [Outils de Monitoring](#-3-outils-de-monitoring)
4. [Outils de Backup](#-4-outils-de-backup)
5. [Intégration avec Docker](#-5-intégration-avec-docker)
6. [Recommandations par Profil](#-6-recommandations-par-profil)

---

## 🖥️ 1. Clients GUI pour Bases de Données

### 1.1 Qu'est-ce qu'un Client GUI ?

**GUI** = Graphical User Interface (Interface Utilisateur Graphique)

**Analogie :**
```
Ligne de commande = Conduire avec un tableau de bord de cockpit d'avion
Client GUI = Conduire une voiture normale avec tableau de bord clair
```

**Avec la ligne de commande :**
```bash
docker exec -it mariadb mariadb -u root -p
SELECT * FROM users WHERE active = 1;
```

**Avec un client GUI :**
```
[Fenêtre graphique]
🗂️ Tables → users → [Double-clic]
→ Les données s'affichent dans une grille
→ Filtrer : active = 1 [clic]
→ Résultats affichés visuellement
```

---

### 1.2 MariaDB / MySQL

#### phpMyAdmin (Interface Web)

**Description :** Le client web le plus populaire pour MySQL/MariaDB. S'exécute dans le navigateur.

**Avantages :**
- ✅ Gratuit et open source
- ✅ Pas d'installation sur votre PC (s'exécute dans Docker)
- ✅ Accessible depuis n'importe quel navigateur
- ✅ Interface intuitive

**Inconvénients :**
- ⚠️ Moins de fonctionnalités avancées
- ⚠️ Performances moyennes sur grosses bases

---

**Installation avec Docker Compose :**

```yaml
version: '3.8'

services:
  # Base de données MariaDB
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_server
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: app_db
    networks:
      - backend
    volumes:
      - mariadb_data:/var/lib/mysql

  # phpMyAdmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    environment:
      PMA_HOST: mariadb          # Nom du service MariaDB
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    ports:
      - "8080:80"                # Accessible sur localhost:8080
    networks:
      - backend
    depends_on:
      - mariadb

networks:
  backend:

volumes:
  mariadb_data:
```

**Utilisation :**
```bash
# Démarrer
docker-compose up -d

# Accéder dans le navigateur
http://localhost:8080

# Connexion :
# Serveur : mariadb
# Utilisateur : root
# Mot de passe : votre_mot_de_passe
```

**Fonctionnalités principales :**
- 📊 Visualiser les tables et données
- ✏️ Exécuter des requêtes SQL
- 🔧 Créer/modifier tables et bases
- 📤 Importer/Exporter des données (SQL, CSV)
- 👥 Gérer les utilisateurs et permissions

---

#### HeidiSQL (Application Desktop - Windows)

**Description :** Client léger et rapide pour Windows.

**Avantages :**
- ✅ Gratuit et open source
- ✅ Très rapide
- ✅ Multi-onglets (plusieurs connexions)
- ✅ Éditeur SQL avancé

**Inconvénients :**
- ❌ Windows uniquement

**Installation :**
1. Télécharger : https://www.heidisql.com/download.php
2. Installer normalement
3. Lancer HeidiSQL

**Connexion :**
```
Nouvelle connexion
├─ Type de réseau : MySQL/MariaDB
├─ Hôte : localhost
├─ Port : 3306
├─ Utilisateur : root
├─ Mot de passe : votre_mot_de_passe
└─ [Ouvrir]
```

---

#### MySQL Workbench (Multi-plateformes)

**Description :** Client officiel de MySQL/MariaDB, très complet.

**Avantages :**
- ✅ Gratuit
- ✅ Très complet (modélisation, diagrammes ER)
- ✅ Windows, macOS, Linux

**Inconvénients :**
- ⚠️ Interface complexe pour débutants
- ⚠️ Plus lourd (ressources)

**Installation :**
1. Télécharger : https://www.mysql.com/products/workbench/
2. Installer selon votre OS
3. Créer une connexion

**Utilisation avancée :**
- 📐 Modélisation de bases de données
- 🔄 Migrations de schémas
- 📊 Diagrammes ER (Entity-Relationship)
- 🔍 Analyse de performances

---

### 1.3 PostgreSQL

#### pgAdmin (Interface Web/Desktop)

**Description :** Le client officiel de PostgreSQL. Disponible en version web ou desktop.

**Avantages :**
- ✅ Gratuit et open source
- ✅ Client officiel (support complet)
- ✅ Interface moderne
- ✅ Version web (Docker) ou desktop

**Installation avec Docker Compose :**

```yaml
version: '3.8'

services:
  # Base de données PostgreSQL
  postgres:
    image: postgres:15
    container_name: postgres_server
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: app_db
    networks:
      - backend
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # pgAdmin
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}
    ports:
      - "5050:80"              # Accessible sur localhost:5050
    networks:
      - backend
    depends_on:
      - postgres

networks:
  backend:

volumes:
  postgres_data:
```

**Utilisation :**
```bash
# Accéder
http://localhost:5050

# Se connecter avec :
# Email : admin@example.com
# Mot de passe : votre_password_pgadmin

# Ajouter un serveur :
# Nom : PostgreSQL Local
# Host : postgres
# Port : 5432
# Username : postgres
# Password : votre_password_postgres
```

**Fonctionnalités :**
- 🗂️ Explorateur de bases et tables
- ✏️ Éditeur SQL avec autocomplétion
- 📊 Visualisation des données
- 🔧 Gestion des rôles et permissions
- 📈 Dashboard de monitoring

---

### 1.4 MongoDB

#### Mongo Express (Interface Web)

**Description :** Interface web simple pour MongoDB.

**Avantages :**
- ✅ Léger et rapide
- ✅ Facile à déployer avec Docker
- ✅ Bonne pour débuter

**Installation avec Docker Compose :**

```yaml
version: '3.8'

services:
  # MongoDB
  mongo:
    image: mongo:7
    container_name: mongodb_server
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
    networks:
      - backend
    volumes:
      - mongo_data:/data/db

  # Mongo Express
  mongo-express:
    image: mongo-express
    container_name: mongo_express
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_ROOT_PASSWORD}
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: ${MONGO_EXPRESS_PASSWORD}
    ports:
      - "8081:8081"
    networks:
      - backend
    depends_on:
      - mongo

networks:
  backend:

volumes:
  mongo_data:
```

**Utilisation :**
```bash
# Accéder
http://localhost:8081

# Authentification :
# Username : admin
# Password : votre_password_mongo_express
```

---

#### MongoDB Compass (Application Desktop)

**Description :** Client officiel de MongoDB, très visuel.

**Avantages :**
- ✅ Gratuit
- ✅ Interface moderne et intuitive
- ✅ Visualisation graphique des données
- ✅ Windows, macOS, Linux

**Installation :**
1. Télécharger : https://www.mongodb.com/products/compass
2. Installer
3. Se connecter

**Connexion :**
```
URI de connexion :
mongodb://root:password@localhost:27017/?authSource=admin
```

**Fonctionnalités :**
- 📊 Visualisation des documents JSON
- 🔍 Requêtes visuelles (query builder)
- 📈 Analyse de schéma
- 🎯 Indexation recommandée
- 📉 Statistiques de performances

---

### 1.5 Redis

#### RedisInsight (Application Desktop/Web)

**Description :** Client officiel de Redis avec interface moderne.

**Avantages :**
- ✅ Gratuit
- ✅ Interface très claire
- ✅ Monitoring intégré
- ✅ Browser de clés intelligent

**Installation Docker :**

```yaml
version: '3.8'

services:
  # Redis
  redis:
    image: redis:7-alpine
    container_name: redis_server
    command: redis-server --requirepass ${REDIS_PASSWORD}
    networks:
      - backend
    volumes:
      - redis_data:/data

  # RedisInsight
  redisinsight:
    image: redislabs/redisinsight:latest
    container_name: redisinsight
    ports:
      - "8001:8001"
    networks:
      - backend
    volumes:
      - redisinsight_data:/db

networks:
  backend:

volumes:
  redis_data:
  redisinsight_data:
```

**Utilisation :**
```bash
# Accéder
http://localhost:8001

# Ajouter une connexion :
# Host : redis
# Port : 6379
# Password : votre_redis_password
```

---

#### Redis Commander (Interface Web)

**Description :** Interface web légère pour Redis.

**Avantages :**
- ✅ Très léger
- ✅ Simple à déployer

**Installation Docker :**

```yaml
services:
  redis-commander:
    image: rediscommander/redis-commander
    container_name: redis_commander
    environment:
      REDIS_HOSTS: local:redis:6379:0:${REDIS_PASSWORD}
    ports:
      - "8082:8081"
    networks:
      - backend
    depends_on:
      - redis
```

---

### 1.6 Autres Bases de Données

#### Neo4j Browser (Intégré)

**Description :** Interface web intégrée à Neo4j.

```yaml
services:
  neo4j:
    image: neo4j:5
    ports:
      - "7474:7474"   # Browser
      - "7687:7687"   # Bolt
    environment:
      NEO4J_AUTH: neo4j/${NEO4J_PASSWORD}
```

**Accès :** http://localhost:7474

---

#### Grafana + InfluxDB

**Description :** Grafana pour visualiser les données InfluxDB.

```yaml
services:
  influxdb:
    image: influxdb:2

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

**Accès :** http://localhost:3000

---

## 🌐 2. Clients Universels

### 2.1 DBeaver Community (Recommandé)

**Description :** Client universel gratuit supportant la plupart des bases de données.

**Bases supportées :**
- ✅ MySQL / MariaDB
- ✅ PostgreSQL
- ✅ MongoDB
- ✅ SQLite
- ✅ SQL Server
- ✅ Oracle
- ✅ Cassandra
- ✅ Redis
- ✅ Et 80+ autres !

**Avantages :**
- ✅ **Un seul outil pour tout**
- ✅ Gratuit et open source
- ✅ Multi-plateformes (Windows, macOS, Linux)
- ✅ Interface moderne
- ✅ Éditeur SQL excellent
- ✅ Import/Export de données

**Inconvénients :**
- ⚠️ Peut être complexe pour débutants
- ⚠️ Plus lourd qu'un client spécialisé

---

**Installation :**

1. Télécharger : https://dbeaver.io/download/
2. Choisir la version Community (gratuite)
3. Installer selon votre OS

---

**Connexion à MariaDB :**

```
Nouvelle Connexion
├─ Sélectionner : MySQL
├─ Host : localhost
├─ Port : 3306
├─ Database : app_db
├─ Username : root
├─ Password : votre_password
└─ [Test Connection] → [Finish]
```

**Connexion à PostgreSQL :**

```
Nouvelle Connexion
├─ Sélectionner : PostgreSQL
├─ Host : localhost
├─ Port : 5432
├─ Database : app_db
├─ Username : postgres
├─ Password : votre_password
└─ [Test Connection] → [Finish]
```

**Connexion à MongoDB :**

```
Nouvelle Connexion
├─ Sélectionner : MongoDB
├─ Connection String : mongodb://root:password@localhost:27017/?authSource=admin
└─ [Test Connection] → [Finish]
```

---

**Fonctionnalités principales :**

| Fonctionnalité | Description |
|----------------|-------------|
| 📊 **Explorateur de données** | Naviguer dans les bases, tables, colonnes |
| ✏️ **Éditeur SQL** | Autocomplétion, coloration syntaxique |
| 📈 **Diagrammes ER** | Visualiser les relations entre tables |
| 📤 **Import/Export** | CSV, JSON, SQL, Excel |
| 🔍 **Recherche globale** | Chercher dans toutes les tables |
| 📋 **Générateur de requêtes** | Créer des requêtes visuellement |
| 🎨 **Thèmes** | Personnaliser l'interface |

---

### 2.2 DataGrip (JetBrains - Payant)

**Description :** IDE professionnel pour bases de données par JetBrains.

**Avantages :**
- ✅ Interface très puissante
- ✅ Refactoring SQL intelligent
- ✅ Intégration Git
- ✅ Support de toutes les BDD

**Inconvénients :**
- 💰 Payant (149€/an)
- ⚠️ Courbe d'apprentissage

**Pour qui ?**
- Développeurs professionnels
- Équipes déjà abonnées JetBrains

**Site :** https://www.jetbrains.com/datagrip/

---

### 2.3 TablePlus (Freemium)

**Description :** Client moderne pour macOS, Windows, Linux.

**Avantages :**
- ✅ Interface très élégante
- ✅ Rapide et léger
- ✅ Multi-onglets

**Inconvénients :**
- 💰 Version gratuite limitée (2 connexions max)
- 💰 Version complète : 89$ one-time

**Pour qui ?**
- Développeurs macOS
- Ceux qui veulent une interface belle

**Site :** https://tableplus.com/

---

## 📊 3. Outils de Monitoring

### 3.1 Pourquoi Monitorer ?

**Sans monitoring :**
```
Votre base de données rame
    ↓
Vous ne savez pas pourquoi
    ↓
Vous devinez et testez au hasard
    ↓
💥 Problème persistant
```

**Avec monitoring :**
```
Alerte : CPU à 95% sur MariaDB
    ↓
Dashboard : Requête X prend 30 secondes
    ↓
Vous optimisez cette requête
    ↓
✅ Problème résolu en 5 minutes
```

---

### 3.2 Portainer (Monitoring Docker)

**Description :** Interface web pour gérer Docker, avec monitoring intégré.

**Fonctionnalités :**
- 📊 Utilisation CPU/RAM/Disque
- 📝 Logs en temps réel
- 🔧 Gestion des conteneurs (start/stop/restart)
- 🌐 Gestion des réseaux et volumes
- 📈 Statistiques historiques

---

**Installation :**

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

**Utilisation :**

```bash
# Démarrer
docker-compose up -d

# Accéder
http://localhost:9000

# Premier lancement :
# 1. Créer un compte admin
# 2. Choisir "Local" (Docker local)
# 3. Explorer l'interface
```

**Dashboard :**
- Voir tous les conteneurs en un coup d'œil
- Stats en temps réel
- Actions rapides (redémarrer, logs, shell)

---

### 3.3 cAdvisor (Métriques Détaillées)

**Description :** Outil de monitoring de Google pour conteneurs.

**Avantages :**
- ✅ Métriques très détaillées
- ✅ Graphiques en temps réel
- ✅ Historique des performances

---

**Installation :**

```yaml
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    privileged: true
```

**Utilisation :**

```bash
# Accéder
http://localhost:8080

# Voir les métriques :
# - CPU usage
# - Memory usage
# - Network I/O
# - Disk I/O
# - Historique sur 1 minute
```

---

### 3.4 Grafana + Prometheus (Stack Complète)

**Description :** Solution professionnelle de monitoring avec dashboards personnalisables.

**Architecture :**
```
Base de données
    ↓ (expose métriques)
Prometheus (collecte)
    ↓ (stocke)
Grafana (visualise)
    ↓
Dashboard avec graphiques
```

---

**Installation Complète :**

```yaml
version: '3.8'

services:
  # Votre base de données
  mariadb:
    image: mariadb:10.11
    # ... (config habituelle)

  # Prometheus (collecteur de métriques)
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  # Grafana (visualisation)
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - prometheus

  # Node Exporter (métriques système)
  node-exporter:
    image: prom/node-exporter
    container_name: node_exporter
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
  grafana_data:
```

**Fichier prometheus.yml :**

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

---

**Utilisation Grafana :**

```bash
# Accéder
http://localhost:3000

# Connexion :
# Username : admin
# Password : votre_grafana_password

# Configuration :
1. Ajouter Data Source → Prometheus
   URL : http://prometheus:9090
   [Save & Test]

2. Importer un dashboard :
   + → Import → ID 1860 (Node Exporter Full)

3. Profiter des graphiques !
```

**Dashboards disponibles :**
- CPU, RAM, Disk usage
- Network traffic
- Requêtes par seconde
- Latence moyenne
- Alertes personnalisables

---

### 3.5 Monitoring Spécifique par BDD

#### MySQL/MariaDB : MySQL Exporter

```yaml
services:
  mysql-exporter:
    image: prom/mysqld-exporter
    environment:
      DATA_SOURCE_NAME: "root:password@(mariadb:3306)/"
    ports:
      - "9104:9104"
```

---

#### PostgreSQL : PostgreSQL Exporter

```yaml
services:
  postgres-exporter:
    image: prometheuscommunity/postgres-exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://postgres:password@postgres:5432/postgres?sslmode=disable"
    ports:
      - "9187:9187"
```

---

#### MongoDB : MongoDB Exporter

```yaml
services:
  mongodb-exporter:
    image: percona/mongodb_exporter
    environment:
      MONGODB_URI: "mongodb://root:password@mongo:27017"
    ports:
      - "9216:9216"
```

---

### 3.6 Outils de Monitoring Simples

#### Docker Stats (Ligne de Commande)

```bash
# Monitoring en temps réel
docker stats

# Résultat :
# CONTAINER   CPU %   MEM USAGE / LIMIT     MEM %
# mariadb     2.5%    450MB / 2GB          22.5%
# redis       0.5%    50MB / 512MB         9.8%
```

---

#### Script de Monitoring Personnel

**Créer `monitor.sh` :**

```bash
#!/bin/bash

echo "=== Monitoring Docker ==="
echo ""

echo "📊 Ressources :"
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

echo ""
echo "💾 Espace disque :"
docker system df

echo ""
echo "🔍 Conteneurs en erreur :"
docker ps -a --filter "status=exited" --filter "status=restarting"

echo ""
echo "📝 Logs récents avec erreurs :"
for container in $(docker ps -q); do
    name=$(docker inspect --format='{{.Name}}' $container | sed 's/\///')
    errors=$(docker logs --since 1h $container 2>&1 | grep -i "error" | wc -l)
    if [ $errors -gt 0 ]; then
        echo "  $name: $errors erreurs"
    fi
done
```

**Utilisation :**

```bash
chmod +x monitor.sh
./monitor.sh

# Surveiller en continu (refresh toutes les 5 secondes)
watch -n 5 ./monitor.sh
```

---

## 💾 4. Outils de Backup

### 4.1 Stratégies de Backup

#### Les 3-2-1 Rule

```
3 copies de vos données
2 sur des supports différents
1 hors site (cloud, autre localisation)
```

**Exemple :**
```
1. Données en production (Docker volume)
2. Backup local (disque externe)
3. Backup cloud (AWS S3, Google Drive)
```

---

### 4.2 Scripts de Backup Manuels

#### Backup MariaDB/MySQL

**Script `backup-mariadb.sh` :**

```bash
#!/bin/bash

# Configuration
CONTAINER="mariadb_server"
BACKUP_DIR="./backups/mariadb"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/backup_${TIMESTAMP}.sql.gz"

# Créer le dossier de backup
mkdir -p "$BACKUP_DIR"

# Backup
echo "🔄 Backup de MariaDB..."
docker exec $CONTAINER mysqldump \
    -u root -p"${MYSQL_ROOT_PASSWORD}" \
    --all-databases \
    --single-transaction \
    --quick \
    | gzip > "$BACKUP_FILE"

if [ $? -eq 0 ]; then
    echo "✅ Backup créé : $BACKUP_FILE"
    ls -lh "$BACKUP_FILE"

    # Rotation : garder seulement les 7 derniers
    find "$BACKUP_DIR" -name "backup_*.sql.gz" -mtime +7 -delete
    echo "🧹 Anciens backups supprimés (> 7 jours)"
else
    echo "❌ Erreur lors du backup"
    exit 1
fi
```

---

#### Backup PostgreSQL

**Script `backup-postgres.sh` :**

```bash
#!/bin/bash

CONTAINER="postgres_server"
BACKUP_DIR="./backups/postgres"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/backup_${TIMESTAMP}.sql.gz"

mkdir -p "$BACKUP_DIR"

echo "🔄 Backup de PostgreSQL..."
docker exec $CONTAINER pg_dumpall -U postgres | gzip > "$BACKUP_FILE"

if [ $? -eq 0 ]; then
    echo "✅ Backup créé : $BACKUP_FILE"

    # Rotation
    find "$BACKUP_DIR" -name "backup_*.sql.gz" -mtime +7 -delete
else
    echo "❌ Erreur lors du backup"
    exit 1
fi
```

---

#### Backup MongoDB

**Script `backup-mongo.sh` :**

```bash
#!/bin/bash

CONTAINER="mongo_server"
BACKUP_DIR="./backups/mongo"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

echo "🔄 Backup de MongoDB..."
docker exec $CONTAINER mongodump \
    --username root \
    --password "${MONGO_ROOT_PASSWORD}" \
    --authenticationDatabase admin \
    --out /tmp/backup

docker cp $CONTAINER:/tmp/backup "$BACKUP_DIR/backup_${TIMESTAMP}"

if [ $? -eq 0 ]; then
    echo "✅ Backup créé : $BACKUP_DIR/backup_${TIMESTAMP}"

    # Compression
    cd "$BACKUP_DIR"
    tar czf "backup_${TIMESTAMP}.tar.gz" "backup_${TIMESTAMP}"
    rm -rf "backup_${TIMESTAMP}"

    # Rotation
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
else
    echo "❌ Erreur lors du backup"
    exit 1
fi
```

---

### 4.3 Automatisation des Backups

#### Avec Cron (Linux/macOS)

```bash
# Éditer le crontab
crontab -e

# Ajouter (backup quotidien à 2h du matin)
0 2 * * * /chemin/vers/backup-mariadb.sh >> /var/log/backup.log 2>&1

# Backup hebdomadaire (dimanche à 3h)
0 3 * * 0 /chemin/vers/backup-mariadb.sh >> /var/log/backup.log 2>&1
```

---

#### Avec Task Scheduler (Windows)

```powershell
# Créer une tâche planifiée
$Action = New-ScheduledTaskAction -Execute "bash" -Argument "C:\projet\backup-mariadb.sh"
$Trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName "Backup MariaDB" -Action $Action -Trigger $Trigger
```

---

### 4.4 Outils de Backup Automatisés

#### Duplicati (Backup Complet avec Interface Web)

**Description :** Solution de backup complète avec chiffrement et support cloud.

**Fonctionnalités :**
- ✅ Interface web intuitive
- ✅ Chiffrement AES-256
- ✅ Backup incrémentiel
- ✅ Support cloud (AWS S3, Google Drive, Dropbox, etc.)
- ✅ Planification automatique
- ✅ Restauration facile

---

**Installation :**

```yaml
version: '3.8'

services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/Paris
    volumes:
      - ./duplicati_config:/config
      - ./backups:/backups           # Backups locaux
      - ./data:/source:ro             # Données à sauvegarder (read-only)
    ports:
      - "8200:8200"
    restart: unless-stopped
```

**Utilisation :**

```bash
# Accéder
http://localhost:8200

# Configuration :
1. Ajouter un backup
2. Choisir la destination (local ou cloud)
3. Sélectionner les dossiers à sauvegarder
4. Planifier (quotidien, hebdomadaire)
5. Activer le chiffrement
6. [Enregistrer]
```

---

#### Restic (Backup en Ligne de Commande)

**Description :** Outil moderne de backup avec déduplication.

**Avantages :**
- ✅ Très rapide
- ✅ Déduplication automatique
- ✅ Chiffrement intégré
- ✅ Support multi-cloud

---

**Installation et Usage :**

```bash
# Installer Restic
# Linux
sudo apt-get install restic

# macOS
brew install restic

# Initialiser un dépôt de backup
restic init --repo /chemin/vers/backup

# Backup
restic backup /chemin/vers/data --repo /chemin/vers/backup

# Restaurer
restic restore latest --repo /chemin/vers/backup --target /chemin/restauration

# Lister les backups
restic snapshots --repo /chemin/vers/backup
```

---

#### BorgBackup (Déduplication Avancée)

**Description :** Backup avec déduplication et compression.

**Avantages :**
- ✅ Déduplication très efficace
- ✅ Compression
- ✅ Chiffrement
- ✅ Parfait pour grandes quantités de données

---

### 4.5 Backup vers le Cloud

#### AWS S3

**Script avec AWS CLI :**

```bash
#!/bin/bash

# Backup local
./backup-mariadb.sh

# Synchroniser vers S3
aws s3 sync ./backups/mariadb s3://mon-bucket/backups/mariadb/ \
    --storage-class GLACIER \
    --exclude "*" --include "*.sql.gz"

echo "✅ Backup synchronisé avec AWS S3"
```

---

#### Google Drive avec rclone

```bash
# Installer rclone
curl https://rclone.org/install.sh | sudo bash

# Configurer Google Drive
rclone config

# Synchroniser
rclone sync ./backups/mariadb gdrive:backups/mariadb
```

---

## 🐳 5. Intégration avec Docker

### 5.1 Stack Complète de Gestion

**docker-compose.yml complet avec tous les outils :**

```yaml
version: '3.8'

services:
  # === Base de données ===
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_server
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: app_db
    networks:
      - backend
    volumes:
      - mariadb_data:/var/lib/mysql

  # === GUI : phpMyAdmin ===
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    environment:
      PMA_HOST: mariadb
    ports:
      - "8080:80"
    networks:
      - backend
    depends_on:
      - mariadb

  # === Monitoring : Portainer ===
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: always

  # === Monitoring : Grafana ===
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
    restart: always

  # === Backup : Duplicati ===
  duplicati:
    image: lscr.io/linuxserver/duplicati
    container_name: duplicati
    environment:
      PUID: 1000
      PGID: 1000
    volumes:
      - ./duplicati_config:/config
      - ./backups:/backups
      - mariadb_data:/source/mariadb:ro
    ports:
      - "8200:8200"
    restart: unless-stopped

networks:
  backend:

volumes:
  mariadb_data:
  portainer_data:
  grafana_data:
```

**Points d'accès :**
- phpMyAdmin : http://localhost:8080
- Portainer : http://localhost:9000
- Grafana : http://localhost:3000
- Duplicati : http://localhost:8200

---

### 5.2 Stack Multi-BDD avec Outils

```yaml
version: '3.8'

services:
  # MariaDB + phpMyAdmin
  mariadb:
    image: mariadb:10.11
    networks: [backend]
    volumes: [mariadb_data:/var/lib/mysql]
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports: ["8080:80"]
    networks: [backend]
    environment:
      PMA_HOST: mariadb

  # PostgreSQL + pgAdmin
  postgres:
    image: postgres:15
    networks: [backend]
    volumes: [postgres_data:/var/lib/postgresql/data]
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  pgadmin:
    image: dpage/pgadmin4
    ports: ["5050:80"]
    networks: [backend]
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}

  # MongoDB + Mongo Express
  mongo:
    image: mongo:7
    networks: [backend]
    volumes: [mongo_data:/data/db]
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}

  mongo-express:
    image: mongo-express
    ports: ["8081:8081"]
    networks: [backend]
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_PASSWORD}

  # Redis + RedisInsight
  redis:
    image: redis:7-alpine
    networks: [backend]
    command: redis-server --requirepass ${REDIS_PASSWORD}

  redisinsight:
    image: redislabs/redisinsight
    ports: ["8001:8001"]
    networks: [backend]

  # Monitoring global
  portainer:
    image: portainer/portainer-ce
    ports: ["9000:9000"]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

networks:
  backend:

volumes:
  mariadb_data:
  postgres_data:
  mongo_data:
  portainer_data:
```

---

## 👥 6. Recommandations par Profil

### 6.1 Débutant (Apprenant)

**Outils recommandés :**

| Besoin | Outil | Pourquoi |
|--------|-------|----------|
| **Client SQL** | DBeaver Community | Gratuit, simple, universel |
| **GUI Web** | phpMyAdmin / pgAdmin | Facile à déployer avec Docker |
| **Monitoring** | Portainer | Interface claire, tout-en-un |
| **Backup** | Scripts manuels | Comprendre les mécanismes |

**Stack minimale :**
```yaml
services:
  mariadb:
    image: mariadb:10.11
    # ... config de base

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports: ["8080:80"]

  portainer:
    image: portainer/portainer-ce
    ports: ["9000:9000"]
```

---

### 6.2 Développeur Intermédiaire

**Outils recommandés :**

| Besoin | Outil | Pourquoi |
|--------|-------|----------|
| **Client SQL** | DBeaver ou TablePlus | Productivité accrue |
| **Monitoring** | Portainer + cAdvisor | Métriques détaillées |
| **Backup** | Scripts automatisés + Duplicati | Automatisation |
| **Alertes** | Grafana (optionnel) | Visualisation avancée |

---

### 6.3 DevOps / Production

**Outils recommandés :**

| Besoin | Outil | Pourquoi |
|--------|-------|----------|
| **Client SQL** | DBeaver ou DataGrip | Fonctionnalités avancées |
| **Monitoring** | Grafana + Prometheus | Stack complète |
| **Backup** | Restic ou Borg + S3 | Déduplication, cloud |
| **Alertes** | Prometheus Alertmanager | Notifications automatiques |
| **Logs** | ELK Stack (Elasticsearch, Logstash, Kibana) | Centralisation |

---

### 6.4 Équipe / Startup

**Outils recommandés :**

| Besoin | Outil | Pourquoi |
|--------|-------|----------|
| **Client SQL** | DBeaver (licence team) | Collaboration |
| **GUI Web** | Outils web (accès partagé) | Pas d'installation individuelle |
| **Monitoring** | Grafana Cloud | Hébergé, pas de maintenance |
| **Backup** | AWS Backup ou Duplicati | Automatisé, fiable |
| **Documentation** | Confluence + Diagrammes | Connaissance partagée |

---

## 📊 Tableaux Comparatifs

### Clients GUI par Base de Données

| Base | Client Web | Client Desktop | Client Universel |
|------|------------|----------------|------------------|
| **MariaDB/MySQL** | phpMyAdmin | HeidiSQL (Win), MySQL Workbench | DBeaver, DataGrip |
| **PostgreSQL** | pgAdmin | pgAdmin Desktop | DBeaver, DataGrip |
| **MongoDB** | Mongo Express | MongoDB Compass | DBeaver, Studio 3T |
| **Redis** | Redis Commander | RedisInsight | Medis, Another Redis Desktop Manager |
| **SQLite** | - | DB Browser for SQLite | DBeaver |
| **Neo4j** | Neo4j Browser (intégré) | Neo4j Desktop | - |
| **Cassandra** | - | DataStax DevCenter | DBeaver |

---

### Outils de Monitoring

| Outil | Type | Complexité | Fonctionnalités | Gratuit |
|-------|------|------------|-----------------|---------|
| **Portainer** | Web | 🟢 Facile | Gestion Docker + monitoring basique | ✅ |
| **cAdvisor** | Web | 🟢 Facile | Métriques conteneurs | ✅ |
| **Grafana** | Web | 🟡 Moyen | Dashboards avancés | ✅ |
| **Prometheus** | Backend | 🟡 Moyen | Collecte métriques | ✅ |
| **Datadog** | SaaS | 🟢 Facile | Monitoring complet | 💰 Payant |
| **New Relic** | SaaS | 🟢 Facile | APM complet | 💰 Freemium |

---

### Outils de Backup

| Outil | Type | Chiffrement | Cloud | Déduplication | Gratuit |
|-------|------|-------------|-------|---------------|---------|
| **Scripts manuels** | CLI | ❌ | ❌ | ❌ | ✅ |
| **Duplicati** | Web | ✅ | ✅ | ✅ | ✅ |
| **Restic** | CLI | ✅ | ✅ | ✅ | ✅ |
| **BorgBackup** | CLI | ✅ | ⚠️ Via rclone | ✅ | ✅ |
| **Veeam** | Desktop | ✅ | ✅ | ✅ | 💰 Payant |
| **AWS Backup** | SaaS | ✅ | ✅ | ✅ | 💰 Pay-as-you-go |

---

## 💡 Conseils Pratiques

### Bonnes Pratiques

#### Pour les Clients GUI

✅ **Utilisez des connexions nommées**
```
"Prod - MariaDB"
"Dev - PostgreSQL"
"Test - MongoDB"
```

✅ **Sauvegardez vos connexions**
- DBeaver : File → Export → Connections
- Stocker dans un endroit sûr

✅ **Utilisez des couleurs différentes**
```
Production : Rouge
Staging : Orange
Développement : Vert
```

✅ **Lisez en read-only en production**
```
Pour éviter les erreurs, connexion en lecture seule
```

---

#### Pour le Monitoring

✅ **Configurez des alertes**
```yaml
# Exemple Grafana
Alert si CPU > 80% pendant 5 minutes
Alert si Mémoire > 90%
Alert si Disque > 85%
```

✅ **Gardez l'historique**
```
Minimum 30 jours pour analyser les tendances
```

✅ **Dashboard par environnement**
```
Dashboard "Production"
Dashboard "Staging"
Dashboard "Développement"
```

---

#### Pour les Backups

✅ **Testez vos restaurations**
```bash
# Au moins une fois par mois
./backup-mariadb.sh
./restore-mariadb.sh backup.sql.gz
# Vérifier que tout fonctionne
```

✅ **Rotation automatique**
```
Garder : 7 backups quotidiens
        4 backups hebdomadaires
        12 backups mensuels
```

✅ **Backups hors site**
```
Ne pas garder uniquement sur le même serveur !
```

✅ **Documentation**
```
Documenter la procédure de restauration
(pour vous dans 6 mois ou pour un collègue)
```

---

## 🎓 Conclusion

Les outils de gestion sont essentiels pour travailler efficacement avec Docker et les bases de données. Ils vous font gagner du temps, réduisent les erreurs, et vous permettent de vous concentrer sur le développement plutôt que sur l'administration.

**Récapitulatif des choix :**

```
Client GUI :
├─ Débutant → DBeaver Community + phpMyAdmin
├─ Intermédiaire → DBeaver ou TablePlus
└─ Pro → DataGrip

Monitoring :
├─ Simple → Portainer
├─ Avancé → Grafana + Prometheus
└─ Entreprise → Datadog ou New Relic

Backup :
├─ Manuel → Scripts bash
├─ Automatisé → Duplicati
└─ Production → Restic/Borg + Cloud
```

**N'oubliez pas :**
- 🔧 **Commencez simple** : Pas besoin de tout installer d'un coup
- 📈 **Évoluez progressivement** : Ajoutez des outils au fur et à mesure
- 🧪 **Testez** : Surtout les backups !
- 📝 **Documentez** : Vos configurations et procédures

---

## 🚀 Pour Aller Plus Loin

### Ressources

- 📖 [DBeaver Documentation](https://dbeaver.com/docs/)
- 📖 [Grafana Tutorials](https://grafana.com/tutorials/)
- 📖 [Duplicati Manual](https://duplicati.readthedocs.io/)
- 📖 [Portainer Documentation](https://docs.portainer.io/)

### Annexes Connexes

- **[Annexe A - Commandes](A-reference-commandes.md)** - Commandes de gestion
- **[Annexe E - Dépannage](E-depannage.md)** - Diagnostiquer les problèmes
- **[Annexe D - Sécurité](D-securite-bonnes-pratiques.md)** - Sécuriser les outils

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Les bons outils font les bons artisans ! 🛠️*
