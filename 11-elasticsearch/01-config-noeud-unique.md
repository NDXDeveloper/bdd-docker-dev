# 11.1 Configuration basique d'un nœud unique Elasticsearch

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**Elasticsearch** est un moteur de recherche et d'analyse distribué, basé sur Apache Lucene. Il est conçu pour gérer de grandes quantités de données et effectuer des recherches en temps réel.

### 🎯 Cas d'usage typiques

- 🔍 **Moteur de recherche** : pour sites web, applications
- 📊 **Analyse de logs** : centralisation et analyse (stack ELK)
- 📈 **Analytics** : analyse de données en temps réel
- 🛒 **E-commerce** : recherche de produits, filtres avancés

### 📦 Ce que vous allez apprendre

Dans cette fiche, vous allez mettre en place un **nœud unique** Elasticsearch avec Docker, c'est-à-dire une instance simple et autonome, parfaite pour le développement et les tests.

---

## 🚀 Configuration Basique avec Docker Compose

### Étape 1 : Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ **Docker** installé (version 20.10+)
- ✅ **Docker Compose** installé (version 2.0+)
- ✅ Au moins **4 GB de RAM** disponibles pour Elasticsearch

> ⚠️ **Important** : Elasticsearch est gourmand en mémoire. Si vous avez moins de 8 GB de RAM, ajustez les paramètres mémoire.

### Étape 2 : Créer le fichier `docker-compose.yml`

Créez un nouveau dossier pour votre projet (par exemple `elasticsearch_dev`) et placez-y le fichier suivant :

```yaml
version: '3.8'

services:
  elasticsearch:
    # Version 8.x d'Elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch_dev
    restart: unless-stopped

    environment:
      # Nom du nœud (peut être personnalisé)
      - node.name=es-node01

      # Mode découverte : nœud unique (pas de cluster)
      - discovery.type=single-node

      # Désactive la sécurité (pour développement uniquement)
      # ⚠️ NE JAMAIS faire ça en production !
      - xpack.security.enabled=false

      # Limite la mémoire utilisée (ajustez selon votre RAM)
      # Ici : 1 GB min, 2 GB max
      - "ES_JAVA_OPTS=-Xms1g -Xmx2g"

    ports:
      # Port REST API (pour les requêtes HTTP)
      - "9200:9200"
      # Port de communication inter-nœuds (si cluster futur)
      - "9300:9300"

    volumes:
      # Stockage persistant des données et index
      - elasticsearch_data:/usr/share/elasticsearch/data

    # Augmente les limites de mémoire virtuelle (requis par Elasticsearch)
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

volumes:
  # Volume Docker géré pour les données
  elasticsearch_data:
    driver: local
```

### 📝 Explication des paramètres clés

| Paramètre | Explication |
|-----------|-------------|
| `discovery.type=single-node` | Mode nœud unique (pas de cluster) |
| `xpack.security.enabled=false` | Désactive l'authentification (dev seulement) |
| `ES_JAVA_OPTS=-Xms1g -Xmx2g` | Mémoire : 1 GB min, 2 GB max |
| `9200:9200` | Port API REST pour les requêtes |
| `9300:9300` | Port communication cluster (facultatif ici) |
| `ulimits.memlock` | Désactive le swap (performance) |
| `ulimits.nofile` | Augmente le nombre de fichiers ouverts |

---

## ▶️ Étape 3 : Configuration système (Linux uniquement)

Si vous êtes sur **Linux**, Elasticsearch nécessite une configuration système supplémentaire.

### Augmenter la limite de mémoire virtuelle

```bash
# Vérifier la valeur actuelle
sysctl vm.max_map_count

# Si < 262144, augmentez-la temporairement
sudo sysctl -w vm.max_map_count=262144

# Pour rendre le changement permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

> 💡 **Note** : Sur **Windows** et **macOS** avec Docker Desktop, cette configuration est déjà gérée automatiquement.

---

## ▶️ Étape 4 : Lancer Elasticsearch

Dans le dossier contenant votre `docker-compose.yml`, exécutez :

```bash
# Démarrer en arrière-plan
docker-compose up -d

