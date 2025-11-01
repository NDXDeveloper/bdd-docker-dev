# 9.3 InfluxDB avec Grafana

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**InfluxDB** est excellent pour stocker des données de séries temporelles (métriques, capteurs, logs...), mais son interface web est basique pour la visualisation.

**Grafana** est la solution de référence pour créer des **tableaux de bord magnifiques et interactifs** à partir de vos données InfluxDB.

Ensemble, ils forment une **stack de monitoring professionnelle** utilisée par des milliers d'entreprises dans le monde.

**Cas d'usage typiques :**
- 📊 **Monitoring système** (CPU, RAM, disque, réseau)
- 🌡️ **Tableaux de bord IoT** (température, humidité, capteurs)
- 📈 **Métriques applicatives** (temps de réponse, erreurs, utilisateurs)
- 💹 **Données financières** (cours de bourse, transactions)
- 🔋 **Monitoring infrastructure** (serveurs, containers, bases de données)

---

## 🎯 Ce que vous allez apprendre

✅ Comprendre l'architecture InfluxDB + Grafana
✅ Déployer les deux services ensemble avec Docker Compose
✅ Configurer la connexion entre Grafana et InfluxDB
✅ Insérer des données de test dans InfluxDB
✅ Créer votre premier tableau de bord dans Grafana
✅ Utiliser les panels, graphiques et requêtes Flux
✅ Configurer des alertes (optionnel)

---

## 🔑 Qu'est-ce que Grafana ?

**Grafana** est une plateforme open-source de **visualisation et d'analyse de données**.

**Fonctionnalités principales :**
- 📊 **Tableaux de bord interactifs** avec multiples types de graphiques
- 🔌 **Connexion à de multiples sources** (InfluxDB, Prometheus, MySQL, PostgreSQL...)
- 🚨 **Système d'alertes** avec notifications (email, Slack, Discord...)
- 👥 **Gestion des utilisateurs** et des permissions
- 📱 **Responsive** : fonctionne sur mobile et tablette
- 🎨 **Hautement personnalisable** (thèmes, plugins, variables...)

**Interface Grafana :**
```
┌─────────────────────────────────────────┐
│  🏠 Grafana Dashboard                   │
├─────────────────────────────────────────┤
│                                         │
│  📊 CPU Usage        📈 Memory Usage    │
│  [Graphique ligne]   [Graphique jauge]  │
│                                         │
│  🌡️ Temperature      📉 Disk Space      │
│  [Graphique courbe]  [Graphique barre]  │
│                                         │
└─────────────────────────────────────────┘
```

---

## 🏗️ Architecture de la stack

```
┌──────────────┐     API      ┌──────────────┐
│              │  ←────────→  │              │
│   Grafana    │   Requêtes   │   InfluxDB   │
│  (Port 3000) │     Flux     │  (Port 8086) │
│              │              │              │
└──────────────┘              └──────────────┘
       ↑                             ↑
       │                             │
       │                             │
   Navigateur                   Applications
   (Visualisation)              (Écriture données)
```

**Flux de données :**
1. Les **applications/capteurs** envoient des données à **InfluxDB**
2. **Grafana** interroge **InfluxDB** pour récupérer les données
3. **Grafana** affiche les données sous forme de graphiques
4. L'**utilisateur** accède aux tableaux de bord via son navigateur

---

## 📦 Prérequis

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Un **navigateur web** moderne
- ✅ Avoir lu les fiches [9.1 - Configuration basique](01-config-basique-v2.md) (recommandé)

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
mkdir influxdb_grafana_stack
cd influxdb_grafana_stack
```

### B. Créer le fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec le contenu suivant :

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
      # Configuration initiale automatique
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_ORG=monitoring_org
      - DOCKER_INFLUXDB_INIT_BUCKET=metrics
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      # ⚠️ CHANGEZ ces mots de passe en production !
      - DOCKER_INFLUXDB_INIT_PASSWORD=admin_password_123
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my_super_secret_token_12345

  # ============================================
  # SERVICE GRAFANA
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
      # Configuration de base de Grafana
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      # Permet les connexions anonymes (optionnel, désactivez en production)
      - GF_AUTH_ANONYMOUS_ENABLED=false
    depends_on:
      - influxdb

volumes:
  influxdb_data:
  influxdb_config:
  grafana_data:
```

**🔑 Points importants :**

