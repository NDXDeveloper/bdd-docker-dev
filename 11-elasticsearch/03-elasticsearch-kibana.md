# 11.3 Elasticsearch avec Kibana

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**Kibana** est l'interface graphique officielle d'Elasticsearch. C'est un outil puissant qui vous permet de visualiser et d'explorer vos données Elasticsearch sans écrire de code.

### 🎨 Qu'est-ce que Kibana ?

Kibana offre :

- 🔍 **Interface de recherche** : Explorez vos données visuellement
- 📊 **Visualisations** : Graphiques, cartes, tableaux de bord
- 🗺️ **Découverte** : Parcourez vos index facilement
- ⚙️ **Dev Tools** : Console pour tester des requêtes Elasticsearch
- 📈 **Dashboards** : Tableaux de bord personnalisables

### 🎯 Cas d'usage typiques

- 📊 **Analyse de logs** : Visualiser les logs d'application
- 🔎 **Exploration de données** : Interface intuitive pour les recherches
- 📈 **Métriques** : Suivre des KPIs en temps réel
- 🛠️ **Développement** : Tester des requêtes Elasticsearch

### 📦 Ce que vous allez apprendre

Dans cette fiche, nous allons déployer **Elasticsearch + Kibana** ensemble, les configurer pour qu'ils communiquent, et accéder à l'interface web de Kibana.

---

## 🚀 Configuration : Elasticsearch + Kibana

### Étape 1 : Prérequis

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Au moins **6 GB de RAM** disponibles (4 GB pour Elasticsearch + 2 GB pour Kibana)

> ⚠️ **Important** : La combinaison Elasticsearch + Kibana nécessite plus de mémoire qu'Elasticsearch seul.

### Étape 2 : Créer le fichier `docker-compose.yml`

Créez un nouveau dossier (par exemple `elasticsearch_kibana`) et placez-y ce fichier :

```yaml
version: '3.8'

services:
  # ===== SERVICE ELASTICSEARCH =====
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch_dev
    restart: unless-stopped

    environment:
      # Nom du nœud
      - node.name=es-node01

      # Mode découverte : nœud unique
      - discovery.type=single-node

      # Désactive la sécurité (développement uniquement)
      - xpack.security.enabled=false

      # Mémoire : 2 GB min, 3 GB max
      - "ES_JAVA_OPTS=-Xms2g -Xmx3g"

      # Interface d'écoute
      - network.host=0.0.0.0

    ports:
      # API REST Elasticsearch
      - "9200:9200"

    volumes:
      # Données Elasticsearch
      - elasticsearch_data:/usr/share/elasticsearch/data

    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

    networks:
      - elastic

    # Vérification de santé
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # ===== SERVICE KIBANA =====
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana_dev
    restart: unless-stopped

    environment:
      # URL d'Elasticsearch (nom du service Docker)
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

      # Désactive la sécurité (développement uniquement)
      - xpack.security.enabled=false

      # Interface d'écoute (0.0.0.0 pour accès externe)
      - SERVER_HOST=0.0.0.0

      # Nom affiché dans Kibana
      - SERVER_NAME=kibana-dev

    ports:
      # Interface web Kibana
      - "5601:5601"

    networks:
      - elastic

    # Attendre qu'Elasticsearch soit prêt
    depends_on:
      elasticsearch:
        condition: service_healthy

    # Vérification de santé
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5601/api/status || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

# Volumes
volumes:
  elasticsearch_data:
    driver: local

# Réseau partagé
networks:
  elastic:
    driver: bridge
```

### 📝 Explication des paramètres clés

#### Configuration Elasticsearch

| Paramètre | Explication |
|-----------|-------------|
| `xpack.security.enabled=false` | Désactive l'authentification (dev seulement) |
| `ES_JAVA_OPTS=-Xms2g -Xmx3g` | Alloue 2-3 GB de RAM |
| `healthcheck` | Docker vérifie qu'Elasticsearch est prêt |

