# 9.1 Configuration basique InfluxDB v2.x avec Docker Compose

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**InfluxDB** est une base de données optimisée pour les **séries temporelles** (time-series). Elle est idéale pour stocker et analyser des données qui évoluent dans le temps, comme :

- 📊 **Métriques système** (CPU, RAM, disque)
- 🌡️ **Données IoT** (température, humidité, capteurs)
- 📈 **Métriques applicatives** (temps de réponse, nombre de requêtes)
- 💹 **Données financières** (cours de bourse, transactions)

**InfluxDB v2.x** apporte une interface web moderne, une gestion simplifiée et de nouvelles fonctionnalités par rapport à la v1.x.

---

## 🎯 Ce que vous allez apprendre

Dans cette fiche, vous allez :

✅ Comprendre les concepts de base d'InfluxDB v2.x
✅ Déployer InfluxDB avec Docker Compose
✅ Effectuer la configuration initiale via l'interface web
✅ Créer votre premier bucket (équivalent d'une "base de données")
✅ Insérer et interroger des données simples
✅ Gérer votre conteneur (arrêt, redémarrage, suppression)

---

## 🔑 Concepts de base d'InfluxDB v2.x

Avant de commencer, quelques termes importants :

| Terme | Explication | Équivalent SQL |
|-------|-------------|----------------|
| **Organization** | Espace de travail principal qui regroupe tout | "Schema" ou "Database" |
| **Bucket** | Conteneur de données avec rétention définie | "Table" |
| **Token** | Clé d'API pour accéder aux données | Mot de passe utilisateur |
| **Point** | Une mesure avec timestamp, tags et champs | "Ligne" dans une table |
| **Measurement** | Type de données (ex: "temperature", "cpu") | "Table" |
| **Tag** | Métadonnée indexée (ex: location="paris") | "Index" |
| **Field** | Valeur mesurée (ex: value=22.5) | "Colonne" |

**Exemple de point InfluxDB :**
```
temperature,location=paris,sensor=A value=22.5 1635789600000000000
   ↑            ↑              ↑         ↑              ↑
measurement   tags          field    field value   timestamp
```

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Un **navigateur web** (pour l'interface InfluxDB)
- ✅ Un **éditeur de texte** (VS Code, Notepad++, nano...)

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🚀 Étape 1 : Préparation des fichiers

### A. Créer le dossier du projet

Créez un nouveau dossier pour votre projet InfluxDB :

```bash
# Créer et entrer dans le dossier
mkdir influxdb_dev
cd influxdb_dev
```

### B. Créer le fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  influxdb:
    # Image officielle InfluxDB v2.x
    image: influxdb:2.7
    container_name: influxdb_local
    restart: unless-stopped
    ports:
      # Port de l'interface web et de l'API
      - "8086:8086"
    volumes:
      # Volume pour la persistance des données
      - influxdb_data:/var/lib/influxdb2
      # Volume pour la configuration
      - influxdb_config:/etc/influxdb2
    environment:
      # Mode setup automatique désactivé (on fera la config via l'UI)
      - DOCKER_INFLUXDB_INIT_MODE=setup
      # Nom de l'organisation (modifiable)
      - DOCKER_INFLUXDB_INIT_ORG=ma_organisation
      # Nom du bucket par défaut (modifiable)
      - DOCKER_INFLUXDB_INIT_BUCKET=mon_bucket
      # Nom d'utilisateur administrateur
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      # Mot de passe administrateur (CHANGEZ-LE !)
      - DOCKER_INFLUXDB_INIT_PASSWORD=motdepasse_securise_123
      # Token d'admin (facultatif, sera généré si non fourni)
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=mon_super_token_secret_a_changer

volumes:
  # Déclaration des volumes pour Docker
  influxdb_data:
  influxdb_config:
```

**💡 Points importants :**

- **Port 8086** : C'est le port par défaut d'InfluxDB pour l'interface web ET l'API
- **Volumes** : Permettent de conserver vos données même si le conteneur est supprimé
- **Variables d'environnement** : Configurent InfluxDB au premier démarrage
- **⚠️ CHANGEZ** le mot de passe et le token par des valeurs sécurisées !

---

## ▶️ Étape 2 : Lancement du conteneur

### A. Démarrer InfluxDB

Dans le dossier contenant votre `docker-compose.yml`, exécutez :

```bash
docker-compose up -d
```

**Explication de la commande :**
- `up` : Démarre les services
- `-d` : Mode "detached" (arrière-plan)

**Sortie attendue :**
```
Creating network "influxdb_dev_default" with the default driver
Creating volume "influxdb_dev_influxdb_data" with default driver
Creating volume "influxdb_dev_influxdb_config" with default driver
Creating influxdb_local ... done
```

### B. Vérifier que le conteneur fonctionne

```bash
docker-compose ps
```

**Sortie attendue :**
```
     Name                   Command               State           Ports
--------------------------------------------------------------------------------
influxdb_local   /entrypoint.sh influxd           Up      0.0.0.0:8086->8086/tcp
```

Le statut doit être `Up`.

### C. Consulter les logs (facultatif)

Pour voir ce qui se passe dans le conteneur :

```bash
docker-compose logs -f influxdb
```

Appuyez sur `Ctrl+C` pour quitter les logs.

---

## 🌐 Étape 3 : Configuration initiale via l'interface web

### A. Accéder à l'interface web

Ouvrez votre navigateur et allez à l'adresse :

```
http://localhost:8086
```

**🎉 Félicitations !** Vous devriez voir la page d'accueil d'InfluxDB.

### B. Connexion

Puisque nous avons configuré le mode `setup` automatique, **la configuration est déjà faite** avec les valeurs du `docker-compose.yml`.

**Connectez-vous avec :**
- **Username** : `admin`
- **Password** : `motdepasse_securise_123` (celui que vous avez défini)

### C. Interface de bienvenue

Une fois connecté, vous arrivez sur le **tableau de bord** principal d'InfluxDB. Vous verrez :

- 📊 **Data Explorer** : Pour visualiser et interroger vos données
- 📝 **Notebooks** : Pour créer des analyses interactives
- ⚙️ **Settings** : Configuration, tokens, buckets, etc.
- 📈 **Dashboards** : Créer des tableaux de bord personnalisés

---

## 🗄️ Étape 4 : Vérifier la configuration

### A. Voir l'organisation et le bucket créés

1. Cliquez sur l'icône **Settings** (⚙️) dans la barre latérale gauche
2. Vous verrez votre organisation : `ma_organisation`
3. Cliquez sur **Buckets** dans le menu
4. Vous devriez voir le bucket : `mon_bucket`

**Qu'est-ce qu'un bucket ?**
Un bucket est comme une "base de données" dans InfluxDB. Il stocke vos données et a une **politique de rétention** (combien de temps les données sont conservées).

### B. Voir le token d'API

Les tokens sont utilisés pour **authentifier** les requêtes API vers InfluxDB.

1. Dans **Settings** (⚙️), cliquez sur **API Tokens**
2. Vous verrez un token nommé `admin's Token` (créé automatiquement)
3. Cliquez dessus pour voir sa valeur complète
4. **⚠️ Gardez ce token secret !** Il donne un accès total à votre InfluxDB

**💡 Astuce :** Notez ce token quelque part (par exemple dans un fichier `.env`), vous en aurez besoin pour les requêtes API.

---

## 📝 Étape 5 : Insérer vos premières données

Nous allons maintenant insérer des données dans InfluxDB pour tester que tout fonctionne.

### A. Via l'interface web (Data Explorer)

1. Cliquez sur **Data Explorer** (📊) dans la barre latérale
2. En bas de la page, cliquez sur l'onglet **"Write Data"**
3. Vous pouvez soit :
   - Utiliser le **Line Protocol** (format texte)
   - Utiliser l'**interface graphique** (formulaire)

**Exemple avec Line Protocol :**

Collez ceci dans le champ de texte :

```
temperature,location=paris,sensor=A value=22.5
temperature,location=lyon,sensor=B value=25.0
temperature,location=paris,sensor=A value=23.0
```

- Sélectionnez votre **bucket** : `mon_bucket`
- Cliquez sur **"Write Data"**

**🎉 Bravo !** Vous venez d'insérer vos premières données.

### B. Via la ligne de commande (CLI)

Vous pouvez aussi insérer des données depuis le terminal du conteneur.

```bash
# Se connecter au conteneur
docker exec -it influxdb_local bash

# Insérer des données (remplacez <TOKEN> par votre token)
influx write \
  --bucket mon_bucket \
  --org ma_organisation \
  --token <VOTRE_TOKEN> \
  --precision s \
  "temperature,location=marseille,sensor=C value=28.5"

# Quitter le conteneur
exit
```

---

## 🔍 Étape 6 : Interroger vos données

Maintenant que nous avons des données, interrogeons-les !

### A. Via Data Explorer (interface graphique)

1. Dans **Data Explorer**, utilisez le **Query Builder** (constructeur de requête visuel) :
   - Sélectionnez votre **bucket** : `mon_bucket`
   - Sélectionnez **Measurement** : `temperature`
   - Sélectionnez **Fields** : `value`
   - Cliquez sur **"Submit"**

Vous verrez un **graphique** avec vos données de température !

### B. Via Flux (langage de requête d'InfluxDB)

InfluxDB v2.x utilise **Flux**, un langage de requête puissant.

Dans l'onglet **"Script Editor"** de Data Explorer, collez ceci :

```flux
from(bucket: "mon_bucket")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> filter(fn: (r) => r._field == "value")
```

**Explication :**
- `from(bucket: "mon_bucket")` : Lit depuis le bucket
- `range(start: -1h)` : Récupère les données de la dernière heure
- `filter(...)` : Filtre sur la mesure et le champ

Cliquez sur **"Submit"** pour voir les résultats.

---

## 🔐 Étape 7 : Créer un nouveau bucket

Créons un bucket séparé pour organiser nos données.

1. Allez dans **Settings** (⚙️) → **Buckets**
2. Cliquez sur **"Create Bucket"**
3. Remplissez :
   - **Name** : `test_metrics`
   - **Retention Policy** : `30d` (30 jours) ou laissez "Never" (jamais)
4. Cliquez sur **"Create"**

**🎉 Voilà !** Vous avez un nouveau bucket prêt à recevoir des données.

---

## 🛑 Étape 8 : Gestion du conteneur

### A. Arrêter InfluxDB (sans supprimer les données)

```bash
docker-compose stop
```

Les données sont conservées dans les volumes Docker.

### B. Redémarrer InfluxDB

```bash
docker-compose start
```

Vos données sont toujours là !

### C. Voir les logs en temps réel

```bash
docker-compose logs -f influxdb
```

### D. Redémarrer complètement

```bash
docker-compose restart
```

---

## 🧹 Étape 9 : Suppression complète (nettoyage)

**⚠️ ATTENTION : Cette action supprime TOUTES vos données !**

Si vous voulez tout supprimer (conteneur + volumes + données) :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer les volumes (SUPPRIME LES DONNÉES !)
docker volume rm influxdb_dev_influxdb_data influxdb_dev_influxdb_config

# Ou en une seule commande :
docker-compose down -v
```

---

## ✅ Récapitulatif

**Ce que vous avez appris :**

✅ Déployer InfluxDB v2.x avec Docker Compose
✅ Accéder à l'interface web (port 8086)
✅ Comprendre les concepts : organization, bucket, token
✅ Insérer des données (Line Protocol et CLI)
✅ Interroger des données (Query Builder et Flux)
✅ Créer un bucket personnalisé
✅ Gérer le conteneur (stop, start, logs, suppression)

---

## 💡 Commandes essentielles

| Action | Commande |
|--------|----------|
| Démarrer | `docker-compose up -d` |
| Arrêter | `docker-compose stop` |
| Redémarrer | `docker-compose restart` |
| Voir les logs | `docker-compose logs -f influxdb` |
| Supprimer tout | `docker-compose down -v` |
| Entrer dans le conteneur | `docker exec -it influxdb_local bash` |

---

## 🔗 Ressources utiles

- 📘 [Documentation officielle InfluxDB](https://docs.influxdata.com/influxdb/v2.7/)
- 📖 [Guide Flux Query Language](https://docs.influxdata.com/flux/v0.x/)
- 🐙 [Image Docker officielle](https://hub.docker.com/_/influxdb)

---

## 🚀 Pour aller plus loin

Maintenant que vous maîtrisez la configuration basique, vous pouvez explorer :

- **9.2** [Configuration avec IP fixe](02-config-ip-fixe.md)
- **9.3** [InfluxDB avec Grafana](03-influxdb-grafana.md) - Visualisation avancée
- **9.4** [Configuration avec Telegraf](04-config-telegraf.md) - Collecte de métriques

---

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Novembre 2025*