- **InfluxDB** :
  - Port `8086` : API et interface web
  - Bucket par défaut : `metrics`
  - Token d'admin : `my_super_secret_token_12345`

- **Grafana** :
  - Port `3000` : Interface web
  - Utilisateur par défaut : `admin` / `admin`
  - `depends_on: influxdb` : Grafana attend qu'InfluxDB démarre

- **Volumes** :
  - Persistance des données même après suppression des conteneurs

---

## ▶️ Étape 2 : Lancement de la stack

### A. Démarrer les services

Dans le dossier contenant votre `docker-compose.yml`, exécutez :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating network "influxdb_grafana_stack_default" with the default driver
Creating volume "influxdb_grafana_stack_influxdb_data" with default driver
Creating volume "influxdb_grafana_stack_influxdb_config" with default driver
Creating volume "influxdb_grafana_stack_grafana_data" with default driver
Creating influxdb_server ... done
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
```

Les deux services doivent être `Up`.

### C. Consulter les logs (optionnel)

Pour voir ce qui se passe :

```bash
# Logs d'InfluxDB
docker-compose logs -f influxdb

# Logs de Grafana
docker-compose logs -f grafana
```

Appuyez sur `Ctrl+C` pour quitter les logs.

---

## 🌐 Étape 3 : Vérification des interfaces

### A. Accéder à InfluxDB

Ouvrez votre navigateur et allez à :

```
http://localhost:8086
```

**Connexion :**
- Username : `admin`
- Password : `admin_password_123`

✅ Vous devriez voir le tableau de bord InfluxDB.

### B. Accéder à Grafana

Ouvrez un nouvel onglet et allez à :

```
http://localhost:3000
```

**Première connexion :**
- Username : `admin`
- Password : `admin`

Grafana vous demandera de **changer le mot de passe** au premier login (vous pouvez le garder comme `admin` pour le développement).

✅ Vous devriez voir la page d'accueil de Grafana.

---

## 🔗 Étape 4 : Connecter Grafana à InfluxDB

Maintenant, nous allons configurer Grafana pour qu'il puisse lire les données depuis InfluxDB.

### A. Ajouter une source de données

1. Dans Grafana, cliquez sur l'icône **☰** (menu hamburger) en haut à gauche
2. Allez dans **"Connections"** → **"Data sources"**
3. Cliquez sur **"Add data source"**
4. Cherchez et sélectionnez **"InfluxDB"**

### B. Configuration de la source InfluxDB

Remplissez les champs suivants :

**Section "HTTP" :**
- **Name** : `InfluxDB-Monitoring` (ou le nom que vous voulez)
- **URL** : `http://influxdb:8086`
  - 💡 Utilisez le **nom du service** (`influxdb`) car les conteneurs sont sur le même réseau Docker

**Section "InfluxDB Details" :**
- **Query Language** : Sélectionnez **"Flux"** (langage de requête v2.x)
- **Organization** : `monitoring_org` (celui défini dans docker-compose)
- **Token** : `my_super_secret_token_12345` (le token défini dans docker-compose)
- **Default Bucket** : `metrics` (le bucket par défaut)

### C. Tester et sauvegarder

1. Faites défiler en bas de la page
2. Cliquez sur **"Save & Test"**
3. Vous devriez voir un message vert : **"✓ datasource is working. 1 buckets found"**

🎉 **Félicitations !** Grafana est maintenant connecté à InfluxDB.

---

## 📊 Étape 5 : Insérer des données de test dans InfluxDB

Avant de créer des graphiques, nous avons besoin de données. Insérons quelques métriques de test.

### A. Via l'interface web InfluxDB

1. Ouvrez InfluxDB : `http://localhost:8086`
2. Cliquez sur **"Data Explorer"** (📊) dans la barre latérale
3. Cliquez sur l'onglet **"Write Data"**
4. Collez ces données dans le champ :

```
# Données de CPU
cpu_usage,host=server1,region=eu value=45.5
cpu_usage,host=server2,region=us value=62.3
cpu_usage,host=server3,region=asia value=38.2

# Données de mémoire
memory_usage,host=server1,region=eu value=78.2
memory_usage,host=server2,region=us value=85.5
memory_usage,host=server3,region=asia value=67.8

# Données de température
temperature,location=room1,sensor=A value=22.5
temperature,location=room2,sensor=B value=24.0
temperature,location=room3,sensor=C value=21.8
```