#### Configuration Kibana

| Paramètre | Explication |
|-----------|-------------|
| `ELASTICSEARCH_HOSTS` | URL pour se connecter à Elasticsearch |
| `SERVER_HOST=0.0.0.0` | Kibana écoute sur toutes les interfaces |
| `5601:5601` | Port de l'interface web |
| `depends_on` | Kibana attend qu'Elasticsearch soit prêt |

> 💡 **Note** : `http://elasticsearch:9200` utilise le nom du service Docker. Docker Compose crée automatiquement un DNS pour résoudre ce nom.

---

## ▶️ Étape 3 : Configuration Système (Linux uniquement)

Si vous êtes sur **Linux** :

```bash
# Vérifier la valeur actuelle
sysctl vm.max_map_count

# Si < 262144, augmentez-la
sudo sysctl -w vm.max_map_count=262144

# Rendre le changement permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

---

## ▶️ Étape 4 : Lancer la Stack EK (Elasticsearch + Kibana)

Dans le dossier contenant votre `docker-compose.yml` :

```bash
# Démarrer les deux services
docker-compose up -d

# Voir les logs des deux services
docker-compose logs -f
```

### Ce qui se passe :

1. ⏳ **Elasticsearch démarre** (30-60 secondes)
2. ✅ **Healthcheck** : Docker vérifie qu'Elasticsearch est prêt
3. ⏳ **Kibana démarre** et se connecte à Elasticsearch (30-60 secondes)
4. ✅ **Kibana est prêt** : Interface accessible

⏱️ **Temps total** : environ 1-2 minutes.

### Logs à surveiller

**Elasticsearch** :

```
[es-node01] started
```

**Kibana** :

```
[info][server][Kibana][http] http server running at http://0.0.0.0:5601
[info][status] Kibana is now available
```

---

## ✅ Étape 5 : Accéder à Kibana

### Ouvrir l'interface web

Dans votre navigateur, ouvrez :

```
http://localhost:5601
```

🎉 **Vous devriez voir la page d'accueil de Kibana !**

### Page d'accueil Kibana

Vous verrez plusieurs options :

- 🏠 **Home** : Page d'accueil
- 🔍 **Discover** : Explorer vos données
- 📊 **Visualize** : Créer des visualisations
- 🗺️ **Dashboard** : Tableaux de bord
- ⚙️ **Dev Tools** : Console pour requêtes Elasticsearch

---

## 🛠️ Utilisation de Kibana : Dev Tools

Les **Dev Tools** sont l'endroit idéal pour tester des requêtes Elasticsearch.

### Accéder à la Console

1. Cliquez sur le menu **☰** (hamburger) en haut à gauche
2. Sous **Management**, cliquez sur **Dev Tools**
3. Ou allez directement à : `http://localhost:5601/app/dev_tools#/console`

### Première requête : Vérifier le cluster

Dans la console, tapez :

```json
GET /
```

Cliquez sur le **▶️ bouton Play** ou appuyez sur **Ctrl+Entrée**.

**Résultat attendu** :

```json
{
  "name" : "es-node01",
  "cluster_name" : "docker-cluster",
  "version" : {
    "number" : "8.11.0",
    ...
  },
  "tagline" : "You Know, for Search"
}
```

### Créer un index et ajouter des documents

```json
# Créer un index
PUT /produits
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}

# Ajouter un document
POST /produits/_doc
{
  "nom": "Ordinateur portable",
  "prix": 899.99,
  "categorie": "Informatique",
  "stock": 15
}

# Ajouter un autre document
POST /produits/_doc
{
  "nom": "Souris sans fil",
  "prix": 24.99,
  "categorie": "Accessoires",
  "stock": 150
}

# Rechercher dans l'index
GET /produits/_search
{
  "query": {
    "match_all": {}
  }
}
```

💡 **Astuce** : Kibana colore la syntaxe et autocomplète vos requêtes !

