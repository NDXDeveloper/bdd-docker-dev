# 9.2 Configuration InfluxDB avec IP fixe

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Par défaut, Docker attribue des **adresses IP dynamiques** aux conteneurs. Cela signifie que l'IP peut changer à chaque redémarrage, ce qui peut poser problème si :

- 🔗 Vous avez d'autres conteneurs qui doivent se connecter à InfluxDB via son IP
- 🛠️ Vous configurez des applications externes qui pointent vers une IP spécifique
- 📊 Vous souhaitez une configuration réseau stable et prévisible

Dans cette fiche, vous apprendrez à **assigner une adresse IP fixe** à votre conteneur InfluxDB.

---

## 🎯 Ce que vous allez apprendre

✅ Comprendre pourquoi utiliser une IP fixe
✅ Créer un réseau Docker personnalisé avec un sous-réseau défini
✅ Configurer InfluxDB avec une adresse IP statique
✅ Tester la connexion via l'IP fixe
✅ Connecter d'autres conteneurs à InfluxDB via cette IP

---

## 🔑 Concepts importants

### Pourquoi une IP fixe ?

**Sans IP fixe (comportement par défaut) :**
```
Démarrage 1 : InfluxDB → 172.17.0.3
Redémarrage : InfluxDB → 172.17.0.5  ❌ L'IP a changé !
```

**Avec IP fixe :**
```
Démarrage 1 : InfluxDB → 172.18.0.10
Redémarrage : InfluxDB → 172.18.0.10  ✅ L'IP reste la même !
```

### Réseau Docker personnalisé

Pour avoir une IP fixe, nous devons créer notre **propre réseau Docker** avec :
- Un **nom** (ex: `influxdb_net`)
- Un **sous-réseau** (ex: `172.18.0.0/16`)
- Une **plage d'IP disponibles** (ex: de `172.18.0.1` à `172.18.255.254`)

---

## 📦 Prérequis

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Avoir suivi la [fiche 9.1 - Configuration basique](01-config-basique-v2.md) (recommandé)

---

## 🚀 Étape 1 : Création du réseau Docker dédié

Avant de créer notre conteneur, nous devons d'abord créer un réseau Docker personnalisé.

### A. Créer le réseau

Ouvrez votre terminal et exécutez cette commande **une seule fois** :

```bash
docker network create --subnet=172.18.0.0/16 influxdb_net
```

**Explication de la commande :**
- `docker network create` : Crée un nouveau réseau
- `--subnet=172.18.0.0/16` : Définit la plage d'adresses IP disponibles
  - `172.18.0.0/16` signifie : de `172.18.0.1` à `172.18.255.254`
- `influxdb_net` : Le nom de notre réseau personnalisé

**Sortie attendue :**
```
f8e9a12b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f
```

Vous recevez un **ID unique** pour votre réseau.

### B. Vérifier que le réseau existe

```bash
docker network ls
```

**Sortie attendue :**
```
NETWORK ID     NAME            DRIVER    SCOPE
...
f8e9a12b3c4d   influxdb_net    bridge    local
```

Vous devriez voir votre réseau `influxdb_net` dans la liste.

### C. Inspecter le réseau (facultatif)

Pour voir les détails du réseau :

```bash
docker network inspect influxdb_net
```

Vous verrez la configuration complète du réseau, notamment le sous-réseau (`172.18.0.0/16`).

---

## 🚀 Étape 2 : Configuration avec IP fixe

### A. Créer le dossier du projet

```bash
# Créer et entrer dans le dossier
mkdir influxdb_ip_fixe
cd influxdb_ip_fixe
```