# Voir les logs en temps réel
docker-compose logs -f elasticsearch
```

### Que se passe-t-il ?

1. Docker télécharge l'image Elasticsearch (~800 MB)
2. Le conteneur démarre et initialise le cluster
3. Elasticsearch crée les fichiers de données dans le volume

⏱️ **Temps de démarrage** : 30 secondes à 1 minute.

Vous verrez dans les logs :

```
[es-node01] started
```

---

## ✅ Étape 5 : Vérifier que ça fonctionne

### Test 1 : Vérifier l'état du cluster

Ouvrez votre navigateur ou utilisez `curl` :

```bash
curl http://localhost:9200
```

**Réponse attendue** (format JSON) :

```json
{
  "name" : "es-node01",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "abc123...",
  "version" : {
    "number" : "8.11.0",
    ...
  },
  "tagline" : "You Know, for Search"
}
```

✅ Si vous voyez ce JSON, **Elasticsearch fonctionne** !

### Test 2 : Vérifier la santé du cluster

```bash
curl http://localhost:9200/_cluster/health?pretty
```

**Réponse attendue** :

```json
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1
}
```

| Status | Signification |
|--------|---------------|
| 🟢 `green` | Tout va bien (toutes les données disponibles) |
| 🟡 `yellow` | Fonctionnel mais réplicas manquants (normal en nœud unique) |
| 🔴 `red` | Problème : données manquantes |

> 💡 En mode nœud unique, le statut `yellow` est normal car il n'y a pas de réplicas.

---

## 📊 Premières Requêtes (Optionnel)

### Créer un index

```bash
curl -X PUT "http://localhost:9200/mon_premier_index?pretty"
```

**Réponse** :

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "mon_premier_index"
}
```

### Ajouter un document

```bash
curl -X POST "http://localhost:9200/mon_premier_index/_doc?pretty" \
     -H 'Content-Type: application/json' \
     -d '{
       "titre": "Mon premier document",
       "contenu": "Elasticsearch est génial !",
       "date": "2025-11-01"
     }'
```

### Rechercher dans l'index

```bash
curl -X GET "http://localhost:9200/mon_premier_index/_search?pretty"
```

Vous verrez votre document dans les résultats ! 🎉

---

## 🔧 Commandes Utiles

### Gestion du conteneur

```bash
# Voir l'état du conteneur
docker-compose ps

# Arrêter Elasticsearch (sans supprimer les données)
docker-compose stop

# Redémarrer
docker-compose start

# Voir les logs
docker-compose logs -f elasticsearch

# Redémarrer complètement
docker-compose restart
```

### Vérifications

```bash
# Lister les index
curl http://localhost:9200/_cat/indices?v

# Statistiques du nœud
curl http://localhost:9200/_nodes/stats?pretty

# Informations sur le cluster
curl http://localhost:9200/_cluster/stats?pretty
```

---

## 🛑 Suppression Complète

Pour tout supprimer (conteneur + données) :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer le volume des données (⚠️ IRRÉVERSIBLE)
docker volume rm <nom_du_dossier>_elasticsearch_data

# Pour trouver le nom exact du volume :
docker volume ls | grep elasticsearch
```

> ⚠️ **Attention** : La suppression du volume efface **toutes vos données** Elasticsearch.

---

## 🐛 Problèmes Courants

### 1. Erreur "max virtual memory areas too low"

**Sur Linux uniquement**

```bash
sudo sysctl -w vm.max_map_count=262144
```

### 2. Le conteneur redémarre en boucle

**Causes possibles** :

- Pas assez de RAM disponible
- `vm.max_map_count` trop bas (Linux)

**Solution** :

```bash
# Réduire la mémoire allouée dans docker-compose.yml
- "ES_JAVA_OPTS=-Xms512m -Xmx1g"
```

### 3. Impossible de se connecter sur localhost:9200

**Vérifiez** :

```bash
# Le conteneur tourne-t-il ?
docker-compose ps

# Les logs montrent-ils des erreurs ?
docker-compose logs elasticsearch
```

---

## 📚 Ressources Complémentaires

- 📖 [Documentation officielle Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- 🎓 [Guide de démarrage Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
- 🐙 [Image Docker officielle](https://www.docker.elastic.co/r/elasticsearch)

---

## 🎯 Prochaines Étapes

Maintenant que votre nœud Elasticsearch fonctionne, vous pouvez :

- 🌐 [Configuration avec IP fixe](02-config-ip-fixe.md)
- 🖥️ [Ajouter Kibana pour visualiser](03-elasticsearch-kibana.md)
- 🔗 [Créer un cluster multi-nœuds](04-cluster-simple.md)
- 📊 Consulter les [cas pratiques - Stack ELK](/cas-pratiques/03-stack-elk.md)

---

## 💡 Points Clés à Retenir

✅ Elasticsearch nécessite au moins **2 GB de RAM**
✅ Le mode `single-node` est parfait pour le développement
✅ La sécurité est **désactivée** (ne pas utiliser en production)
✅ Les données persistent dans un **volume Docker**
✅ Le port **9200** expose l'API REST

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

