# 11.2 Configuration avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans cette fiche, nous allons configurer Elasticsearch avec une **adresse IP statique** dans Docker. Cela permet à votre conteneur d'avoir toujours la même adresse IP, ce qui est utile pour :

- 🔗 **Communication inter-conteneurs** : D'autres services peuvent toujours trouver Elasticsearch
- 🧪 **Tests** : L'IP ne change pas entre les redémarrages
- 📝 **Configuration simplifiée** : Plus besoin de chercher l'IP dynamique
- 🔧 **Déploiement** : Configuration réseau prévisible

### 🎯 Ce que vous allez apprendre

- Créer un réseau Docker personnalisé
- Assigner une IP fixe à Elasticsearch
- Vérifier la configuration réseau
- Connecter d'autres services à Elasticsearch

---

## 🚀 Étape 1 : Création du Réseau Docker Dédié

Avant de lancer Elasticsearch, nous devons créer un réseau Docker avec une plage d'adresses IP définie.

### Commande de création du réseau

Ouvrez votre terminal et exécutez **une seule fois** :

```bash
docker network create --subnet=172.20.0.0/16 elasticsearch_net
```

### 📝 Explication des paramètres

| Paramètre | Explication |
|-----------|-------------|
| `--subnet=172.20.0.0/16` | Définit la plage d'adresses IP (de 172.20.0.1 à 172.20.255.254) |
| `elasticsearch_net` | Nom de notre réseau personnalisé |

### Vérifier la création du réseau

```bash
# Lister tous les réseaux Docker
docker network ls

# Inspecter notre réseau
docker network inspect elasticsearch_net
```

Vous devriez voir votre réseau `elasticsearch_net` dans la liste.

---

## 🗂️ Étape 2 : Fichiers de Configuration

Créez un nouveau dossier pour votre projet (par exemple `elasticsearch_ip_fixe`) et placez-y le fichier suivant.

### Fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  elasticsearch:
    # Version 8.x d'Elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch_with_ip
    restart: unless-stopped

    environment:
      # Nom du nœud
      - node.name=es-node01

      # Mode découverte : nœud unique
      - discovery.type=single-node

      # Désactive la sécurité (développement uniquement)
      - xpack.security.enabled=false

      # Mémoire : 1 GB min, 2 GB max (ajustez selon votre RAM)
      - "ES_JAVA_OPTS=-Xms1g -Xmx2g"

      # Important : définit l'interface d'écoute
      # 0.0.0.0 permet les connexions depuis n'importe quelle IP
      - network.host=0.0.0.0

    ports:
      # API REST
      - "9200:9200"
      # Communication inter-nœuds
      - "9300:9300"

    volumes:
      # Stockage persistant
      - elasticsearch_data:/usr/share/elasticsearch/data

    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

    # Configuration réseau : IP fixe
    networks:
      elasticsearch_net:
        # ⭐ L'adresse IP fixe assignée au conteneur
        ipv4_address: 172.20.0.10

# Déclaration du volume
volumes:
  elasticsearch_data:
    driver: local

# Déclaration du réseau
networks:
  elasticsearch_net:
    # Indique que ce réseau existe déjà (créé à l'étape 1)
    external: true
```

### 🔍 Points clés de la configuration

| Élément | Explication |
|---------|-------------|
| `network.host=0.0.0.0` | Permet les connexions depuis n'importe quelle IP (pas seulement localhost) |
| `ipv4_address: 172.20.0.10` | **L'IP fixe** assignée à Elasticsearch |
| `external: true` | Indique que le réseau a été créé manuellement |

> 💡 **Pourquoi 172.20.0.10 ?** Vous pouvez choisir n'importe quelle IP dans la plage 172.20.0.2 à 172.20.255.254. L'adresse .10 est un bon choix pour éviter les conflits avec .2, .3, etc.

---

## ▶️ Étape 3 : Configuration Système (Linux uniquement)

Si vous êtes sur **Linux**, configurez la mémoire virtuelle :

```bash
# Vérifier la valeur actuelle
sysctl vm.max_map_count

# Si < 262144, augmentez-la
sudo sysctl -w vm.max_map_count=262144

# Rendre le changement permanent
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

> 💡 Sur **Windows** et **macOS**, cette configuration est automatique.

---

## ▶️ Étape 4 : Lancer Elasticsearch

