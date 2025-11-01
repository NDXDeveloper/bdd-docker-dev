# 9. InfluxDB - Base de Données de Séries Temporelles

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

## 📋 Présentation d'InfluxDB

**InfluxDB** est une base de données open-source optimisée pour stocker et interroger des **données de séries temporelles** (time-series data). Elle est conçue pour gérer des volumes massifs de données horodatées avec des performances exceptionnelles.

**Développée par :** InfluxData
**Première version :** 2013
**Langage :** Go
**Licence :** MIT (InfluxDB v2.x open source)
**Type :** Base NoSQL orientée séries temporelles

---

## 🎯 Qu'est-ce qu'une série temporelle ?

Une **série temporelle** est une séquence de points de données indexés dans le temps.

**Exemples concrets :**
```
📊 Métriques système :
   Timestamp        CPU    RAM    Disque
   10:00:00        45%    78%    65%
   10:00:10        47%    79%    65%
   10:00:20        43%    77%    66%
   ...

🌡️ Données IoT :
   Timestamp        Température    Humidité    Capteur
   14:30:00        22.5°C         65%         A
   14:30:30        22.7°C         64%         A
   14:31:00        22.6°C         65%         A
   ...

💹 Données financières :
   Timestamp        Prix    Volume    Ticker
   09:30:00        150.25  1000      AAPL
   09:30:01        150.27  500       AAPL
   09:30:02        150.26  750       AAPL
   ...
```

**Caractéristiques communes :**
- ⏰ **Timestamp** : Chaque donnée a une date/heure précise
- 📈 **Évolution** : Les valeurs changent dans le temps
- 📊 **Volume** : Grande quantité de données générées
- 🔍 **Analyse** : Besoin de requêtes sur des plages de temps

---

## 🏆 Pourquoi choisir InfluxDB ?

### Avantages principaux