### B. Créer le fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  influxdb:
    # Image officielle InfluxDB v2.x
    image: influxdb:2.7
    container_name: influxdb_with_ip
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
      # Configuration initiale automatique
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_ORG=ma_organisation
      - DOCKER_INFLUXDB_INIT_BUCKET=mon_bucket
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      # ⚠️ CHANGEZ ce mot de passe !
      - DOCKER_INFLUXDB_INIT_PASSWORD=motdepasse_securise_123
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=mon_super_token_secret
    networks:
      influxdb_net:
        # 🎯 C'EST ICI QU'ON DÉFINIT L'IP FIXE
        ipv4_address: 172.18.0.10

volumes:
  influxdb_data:
  influxdb_config:

networks:
  # On déclare le réseau comme "externe" car on l'a créé manuellement
  influxdb_net:
    external: true
```

**🔑 Points importants :**

- **`networks:`** : Section pour configurer le réseau
- **`influxdb_net:`** : Référence au réseau créé à l'étape 1
- **`ipv4_address: 172.18.0.10`** : L'IP fixe que nous voulons
- **`external: true`** : Indique que le réseau existe déjà (créé à l'étape 1)

**💡 Pourquoi `172.18.0.10` ?**
- Les IP de `172.18.0.1` à `172.18.0.9` sont souvent réservées
- `172.18.0.10` est une IP "sûre" dans notre plage
- Vous pouvez choisir n'importe quelle IP entre `172.18.0.10` et `172.18.255.254`

---

## ▶️ Étape 3 : Lancement du conteneur

### A. Démarrer InfluxDB

Dans le dossier contenant votre `docker-compose.yml`, exécutez :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating volume "influxdb_ip_fixe_influxdb_data" with default driver
Creating volume "influxdb_ip_fixe_influxdb_config" with default driver
Creating influxdb_with_ip ... done
```

### B. Vérifier que le conteneur fonctionne

```bash
docker-compose ps
```

**Sortie attendue :**
```
      Name                    Command               State           Ports
---------------------------------------------------------------------------------
influxdb_with_ip   /entrypoint.sh influxd           Up      0.0.0.0:8086->8086/tcp
```

Le statut doit être `Up`.

---

## ✅ Étape 4 : Vérification de l'IP fixe

### A. Vérifier l'adresse IP du conteneur

```bash
docker inspect influxdb_with_ip | grep "IPAddress"
```

**Sortie attendue :**
```json
"IPAddress": "",
"IPAddress": "172.18.0.10",
```

🎉 **Parfait !** Votre conteneur a bien l'IP `172.18.0.10`.

**💡 Note :** La première ligne `"IPAddress": ""` est normale (réseau par défaut non utilisé).

### B. Méthode alternative (plus détaillée)

Pour voir toutes les informations réseau :