5. Sélectionnez **Bucket** : `metrics`
6. Cliquez sur **"Write Data"**

### B. Via la ligne de commande (alternative)

Vous pouvez aussi insérer des données via le CLI :

```bash
# Se connecter au conteneur InfluxDB
docker exec -it influxdb_server bash

# Insérer des données
influx write \
  --bucket metrics \
  --org monitoring_org \
  --token my_super_secret_token_12345 \
  --precision s \
  "cpu_usage,host=server1,region=eu value=45.5"

# Quitter
exit
```

### C. Script pour générer des données continues (optionnel)

Créez un fichier `generate_metrics.sh` :

```bash
#!/bin/bash

# Boucle infinie pour générer des données
while true; do
  # Générer une valeur aléatoire entre 20 et 100
  CPU_VALUE=$((RANDOM % 80 + 20))

  # Insérer dans InfluxDB
  docker exec influxdb_server influx write \
    --bucket metrics \
    --org monitoring_org \
    --token my_super_secret_token_12345 \
    --precision s \
    "cpu_usage,host=server1,region=eu value=$CPU_VALUE"

  echo "Inserted CPU value: $CPU_VALUE"

  # Attendre 5 secondes
  sleep 5
done
```

**Rendre le script exécutable et le lancer :**
```bash
chmod +x generate_metrics.sh
./generate_metrics.sh
```

Appuyez sur `Ctrl+C` pour arrêter le script.

---

## 📈 Étape 6 : Créer votre premier tableau de bord Grafana

Maintenant que nous avons des données, créons un magnifique tableau de bord !

### A. Créer un nouveau dashboard

1. Dans Grafana, cliquez sur **"+"** (en haut à droite) → **"Dashboard"**
2. Cliquez sur **"Add visualization"**
3. Sélectionnez la source de données **"InfluxDB-Monitoring"**

### B. Configuration du premier panel (CPU Usage)

**Dans l'éditeur de requête :**

1. Sélectionnez le **Query Language** : **"Flux"** (normalement déjà sélectionné)

2. Entrez cette requête Flux dans l'éditeur :

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r._field == "value")
```

**Explication de la requête :**
- `from(bucket: "metrics")` : Lit depuis le bucket "metrics"
- `range(start: -1h)` : Récupère les données de la dernière heure
- `filter(...)` : Filtre sur la mesure "cpu_usage" et le champ "value"

3. Cliquez sur **"Run query"** (bouton en haut à droite)

Vous devriez voir vos données apparaître dans le graphique ! 🎉

### C. Personnaliser le panel

**Dans le panneau de droite :**

1. **Panel Title** : `CPU Usage by Server`
2. **Visualization** : Choisissez **"Time series"** (graphique de lignes)
3. **Panel options** :
   - **Unit** : `percent (0-100)`
   - **Min** : `0`
   - **Max** : `100`

4. **Legend** : Cochez **"Show legend"**

5. Cliquez sur **"Apply"** en haut à droite

### D. Ajouter d'autres panels

Répétez le processus pour créer d'autres graphiques :

**Panel 2 : Memory Usage**
```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "memory_usage")
  |> filter(fn: (r) => r._field == "value")
```

**Panel 3 : Temperature**
```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> filter(fn: (r) => r._field == "value")
```

### E. Organiser le dashboard

1. Cliquez sur l'icône **"Dashboard settings"** (⚙️) en haut à droite
2. Donnez un nom : `Monitoring Dashboard`
3. **Save** le dashboard

Vous pouvez **déplacer** et **redimensionner** les panels en les faisant glisser.

---

## 🎨 Étape 7 : Types de visualisations disponibles

Grafana offre de nombreux types de graphiques :

| Type | Usage | Exemple |
|------|-------|---------|
| **Time series** | Évolution dans le temps | CPU, température, trafic |
| **Gauge** | Valeur actuelle avec seuils | Utilisation disque, batterie |
| **Stat** | Valeur unique grande | Nombre d'utilisateurs, total |
| **Bar chart** | Comparaison de valeurs | Comparaison entre serveurs |
| **Table** | Données tabulaires | Liste de logs, événements |
| **Heatmap** | Densité de données | Distribution de temps de réponse |

**Pour changer le type de visualisation :**
1. Éditez un panel
2. Dans la barre de droite, changez le **"Visualization"**
3. Testez différents types !

---

## 🎯 Étape 8 : Requêtes Flux avancées

### A. Agréger des données (moyenne)

Calculer la moyenne du CPU par intervalle de 1 minute :

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r._field == "value")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
```