Dans le dossier contenant votre `docker-compose.yml` :

```bash
# Démarrer en arrière-plan
docker-compose up -d

# Voir les logs
docker-compose logs -f elasticsearch
```

⏱️ **Attendre 30 secondes à 1 minute** que Elasticsearch démarre complètement.

---

## ✅ Étape 5 : Vérifications

### Test 1 : Vérifier l'IP assignée

```bash
# Inspecter le conteneur pour voir son IP
docker inspect elasticsearch_with_ip | grep "IPAddress"
```

**Résultat attendu** :

```json
"IPAddress": "172.20.0.10"
```

✅ Vous devriez voir `172.20.0.10` !

### Test 2 : Tester la connexion via l'IP fixe

```bash
# Connexion via l'IP fixe
curl http://172.20.0.10:9200

# Connexion via localhost (fonctionne aussi)
curl http://localhost:9200
```

**Réponse attendue** (les deux commandes) :

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

### Test 3 : Vérifier la santé du cluster

```bash
# Via l'IP fixe
curl http://172.20.0.10:9200/_cluster/health?pretty

# Via localhost
curl http://localhost:9200/_cluster/health?pretty
```

---

## 🔗 Utilisation avec d'autres Conteneurs

### Exemple : Application Node.js

Voici un exemple de service qui se connecte à Elasticsearch via son IP fixe.

**Ajoutez ce service dans votre `docker-compose.yml`** :

```yaml
services:
  # ... (elasticsearch service comme avant) ...

  app:
    image: node:18
    container_name: app_nodejs
    working_dir: /app
    command: sh -c "npm install && node app.js"
    volumes:
      - ./app:/app
    networks:
      elasticsearch_net:
        ipv4_address: 172.20.0.20
    depends_on:
      - elasticsearch
    environment:
      # L'app peut utiliser l'IP fixe pour se connecter
      - ELASTICSEARCH_URL=http://172.20.0.10:9200
```

### Connexion depuis l'application

Dans votre code Node.js (`app/app.js`) :

```javascript
const { Client } = require('@elastic/elasticsearch');

// Connexion via l'IP fixe
const client = new Client({
  node: process.env.ELASTICSEARCH_URL || 'http://172.20.0.10:9200'
});

// Test de connexion
async function testConnection() {
  try {
    const info = await client.info();
    console.log('✅ Connecté à Elasticsearch:', info.version.number);
  } catch (error) {
    console.error('❌ Erreur de connexion:', error);
  }
}

testConnection();
```

---

## 🌐 Accès depuis un Autre Conteneur sur le Même Réseau

### Méthode 1 : Via l'IP fixe

```bash
# Depuis un conteneur sur le réseau elasticsearch_net
curl http://172.20.0.10:9200
```

### Méthode 2 : Via le nom du service

Docker Compose crée automatiquement un DNS. Vous pouvez aussi utiliser le nom du service :

```bash
# Fonctionne aussi !
curl http://elasticsearch:9200
```

> 💡 **Recommandation** : Utilisez l'IP fixe si vous avez besoin d'une adresse prévisible, sinon le nom du service est plus simple.

---

## 🔧 Commandes Utiles

### Gestion du réseau

```bash
# Lister les réseaux
docker network ls

# Inspecter le réseau
docker network inspect elasticsearch_net

# Voir quels conteneurs sont sur ce réseau
docker network inspect elasticsearch_net | grep Name
```

### Gestion du conteneur

```bash
# État du conteneur
docker-compose ps

# Logs
docker-compose logs -f elasticsearch

# Redémarrer
docker-compose restart

# Arrêter
docker-compose stop

# Redémarrer
docker-compose start
```

### Tests de connexion

```bash
# Via localhost
curl http://localhost:9200/_cat/health?v

# Via l'IP fixe
curl http://172.20.0.10:9200/_cat/health?v

# Lister les index
curl http://172.20.0.10:9200/_cat/indices?v
```

---

## 🛑 Suppression Complète

Pour tout supprimer (conteneur, volume ET réseau) :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer le volume des données (⚠️ IRRÉVERSIBLE)
docker volume rm <nom_du_dossier>_elasticsearch_data

# Pour trouver le nom exact :
docker volume ls | grep elasticsearch