```bash
docker inspect influxdb_with_ip -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

**Sortie attendue :**
```
172.18.0.10
```

---

## 🌐 Étape 5 : Test de connexion via l'IP fixe

### A. Test depuis votre machine hôte

**Via navigateur web :**

Ouvrez votre navigateur et accédez à :

```
http://172.18.0.10:8086
```

Vous devriez voir l'interface web d'InfluxDB ! 🎉

**Via curl (ligne de commande) :**

```bash
curl http://172.18.0.10:8086/ping
```

**Sortie attendue :**
```
(Rien, mais le code de retour est 204 - c'est normal !)
```

Pour voir le code de retour :

```bash
curl -i http://172.18.0.10:8086/ping
```

**Sortie attendue :**
```
HTTP/1.1 204 No Content
...
```

Le code `204` signifie que le serveur répond correctement.

### B. Test depuis un autre conteneur Docker

Créons un conteneur temporaire pour tester la connexion :

```bash
# Lancer un conteneur Ubuntu sur le même réseau
docker run --rm -it --network influxdb_net ubuntu bash

# Une fois dans le conteneur :
apt-get update && apt-get install -y curl

# Tester la connexion
curl http://172.18.0.10:8086/ping

# Sortir du conteneur
exit
```

Si vous recevez une réponse, **la connexion via IP fixe fonctionne** ! ✅

---

## 🔗 Étape 6 : Connecter une application à InfluxDB via l'IP fixe

Imaginons que vous ayez une application (par exemple, Telegraf ou Grafana) dans un autre conteneur qui doit se connecter à InfluxDB.

### Exemple : Conteneur de test

Créez un fichier `docker-compose-app.yml` :

```yaml
version: '3.8'

services:
  mon_application:
    image: alpine:latest
    container_name: test_app
    networks:
      - influxdb_net
    command: >
      sh -c "
      apk add --no-cache curl &&
      echo 'Test de connexion à InfluxDB...' &&
      curl -i http://172.18.0.10:8086/ping &&
      echo 'Connexion réussie!' &&
      sleep infinity
      "

networks:
  influxdb_net:
    external: true
```

**Lancer l'application de test :**

```bash
docker-compose -f docker-compose-app.yml up -d
```

**Voir les logs :**

```bash
docker logs test_app
```

**Sortie attendue :**
```
Test de connexion à InfluxDB...
HTTP/1.1 204 No Content
...
Connexion réussie!
```

✅ Votre application peut communiquer avec InfluxDB via l'IP fixe !

**Nettoyage :**

```bash
docker-compose -f docker-compose-app.yml down
```

---

## 📊 Étape 7 : Configuration de l'URL dans les applications

Maintenant que votre InfluxDB a une IP fixe, voici comment configurer vos applications :

### A. Configuration Telegraf (collecteur de métriques)

Dans votre fichier `telegraf.conf` :

```toml
[[outputs.influxdb_v2]]
  ## URL de votre InfluxDB (utilisez l'IP fixe)
  urls = ["http://172.18.0.10:8086"]

  ## Token d'authentification
  token = "votre_token_admin"

  ## Organisation et bucket
  organization = "ma_organisation"
  bucket = "mon_bucket"
```

### B. Configuration Grafana (visualisation)

Dans Grafana, lors de l'ajout d'une source de données InfluxDB :

- **URL** : `http://172.18.0.10:8086`
- **Organization** : `ma_organisation`
- **Token** : `votre_token_admin`
- **Default Bucket** : `mon_bucket`

### C. Configuration depuis Python

```python
from influxdb_client import InfluxDBClient

# Connexion via l'IP fixe
client = InfluxDBClient(
    url="http://172.18.0.10:8086",
    token="votre_token_admin",
    org="ma_organisation"
)

# Utiliser le client...
```

---

## 🔄 Étape 8 : Avantages de l'IP fixe

| Avantage | Explication |
|----------|-------------|
| 🎯 **Prévisibilité** | L'IP ne change jamais, même après un redémarrage |
| 🔗 **Interconnexion** | Facile de connecter d'autres conteneurs |
| 📝 **Configuration simple** | Pas besoin de DNS ou de service discovery |
| 🐛 **Débogage** | Plus facile de tracer les connexions réseau |

---

## 🛑 Étape 9 : Gestion et suppression

### A. Arrêter InfluxDB (sans supprimer)

```bash
docker-compose stop
```

### B. Redémarrer InfluxDB

```bash
docker-compose start
```

L'IP reste `172.18.0.10` ! ✅

### C. Suppression complète

**⚠️ ATTENTION : Supprime les données !**

```bash
# Supprimer le conteneur et les volumes
docker-compose down -v

# Supprimer le réseau (si vous n'en avez plus besoin)
docker network rm influxdb_net
```

---

## 🆚 Comparaison : Avec et sans IP fixe

### Sans IP fixe (méthode basique)

```yaml
services:
  influxdb:
    image: influxdb:2.7
    # Docker attribue une IP dynamique aléatoire
```

**Connexion :**
- Depuis l'hôte : `http://localhost:8086` ✅
- Depuis un autre conteneur : `http://influxdb:8086` (via nom de service) ✅
- Via IP : `http://172.17.0.3:8086` ❌ Change à chaque redémarrage !

### Avec IP fixe (cette méthode)

```yaml
services:
  influxdb:
    image: influxdb:2.7
    networks:
      influxdb_net:
        ipv4_address: 172.18.0.10
```

**Connexion :**
- Depuis l'hôte : `http://localhost:8086` ✅
- Depuis un autre conteneur : `http://influxdb:8086` ✅
- Via IP : `http://172.18.0.10:8086` ✅ **Toujours la même !**

---

## 💡 Conseils pratiques

### Quelle IP choisir ?

| Plage IP | Usage recommandé |
|----------|------------------|
| `172.18.0.1` - `172.18.0.9` | Réservé (gateway, DNS) ❌ |
| `172.18.0.10` - `172.18.0.99` | Bases de données (InfluxDB, MariaDB...) ✅ |
| `172.18.0.100` - `172.18.0.199` | Applications web ✅ |
| `172.18.0.200` - `172.18.0.254` | Services divers ✅ |

### Plusieurs conteneurs sur le même réseau

Vous pouvez avoir plusieurs conteneurs avec des IP fixes différentes :

```yaml
services:
  influxdb:
    networks:
      influxdb_net:
        ipv4_address: 172.18.0.10

  grafana:
    networks:
      influxdb_net:
        ipv4_address: 172.18.0.20

  telegraf:
    networks:
      influxdb_net:
        ipv4_address: 172.18.0.30
```

### Éviter les conflits

⚠️ **Ne jamais utiliser la même IP pour deux conteneurs différents !**

Docker vous empêchera de démarrer le second conteneur si l'IP est déjà prise.

---

## 🐛 Dépannage

### Erreur : "network influxdb_net not found"

**Cause :** Le réseau n'a pas été créé.

**Solution :**
```bash
docker network create --subnet=172.18.0.0/16 influxdb_net
```

### Erreur : "address already in use"

**Cause :** Un autre conteneur utilise déjà l'IP `172.18.0.10`.

**Solution :**
1. Voir les conteneurs sur le réseau :
   ```bash
   docker network inspect influxdb_net
   ```
2. Choisir une autre IP (ex: `172.18.0.11`)

### Le conteneur ne démarre pas

**Vérifier les logs :**
```bash
docker-compose logs influxdb
```

---

## ✅ Récapitulatif

**Ce que vous avez appris :**

✅ Créer un réseau Docker personnalisé avec un sous-réseau défini
✅ Assigner une adresse IP fixe à InfluxDB
✅ Vérifier l'IP du conteneur avec `docker inspect`
✅ Tester la connexion via l'IP fixe
✅ Connecter d'autres conteneurs à InfluxDB via cette IP
✅ Configurer des applications pour utiliser l'IP fixe

---

## 💡 Commandes essentielles

| Action | Commande |
|--------|----------|
| Créer le réseau | `docker network create --subnet=172.18.0.0/16 influxdb_net` |
| Démarrer InfluxDB | `docker-compose up -d` |
| Vérifier l'IP | `docker inspect influxdb_with_ip \| grep IPAddress` |
| Tester la connexion | `curl http://172.18.0.10:8086/ping` |
| Supprimer le réseau | `docker network rm influxdb_net` |

---

## 🔗 Ressources utiles

- 📘 [Documentation Docker Networks](https://docs.docker.com/network/)
- 📖 [InfluxDB API Documentation](https://docs.influxdata.com/influxdb/v2.7/api/)
- 🐙 [Image Docker InfluxDB](https://hub.docker.com/_/influxdb)

---

## 🚀 Pour aller plus loin

Maintenant que vous maîtrisez l'IP fixe, explorez :

- **9.1** [Configuration basique](01-config-basique-v2.md) - Les bases d'InfluxDB
- **9.3** [InfluxDB avec Grafana](03-influxdb-grafana.md) - Stack complète de monitoring
- **9.4** [Configuration avec Telegraf](04-config-telegraf.md) - Collecte automatique de métriques

---

🔝 [Retour au Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Novembre 2025*