### B. Filtrer par tag

Afficher uniquement les serveurs de la région "eu" :

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r.region == "eu")
  |> filter(fn: (r) => r._field == "value")
```

### C. Calculer des dérivées

Calculer le taux de changement :

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r._field == "value")
  |> derivative(unit: 1m, nonNegative: true)
```

### D. Combiner plusieurs mesures

```flux
cpu = from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r._field == "value")

mem = from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "memory_usage")
  |> filter(fn: (r) => r._field == "value")

union(tables: [cpu, mem])
```

---

## 🔔 Étape 9 : Configurer des alertes (optionnel)

Grafana peut vous alerter quand une métrique dépasse un seuil.

### A. Créer une alerte

1. Éditez un panel (ex: CPU Usage)
2. Allez dans l'onglet **"Alert"**
3. Cliquez sur **"Create alert rule from this panel"**

**Configuration de l'alerte :**
- **Condition** : `WHEN avg() OF query(A, 5m) IS ABOVE 80`
  - Signifie : Alerter si la moyenne du CPU dépasse 80% sur 5 minutes
- **Evaluate every** : `1m` (vérifier toutes les minutes)
- **For** : `5m` (alerter après 5 minutes de dépassement)

4. **Save** l'alerte

### B. Configurer les notifications (optionnel)

Pour recevoir des notifications :

1. Allez dans **"Alerting"** → **"Contact points"**
2. Cliquez sur **"Add contact point"**
3. Choisissez un type :
   - **Email** (nécessite configuration SMTP)
   - **Slack**
   - **Discord**
   - **Webhook** (pour notifications personnalisées)

---

## 📱 Étape 10 : Variables et dashboards dynamiques

Les **variables** permettent de créer des dashboards interactifs.

### A. Créer une variable

1. Ouvrez votre dashboard
2. Cliquez sur **"Dashboard settings"** (⚙️) → **"Variables"**
3. Cliquez sur **"Add variable"**

**Configuration :**
- **Name** : `server`
- **Type** : `Query`
- **Data source** : `InfluxDB-Monitoring`
- **Query** :

```flux
import "influxdata/influxdb/schema"

schema.tagValues(
  bucket: "metrics",
  tag: "host"
)
```

4. **Save** la variable

### B. Utiliser la variable dans une requête

Modifiez votre requête CPU :

```flux
from(bucket: "metrics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu_usage")
  |> filter(fn: (r) => r.host == "${server}")
  |> filter(fn: (r) => r._field == "value")
```

Maintenant, un menu déroulant apparaît en haut du dashboard pour choisir le serveur ! 🎉

---

## 🎨 Étape 11 : Personnalisation de l'apparence

### A. Thèmes

Grafana propose des thèmes clairs et sombres :

1. Cliquez sur votre profil (en bas à gauche)
2. **"Preferences"**
3. **"UI Theme"** : Choisissez **"Dark"** ou **"Light"**

### B. Couleurs personnalisées

Dans un panel, vous pouvez personnaliser les couleurs :

1. Éditez un panel
2. Allez dans **"Overrides"**
3. **"Add field override"**
4. Choisissez **"Color scheme"** et sélectionnez une palette

### C. Seuils (Thresholds)

Définir des seuils pour changer la couleur selon la valeur :

1. Dans un panel, allez dans **"Thresholds"**
2. Ajoutez des seuils :
   - `0-50` : Vert (OK)
   - `50-80` : Orange (Attention)
   - `80-100` : Rouge (Critique)

---

## 🛑 Étape 12 : Gestion de la stack

### A. Arrêter les services (sans supprimer)

```bash
docker-compose stop
```

### B. Redémarrer les services

```bash
docker-compose start
```

Vos dashboards et données sont préservés ! ✅

### C. Voir les logs

```bash
# Logs combinés
docker-compose logs -f

# Logs InfluxDB uniquement
docker-compose logs -f influxdb

# Logs Grafana uniquement
docker-compose logs -f grafana
```

### D. Suppression complète

**⚠️ ATTENTION : Supprime toutes les données et dashboards !**

```bash
docker-compose down -v
```

---

## 💾 Étape 13 : Sauvegarder vos dashboards

### A. Exporter un dashboard