| Avantage | Explication | Exemple d'usage |
|----------|-------------|-----------------|
| ⚡ **Très rapide** | Écritures jusqu'à 1 million points/sec | Monitoring en temps réel |
| 🗜️ **Compression** | Compression automatique (jusqu'à 95%) | Économie d'espace disque |
| 🔍 **Requêtes puissantes** | Langage Flux moderne et expressif | Analyses complexes |
| 📉 **Downsampling** | Agrégation automatique des vieilles données | Optimisation du stockage |
| 🔄 **Rétention** | Suppression automatique selon l'âge | Gestion de la durée de vie |
| 📊 **Agrégations** | Fonctions statistiques intégrées | Moyennes, min, max, percentiles |
| 🌐 **HTTP API** | API REST complète | Intégration facile |
| 🎨 **Interface web** | UI moderne pour v2.x | Exploration des données |

### Comparaison avec les bases SQL traditionnelles

| Fonctionnalité | Base SQL (MySQL/PostgreSQL) | InfluxDB |
|----------------|----------------------------|----------|
| **Optimisation** | Requêtes générales | Séries temporelles |
| **Écriture** | Modérée | Très rapide (x10-100) |
| **Compression** | Standard | Excellente (x20) |
| **Rétention auto** | Non | Oui |
| **Downsampling** | Manuel | Automatique |
| **Agrégations temps** | Lentes | Optimisées |
| **Requêtes complexes** | SQL | Flux |

**💡 Règle d'or :** Utilisez InfluxDB si vos données ont un **timestamp** et que vous devez faire des **analyses temporelles**.

---

## 🎯 Cas d'usage typiques

### 1. 📊 Monitoring système et infrastructure

**Métriques collectées :**
- CPU, RAM, disque, réseau
- Temps de réponse applicatifs
- Nombre de requêtes
- Codes d'erreur

**Outils associés :**
- **Telegraf** : Collecte automatique de métriques
- **Grafana** : Visualisation en dashboards
- **Kapacitor** : Alertes en temps réel

**Exemples :**
- Surveiller la santé de 1000 serveurs
- Détecter les pics de charge
- Alerter sur utilisation CPU > 90%

### 2. 🌡️ Internet des Objets (IoT)

**Données capteurs :**
- Température, humidité, pression
- GPS / Localisation
- Consommation électrique
- État des machines

**Exemples :**
- Smart home : suivi de température par pièce
- Agriculture : monitoring des serres
- Industrie : prédiction de pannes machines

### 3. 💹 Données financières

**Métriques financières :**
- Cours de bourse
- Volumes de transactions
- Carnets d'ordres
- Indicateurs techniques (RSI, MACD...)

**Exemples :**
- Trading algorithmique
- Analyse de volatilité
- Backtesting de stratégies

### 4. 📱 Analytics d'applications

**Événements utilisateurs :**
- Clics, vues de pages
- Temps de session
- Taux de conversion
- Performances frontend

**Exemples :**
- Tableau de bord analytics temps réel
- A/B testing
- Monitoring d'expérience utilisateur

### 5. 🚗 Télémétrie et véhicules connectés

**Données véhicule :**
- Vitesse, accélération
- Consommation carburant
- Diagnostics moteur
- Position GPS

**Exemples :**
- Flottes de véhicules
- Voitures autonomes
- Optimisation de trajets

---

## 🔑 Concepts clés d'InfluxDB v2.x

### Structure des données

```
Organization (Organisation)
    └── Bucket (Conteneur de données)
            └── Measurement (Type de données)
                    ├── Tags (Métadonnées indexées)
                    ├── Fields (Valeurs mesurées)
                    └── Timestamp (Horodatage)
```

### Terminologie

| Terme | Définition | Équivalent SQL | Exemple |
|-------|------------|----------------|---------|
| **Organization** | Espace de travail principal | Database | "monitoring_org" |
| **Bucket** | Conteneur de données avec rétention | Table | "server_metrics" |
| **Measurement** | Type de mesure | Table | "cpu", "temperature" |
| **Tag** | Métadonnée indexée (catégorie) | Index | location="paris" |
| **Field** | Valeur mesurée | Column | value=45.5 |
| **Timestamp** | Horodatage (nanoseconde) | Timestamp | 1635789600000000000 |
| **Point** | Une ligne de données complète | Row | Combinaison de tout |
| **Token** | Clé d'authentification API | Password | "my_secret_token" |

### Exemple de point de données

**Format Line Protocol :**
```
temperature,location=paris,sensor=A value=22.5,humidity=65 1635789600000000000
     ↑           ↑              ↑         ↑         ↑            ↑
Measurement    Tags          Tag       Fields    Field      Timestamp
```

**Décomposition :**
- **Measurement** : `temperature` (type de données)
- **Tags** : `location=paris`, `sensor=A` (métadonnées indexées)
- **Fields** : `value=22.5`, `humidity=65` (valeurs mesurées)
- **Timestamp** : `1635789600000000000` (en nanosecondes)

**📝 Règles importantes :**
- Les **tags** sont indexés → rapides pour filtrer
- Les **fields** ne sont PAS indexés → rapides pour agréger
- Choisissez bien : informations de filtrage = tags, valeurs numériques = fields

---

## 🆚 InfluxDB v1.x vs v2.x

Ce guide se concentre sur **InfluxDB v2.x** (la version moderne).

### Différences principales

| Fonctionnalité | InfluxDB v1.x | InfluxDB v2.x |
|----------------|---------------|---------------|
| **Langage de requête** | InfluxQL (SQL-like) | Flux (fonctionnel) |
| **Organisation** | Databases | Organizations + Buckets |
| **Interface web** | Basique | Moderne et complète |
| **Authentification** | Utilisateurs/passwords | Tokens API |
| **Configuration** | Fichier .conf | Variables d'env + UI |
| **APIs** | HTTP simples | API REST v2 unifiée |
| **Tâches** | Continuous Queries | Tasks (Flux) |
| **Compatibilité** | - | API v1 rétro-compatible |

**💡 Recommandation :** Pour un nouveau projet, utilisez **InfluxDB v2.x**.

---

## 🔧 Langage de requête : Flux

**Flux** est le langage de requête moderne d'InfluxDB v2.x.

### Philosophie de Flux

Flux est un langage **fonctionnel** où les données passent par une série de **transformations** :

```
Données → Filtrage → Agrégation → Transformation → Résultat
```

### Exemple simple

**Objectif :** Récupérer la moyenne du CPU sur la dernière heure

**Requête Flux :**
```flux
from(bucket: "telegraf_metrics")          // Lire depuis le bucket
  |> range(start: -1h)                    // Dernière heure
  |> filter(fn: (r) => r._measurement == "cpu")    // Filtrer sur "cpu"
  |> filter(fn: (r) => r._field == "usage_system") // Filtrer sur le champ
  |> aggregateWindow(every: 1m, fn: mean) // Moyenne par minute
```

**Explication ligne par ligne :**
1. `from(bucket: "...")` : Source de données
2. `range(start: -1h)` : Plage de temps (dernière heure)
3. `filter(...)` : Filtrer les lignes (comme WHERE en SQL)
4. `aggregateWindow(...)` : Agréger par fenêtre de temps

### Opérations courantes en Flux

| Opération | Description | Équivalent SQL |
|-----------|-------------|----------------|
| `from()` | Source de données | FROM |
| `range()` | Plage de temps | WHERE timestamp BETWEEN |
| `filter()` | Filtrage | WHERE |
| `group()` | Groupement | GROUP BY |
| `aggregateWindow()` | Agrégation temporelle | GROUP BY time |
| `map()` | Transformation | SELECT (calculs) |
| `join()` | Jointure | JOIN |
| `sort()` | Tri | ORDER BY |
| `limit()` | Limiter résultats | LIMIT |

---

## 🏗️ Architecture typique d'une stack InfluxDB

### Stack de monitoring complète

```
┌──────────────────────────────────────────────────┐
│                   UTILISATEUR                    │
│              (Navigateur Web)                    │
└────────────────────┬─────────────────────────────┘
                     │
                     ↓
         ┌───────────────────────┐
         │      GRAFANA          │  ← Visualisation
         │    (Port 3000)        │     (Dashboards)
         └───────────┬───────────┘
                     │ Requêtes Flux
                     ↓
         ┌───────────────────────┐
         │      INFLUXDB         │  ← Stockage
         │    (Port 8086)        │     (Time-series DB)
         └───────────┬───────────┘
                     │ Écriture
                     ↓
         ┌───────────────────────┐
         │      TELEGRAF         │  ← Collecte
         │       (Agent)         │     (Métriques)
         └───────────┬───────────┘
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
   ┌────────┐  ┌────────┐  ┌────────┐
   │Serveur │  │Serveur │  │Serveur │  ← Sources
   │   1    │  │   2    │  │   3    │
   └────────┘  └────────┘  └────────┘
```

**Flux de données :**
1. **Telegraf** (collecteur) récupère les métriques des serveurs/applications
2. **InfluxDB** (base) stocke les données de séries temporelles
3. **Grafana** (visualisation) affiche les dashboards en temps réel
4. **Utilisateur** consulte les graphiques via son navigateur

---

## 📚 Contenu de cette section

Cette section du guide couvre **4 fiches pratiques** pour maîtriser InfluxDB avec Docker :

### 9.1 [Configuration basique v2.x avec docker-compose](01-config-basique-v2.md)
**Niveau :** Débutant
**Durée :** 15-20 minutes

**Ce que vous allez apprendre :**
- ✅ Déployer InfluxDB v2.x en quelques minutes
- ✅ Comprendre les concepts de base (organization, bucket, token)
- ✅ Utiliser l'interface web moderne
- ✅ Insérer vos premières données
- ✅ Interroger avec le langage Flux
- ✅ Créer et gérer des buckets

### 9.2 [Configuration avec IP fixe](02-config-ip-fixe.md)
**Niveau :** Intermédiaire
**Durée :** 10-15 minutes

**Ce que vous allez apprendre :**
- ✅ Créer un réseau Docker personnalisé
- ✅ Assigner une adresse IP statique à InfluxDB
- ✅ Tester la connexion via l'IP fixe
- ✅ Connecter d'autres conteneurs facilement
- ✅ Configurer des applications pour utiliser l'IP fixe

### 9.3 [InfluxDB avec Grafana](03-influxdb-grafana.md)
**Niveau :** Intermédiaire
**Durée :** 30-40 minutes

**Ce que vous allez apprendre :**
- ✅ Déployer une stack InfluxDB + Grafana
- ✅ Connecter Grafana à InfluxDB
- ✅ Créer des tableaux de bord professionnels
- ✅ Utiliser différents types de graphiques
- ✅ Configurer des alertes
- ✅ Créer des dashboards dynamiques avec variables
- ✅ Importer des dashboards communautaires

### 9.4 [Configuration avec Telegraf](04-config-telegraf.md)
**Niveau :** Avancé
**Durée :** 45-60 minutes

**Ce que vous allez apprendre :**
- ✅ Déployer une stack complète (Telegraf + InfluxDB + Grafana)
- ✅ Configurer Telegraf pour la collecte automatique
- ✅ Monitorer le système (CPU, RAM, disque, réseau)
- ✅ Monitorer les conteneurs Docker
- ✅ Ajouter des plugins personnalisés
- ✅ Visualiser les métriques dans Grafana
- ✅ Optimiser les performances et la sécurité

---

## 🚀 Par où commencer ?

### Si vous êtes débutant avec InfluxDB

```
1. 📖 Lisez cette introduction (vous y êtes !)
2. ⚡ Suivez la fiche 9.1 - Configuration basique
3. 🌐 (Optionnel) Fiche 9.2 - IP fixe
4. 📊 Suivez la fiche 9.3 - InfluxDB + Grafana
5. 🔧 Suivez la fiche 9.4 - Telegraf (pour automatiser)
```

### Si vous connaissez déjà InfluxDB v1.x

```
1. 📖 Lisez la section "InfluxDB v1.x vs v2.x" ci-dessus
2. ⚡ Suivez la fiche 9.1 pour voir les changements
3. 🔧 Passez directement à la fiche 9.4 pour Telegraf
```

### Si vous voulez un monitoring complet immédiatement

```
→ Allez directement à la fiche 9.4 - Configuration avec Telegraf
  (elle inclut InfluxDB + Grafana + collecte automatique)
```

---

## 💡 Prérequis techniques

Avant de commencer les fiches pratiques, assurez-vous d'avoir :

### Logiciels requis
- ✅ **Docker** installé (version 20.10+)
  - [Installation Docker](https://docs.docker.com/get-docker/)
- ✅ **Docker Compose** installé (version 2.0+)
  - Généralement inclus avec Docker Desktop
- ✅ Un **navigateur web** moderne (Chrome, Firefox, Edge...)
- ✅ Un **éditeur de texte** (VS Code, Notepad++, nano...)

### Connaissances recommandées
- 📝 Notions de base Docker (voir [00-introduction](../00-introduction/))
- 💻 Utilisation du terminal/ligne de commande
- 🌐 Notions de base HTTP/API REST (optionnel)

### Ressources système minimales
- **CPU** : 2 cœurs
- **RAM** : 4 GB
- **Disque** : 10 GB disponibles
- **OS** : Windows 10+, macOS 10.14+, ou Linux

---

## 🎓 Ressources complémentaires

### Documentation officielle
- 📘 [Documentation InfluxDB v2.x](https://docs.influxdata.com/influxdb/v2.7/)
- 📖 [Flux Query Language](https://docs.influxdata.com/flux/v0.x/)
- 🔧 [API Reference](https://docs.influxdata.com/influxdb/v2.7/api/)
- 🎥 [InfluxData YouTube](https://www.youtube.com/c/InfluxData)

### Communauté
- 💬 [InfluxData Community Forums](https://community.influxdata.com/)
- 🐙 [InfluxDB GitHub](https://github.com/influxdata/influxdb)
- 📚 [InfluxDB University](https://university.influxdata.com/) (cours gratuits)

### Images Docker officielles
- 🐳 [InfluxDB Docker Hub](https://hub.docker.com/_/influxdb)
- 🐳 [Telegraf Docker Hub](https://hub.docker.com/_/telegraf)
- 🐳 [Grafana Docker Hub](https://hub.docker.com/r/grafana/grafana)

---

## ❓ FAQ - Questions fréquentes

### InfluxDB est-il gratuit ?

**Oui**, InfluxDB v2.x Open Source est **gratuit** et sous licence MIT. Il existe aussi :
- **InfluxDB Cloud** : version hébergée (payante après free tier)
- **InfluxDB Enterprise** : version avec fonctionnalités avancées (payante)

Pour le développement et la plupart des cas d'usage, la version open source suffit largement.

### InfluxDB remplace-t-il une base SQL ?

**Non**, InfluxDB est **complémentaire** aux bases SQL :
- ✅ Utilisez InfluxDB pour : données temporelles, métriques, IoT, monitoring
- ✅ Utilisez PostgreSQL/MySQL pour : données relationnelles, utilisateurs, produits, commandes

**Exemple d'architecture mixte :**
- PostgreSQL : Stocke les utilisateurs, produits, commandes
- InfluxDB : Stocke les métriques de performance, logs d'accès

### Quelle version choisir : v1.x ou v2.x ?

**Choisissez v2.x** pour tout nouveau projet. La v1.x est en maintenance.

**v2.x apporte :**
- Interface web moderne
- Langage Flux plus puissant
- Meilleure gestion des utilisateurs
- API unifiée
- Meilleures performances

### InfluxDB peut-il gérer de gros volumes ?

**Oui**, InfluxDB est conçu pour ça :
- ⚡ **Écritures** : Jusqu'à 1 million de points/seconde (sur serveur correct)
- 💾 **Stockage** : Plusieurs téraoctets de données
- 🗜️ **Compression** : Réduction jusqu'à 95% de l'espace

**Exemple réel :** Une entreprise monitore 10 000 serveurs avec 100 métriques chacun toutes les 10 secondes = 6 millions de points/minute.

### Telegraf est-il obligatoire ?

**Non**, mais fortement recommandé pour la collecte automatique.

**Alternatives :**
- Écrire manuellement via l'API InfluxDB (scripts Python/Node/...)
- Utiliser des bibliothèques client (influxdb-client-python, etc.)
- D'autres collecteurs (Prometheus exporters + conversion)

Telegraf simplifie énormément la vie avec ses 300+ plugins prêts à l'emploi.

---

## 🎯 Objectifs pédagogiques de cette section

À la fin de cette section InfluxDB, vous serez capable de :

✅ **Comprendre** les concepts de séries temporelles et cas d'usage
✅ **Déployer** InfluxDB v2.x avec Docker en quelques minutes
✅ **Insérer** des données via Line Protocol ou API
✅ **Interroger** avec le langage Flux
✅ **Visualiser** dans Grafana avec dashboards professionnels
✅ **Automatiser** la collecte avec Telegraf
✅ **Monitorer** votre infrastructure (CPU, RAM, Docker...)
✅ **Configurer** des alertes en temps réel
✅ **Optimiser** performances et rétention des données
✅ **Déployer** en production avec sécurité

---

## 🚀 Commençons !

Vous êtes maintenant prêt à découvrir InfluxDB en pratique.

**👉 Passez à la première fiche :** [9.1 - Configuration basique v2.x avec docker-compose](01-config-basique-v2.md)

Ou consultez directement une fiche selon vos besoins :
- 🌐 [9.2 - Configuration avec IP fixe](02-config-ip-fixe.md)
- 📊 [9.3 - InfluxDB avec Grafana](03-influxdb-grafana.md)
- 🔧 [9.4 - Configuration avec Telegraf](04-config-telegraf.md)

**Bon apprentissage ! 🎓**

---

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Novembre 2025*