---

## 🔍 Utilisation : Discover (Explorer les données)

### Accéder à Discover

1. Cliquez sur le menu **☰**
2. Sous **Analytics**, cliquez sur **Discover**
3. Ou allez à : `http://localhost:5601/app/discover`

### Créer un Data View (Vue de données)

Avant de voir vos données, vous devez créer un **Data View** :

1. Kibana vous proposera automatiquement de créer un Data View
2. Cliquez sur **Create data view**
3. **Name** : `produits`
4. **Index pattern** : `produits` (tapez le nom de votre index)
5. Cliquez sur **Save data view to Kibana**

### Explorer vos données

Vous verrez maintenant :

- 📊 **Timeline** : Distribution temporelle
- 📋 **Documents** : Liste de vos documents
- 🔍 **Search bar** : Barre de recherche KQL (Kibana Query Language)

**Exemples de recherches** :

```
categorie: "Informatique"
prix > 50
nom: "Ordinateur"
```

---

## 📊 Créer une Visualisation Simple

### Étape 1 : Accéder à Visualize

1. Menu **☰** → **Analytics** → **Visualize Library**
2. Cliquez sur **Create visualization**

### Étape 2 : Choisir un type

- **Bar** : Graphique en barres
- **Line** : Graphique en lignes
- **Pie** : Camembert
- **Metric** : Métrique unique
- **Table** : Tableau

Choisissons **Pie** pour l'exemple.

### Étape 3 : Configurer

1. **Data view** : Sélectionnez `produits`
2. **Slice by** : Cliquez sur **Add field**
3. Sélectionnez `categorie.keyword`
4. Cliquez sur **Save and return**

Vous avez créé votre première visualisation ! 🎉

---

## 🔧 Commandes Utiles

### Gestion des services

```bash
# Voir l'état des services
docker-compose ps

# Logs en temps réel (tous les services)
docker-compose logs -f

# Logs d'Elasticsearch uniquement
docker-compose logs -f elasticsearch

# Logs de Kibana uniquement
docker-compose logs -f kibana

# Redémarrer un service
docker-compose restart kibana

# Arrêter sans supprimer
docker-compose stop

# Redémarrer tout
docker-compose start
```

### Vérifications via curl

```bash
# Vérifier Elasticsearch
curl http://localhost:9200

# Vérifier la santé du cluster
curl http://localhost:9200/_cluster/health?pretty

# Vérifier le statut de Kibana
curl http://localhost:5601/api/status

# Lister les index
curl http://localhost:9200/_cat/indices?v
```

---

## 🌐 Accès depuis le Réseau Local

Pour accéder à Kibana depuis un autre ordinateur de votre réseau local :

### Étape 1 : Trouver l'IP de votre machine hôte

```bash
# Windows
ipconfig

# Linux / macOS
ip addr  # ou ifconfig
```

Exemple d'IP : `192.168.1.50`

### Étape 2 : Ouvrir le pare-feu

**Windows** : Autorisez le port **5601** dans le pare-feu Windows

**Linux (avec ufw)** :

```bash
sudo ufw allow 5601/tcp
```

### Étape 3 : Accéder depuis un autre PC

Sur l'autre ordinateur, ouvrez :

```
http://192.168.1.50:5601
```

---

## 🛑 Suppression Complète

Pour tout supprimer :

```bash
# 1. Arrêter et supprimer les conteneurs
docker-compose down

# 2. Supprimer le volume des données (⚠️ IRRÉVERSIBLE)
docker volume rm <nom_du_dossier>_elasticsearch_data

# Pour trouver le nom exact :
docker volume ls | grep elasticsearch
```

---

## 🐛 Problèmes Courants

### 1. Kibana affiche "Kibana server is not ready yet"

**Cause** : Kibana ne peut pas se connecter à Elasticsearch.

**Solutions** :