1. Ouvrez un dashboard
2. Cliquez sur **"Dashboard settings"** (⚙️)
3. **"JSON Model"**
4. Copiez tout le JSON
5. Sauvegardez-le dans un fichier `.json`

### B. Importer un dashboard

1. Cliquez sur **"+"** → **"Import dashboard"**
2. Collez le JSON ou uploadez le fichier
3. Cliquez sur **"Load"** puis **"Import"**

### C. Dashboards communautaires

Grafana propose des milliers de dashboards gratuits :

1. Allez sur [https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/)
2. Cherchez "InfluxDB"
3. Notez l'**ID du dashboard** (ex: `1443`)
4. Dans Grafana : **"+"** → **"Import"** → Entrez l'ID

---

## 🔧 Étape 14 : Configuration avancée

### A. Fichier de configuration Grafana (grafana.ini)

Pour une configuration avancée, créez un fichier `grafana.ini` :

```ini
[server]
http_port = 3000

[security]
admin_user = admin
admin_password = admin

[analytics]
reporting_enabled = false

[log]
level = info
```

Montez-le dans le `docker-compose.yml` :

```yaml
grafana:
  volumes:
    - grafana_data:/var/lib/grafana
    - ./grafana.ini:/etc/grafana/grafana.ini
```

### B. Activer les plugins

Grafana supporte des plugins pour étendre ses fonctionnalités.

Dans le `docker-compose.yml`, ajoutez :

```yaml
grafana:
  environment:
    - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource
```

---

## ✅ Récapitulatif

**Ce que vous avez appris :**

✅ Déployer une stack InfluxDB + Grafana avec Docker Compose
✅ Connecter Grafana à InfluxDB avec le langage Flux
✅ Insérer des données de test dans InfluxDB
✅ Créer des dashboards avec multiples panels
✅ Utiliser différents types de visualisations
✅ Écrire des requêtes Flux avancées
✅ Configurer des variables pour des dashboards dynamiques
✅ Exporter/importer des dashboards
✅ Gérer la stack (arrêt, redémarrage, logs)

---

## 💡 Cas d'usage réels

| Secteur | Utilisation | Métriques typiques |
|---------|-------------|-------------------|
| **DevOps** | Monitoring infra | CPU, RAM, disque, réseau |
| **IoT** | Tableaux de bord capteurs | Température, humidité, pression |
| **Finance** | Analyse trading | Prix, volumes, indicateurs |
| **E-commerce** | Analytics business | Ventes, panier moyen, conversion |
| **Industrie** | Monitoring machines | Vibrations, température, production |

---

## 🐛 Dépannage

### Grafana ne se connecte pas à InfluxDB

**Erreur :** `Error reading InfluxDB`

**Solutions :**
1. Vérifiez que les deux conteneurs sont sur le même réseau
2. Utilisez `http://influxdb:8086` (nom du service, pas `localhost`)
3. Vérifiez le token et l'organisation
4. Consultez les logs : `docker-compose logs grafana`

### Les données n'apparaissent pas

**Solutions :**
1. Vérifiez la plage de temps (en haut à droite dans Grafana)
2. Testez la requête dans InfluxDB Data Explorer
3. Vérifiez que des données existent : `range(start: -24h)`

### Dashboard ne charge pas

**Solutions :**
1. Rechargez la page (`Ctrl+F5`)
2. Videz le cache du navigateur
3. Vérifiez les logs Grafana

---

## 🔗 Ressources utiles

- 📘 [Documentation Grafana](https://grafana.com/docs/)
- 📖 [Grafana Dashboards](https://grafana.com/grafana/dashboards/)
- 🎓 [Tutoriels Grafana](https://grafana.com/tutorials/)
- 🐙 [Image Docker Grafana](https://hub.docker.com/r/grafana/grafana)
- 📊 [Flux Query Language](https://docs.influxdata.com/flux/v0.x/)

---

## 🚀 Pour aller plus loin

Maintenant que vous maîtrisez la stack InfluxDB + Grafana, explorez :

- **9.1** [Configuration basique InfluxDB](01-config-basique-v2.md) - Approfondir InfluxDB
- **9.2** [Configuration avec IP fixe](02-config-ip-fixe.md) - Réseau stable
- **9.4** [Configuration avec Telegraf](04-config-telegraf.md) - Collecte automatique de métriques système
- **Cas pratique** [Stack de monitoring complète](../cas-pratiques/03-stack-elk.md) - ELK Stack

---

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Novembre 2025*