# 3. Supprimer le réseau personnalisé
docker network rm elasticsearch_net
```

> ⚠️ **Attention** : Ces commandes suppriment **toutes vos données** et la configuration réseau.

---

## 🔄 Tableau Récapitulatif des Accès

| Depuis... | Méthode d'accès | Exemple |
|-----------|----------------|---------|
| 🖥️ Machine hôte | `localhost` | `http://localhost:9200` |
| 🖥️ Machine hôte | IP fixe (via réseau Docker) | `http://172.20.0.10:9200` |
| 🐳 Autre conteneur (même réseau) | IP fixe | `http://172.20.0.10:9200` |
| 🐳 Autre conteneur (même réseau) | Nom du service | `http://elasticsearch:9200` |
| 🌐 Réseau local | IP de la machine hôte | `http://192.168.x.x:9200` |

---

## 🐛 Problèmes Courants

### 1. Erreur "address already in use"

**Cause** : L'IP 172.20.0.10 est déjà utilisée.

**Solution** :

```bash
# Changer l'IP dans docker-compose.yml
ipv4_address: 172.20.0.11  # ou .12, .13, etc.
```

### 2. Le réseau n'existe pas

**Erreur** : `network elasticsearch_net declared as external, but could not be found`

**Solution** :

```bash
# Créer le réseau manuellement
docker network create --subnet=172.20.0.0/16 elasticsearch_net
```

### 3. Impossible de se connecter via l'IP fixe

**Vérifications** :

```bash
# 1. Le conteneur tourne-t-il ?
docker-compose ps

# 2. L'IP est-elle correcte ?
docker inspect elasticsearch_with_ip | grep "IPAddress"

# 3. Le réseau est-il actif ?
docker network ls
docker network inspect elasticsearch_net
```

### 4. Conflit de subnet

**Erreur** : `Pool overlaps with other one on this address space`

**Solution** : Utilisez une autre plage d'IP :

```bash
# Essayez avec 172.21.0.0/16 au lieu de 172.20.0.0/16
docker network create --subnet=172.21.0.0/16 elasticsearch_net

# Et ajustez l'IP dans docker-compose.yml
ipv4_address: 172.21.0.10
```

---

## 📊 Comparaison : Avec vs Sans IP Fixe

| Critère | Sans IP fixe | Avec IP fixe |
|---------|--------------|--------------|
| **Simplicité** | ✅ Plus simple | ⚠️ Configuration supplémentaire |
| **Prévisibilité** | ❌ IP change à chaque redémarrage | ✅ Toujours la même IP |
| **Communication** | ✅ Via nom de service | ✅ Via IP ou nom |
| **Tests** | ⚠️ IP dynamique | ✅ IP stable |
| **Production** | ✅ Recommandé (avec orchestrateur) | ⚠️ Gérer manuellement |

> 💡 **Recommandation** : Utilisez l'IP fixe pour le développement si vous avez besoin de stabilité. En production, préférez les orchestrateurs (Kubernetes, Docker Swarm) qui gèrent les IP automatiquement.

---

## 📚 Ressources Complémentaires

- 📖 [Documentation Docker - Networking](https://docs.docker.com/network/)
- 📖 [Elasticsearch Network Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)
- 🔗 [Annexe B - Gestion des réseaux Docker](/annexes/B-gestion-reseaux.md)

---

## 🎯 Prochaines Étapes

Maintenant que votre Elasticsearch a une IP fixe, vous pouvez :

- 🖥️ [Ajouter Kibana pour visualiser les données](03-elasticsearch-kibana.md)
- 🔗 [Créer un cluster multi-nœuds](04-cluster-simple.md)
- 📊 Consulter les [cas pratiques - Stack ELK](/cas-pratiques/03-stack-elk.md)
- 🏠 Retour à la [configuration basique](01-config-noeud-unique.md)

---

## 💡 Points Clés à Retenir

✅ Un **réseau Docker dédié** est nécessaire pour les IP fixes
✅ L'IP fixe est définie dans `ipv4_address` du `docker-compose.yml`
✅ Le paramètre `network.host=0.0.0.0` permet les connexions externes
✅ Les autres conteneurs peuvent utiliser l'**IP fixe** ou le **nom du service**
✅ L'IP fixe simplifie les tests et la configuration
✅ Pensez à **supprimer le réseau** lors du nettoyage complet

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