```bash
# 1. Vérifier qu'Elasticsearch tourne
docker-compose ps

# 2. Vérifier les logs d'Elasticsearch
docker-compose logs elasticsearch

# 3. Vérifier les logs de Kibana
docker-compose logs kibana

# 4. Redémarrer Kibana
docker-compose restart kibana
```

### 2. Page blanche ou erreur 502

**Cause** : Kibana n'est pas encore complètement démarré.

**Solution** : Patientez 1-2 minutes, puis rafraîchissez la page.

### 3. "max virtual memory areas too low" (Linux)

**Solution** :

```bash
sudo sysctl -w vm.max_map_count=262144
```

### 4. Manque de mémoire

**Symptôme** : Les conteneurs redémarrent en boucle.

**Solution** : Réduire la mémoire allouée dans `docker-compose.yml` :

```yaml
# Pour Elasticsearch
- "ES_JAVA_OPTS=-Xms1g -Xmx2g"
```

### 5. Impossible de créer un Data View

**Cause** : Aucun index n'existe encore.

**Solution** : Créez d'abord un index avec des données (voir section Dev Tools).

---

## 📊 Tableau Récapitulatif des Ports

| Service | Port | Accès | Usage |
|---------|------|-------|-------|
| Elasticsearch | 9200 | `http://localhost:9200` | API REST |
| Kibana | 5601 | `http://localhost:5601` | Interface web |

---

## 🔄 Architecture de Communication

```
┌─────────────────┐
│   Navigateur    │
│  localhost:5601 │
└────────┬────────┘
         │ HTTP
         ▼
┌─────────────────┐
│     Kibana      │
│   Port 5601     │
└────────┬────────┘
         │ HTTP
         ▼
┌─────────────────┐
│ Elasticsearch   │
│   Port 9200     │
└─────────────────┘
```

---

## 💡 Bonnes Pratiques

### Pour le développement

✅ **Désactivez la sécurité** : Simplifie les tests
✅ **Utilisez des noms explicites** : `elasticsearch`, `kibana`
✅ **Surveillez la RAM** : 6 GB minimum recommandés
✅ **Consultez les logs** : En cas de problème

### Pour la production

🔒 **Activez la sécurité** : Authentification obligatoire
🔐 **Utilisez HTTPS** : Chiffrez les communications
🔑 **Mots de passe forts** : Ne jamais utiliser les valeurs par défaut
🚀 **Cluster multi-nœuds** : Pour la haute disponibilité
💾 **Backups réguliers** : Snapshots Elasticsearch

---

## 📚 Ressources Complémentaires

- 📖 [Documentation Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)
- 🎓 [Kibana Getting Started](https://www.elastic.co/guide/en/kibana/current/get-started.html)
- 🔍 [Kibana Query Language (KQL)](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)
- 📊 [Kibana Visualizations](https://www.elastic.co/guide/en/kibana/current/dashboard.html)
- 🐙 [Images Docker officielles](https://www.docker.elastic.co/)

---

## 🎯 Prochaines Étapes

Maintenant que vous avez Elasticsearch + Kibana, vous pouvez :

- 🔗 [Créer un cluster multi-nœuds](04-cluster-simple.md)
- 📊 Explorer les [cas pratiques - Stack ELK](/cas-pratiques/03-stack-elk.md)
- 🔙 Retour à la [configuration basique](01-config-noeud-unique.md)
- 🌐 Voir la [configuration avec IP fixe](02-config-ip-fixe.md)

---

## 💡 Points Clés à Retenir

✅ **Kibana** est l'interface graphique d'Elasticsearch
✅ Les deux services communiquent via un **réseau Docker partagé**
✅ **Port 5601** pour accéder à Kibana
✅ **Dev Tools** : Console pour tester des requêtes
✅ **Discover** : Explorer vos données visuellement
✅ **Visualizations** : Créer des graphiques
✅ **healthcheck** garantit qu'Elasticsearch est prêt avant Kibana
✅ Au moins **6 GB de RAM** recommandés

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)


