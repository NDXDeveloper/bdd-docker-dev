# 8.1 Configuration basique avec docker-compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction à Neo4j

**Neo4j** est une base de données orientée **graphes** (Graph Database). Contrairement aux bases de données relationnelles traditionnelles qui stockent les données dans des tables, Neo4j stocke les informations sous forme de **nœuds** (nodes) reliés par des **relations** (relationships).

### 🤔 Pourquoi Neo4j ?

Neo4j excelle dans les scénarios où les **relations entre les données** sont aussi importantes que les données elles-mêmes :

- **Réseaux sociaux** : Amis, followers, recommandations
- **Systèmes de recommandation** : "Les clients qui ont acheté X ont aussi acheté Y"
- **Détection de fraude** : Analyse de réseaux suspects
- **Gestion des connaissances** : Graphes de concepts, ontologies
- **Routage et logistique** : Chemins optimaux, dépendances

### 🎯 Concepts de base

| Concept | Description | Exemple |
|---------|-------------|---------|
| **Nœud (Node)** | Entité du graphe | Une personne, un produit |
| **Relation (Relationship)** | Lien entre deux nœuds | "Alice CONNAIT Bob" |
| **Propriétés** | Attributs des nœuds/relations | `nom: "Alice"`, `depuis: 2020` |
| **Label** | Type de nœud | `:Personne`, `:Produit` |
| **Cypher** | Langage de requête Neo4j | `MATCH (n:Personne) RETURN n` |

---

## 🚀 Configuration de base avec Docker Compose

### Prérequis

- **Docker** et **Docker Compose** installés
- **8 Go de RAM** minimum (Neo4j peut être gourmand)
- **Ports disponibles** : 7474 (HTTP), 7687 (Bolt)

---

## 📁 Étape 1 : Créer le projet

Créez un nouveau dossier pour votre projet Neo4j :

```bash
mkdir neo4j-docker
cd neo4j-docker
```

---

## 📝 Étape 2 : Créer le fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` dans ce dossier avec le contenu suivant :

```yaml
version: '3.8'

services:
  neo4j:
    # Image officielle Neo4j (version Community gratuite)
    image: neo4j:5.13
    container_name: neo4j_dev
    restart: unless-stopped

    # Ports exposés
    ports:
      # 7474 : Interface web Neo4j Browser
      - "7474:7474"
      # 7687 : Port Bolt (protocole binaire pour les drivers)
      - "7687:7687"

    # Variables d'environnement
    environment:
      # Désactive l'authentification (POUR LE DEV UNIQUEMENT !)
      # En production, TOUJOURS utiliser un mot de passe
      NEO4J_AUTH: none

      # Accepter la licence (nécessaire pour Neo4j)
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

    # Volume pour la persistance des données
    volumes:
      # Données de la base
      - neo4j_data:/data
      # Logs Neo4j
      - neo4j_logs:/logs
      # Imports (pour charger des fichiers CSV par exemple)
      - neo4j_import:/var/lib/neo4j/import
      # Plugins (extensions Neo4j)
      - neo4j_plugins:/plugins

# Déclaration des volumes nommés
volumes:
  neo4j_data:
  neo4j_logs:
  neo4j_import:
  neo4j_plugins:
```

### 🔍 Explication des éléments clés

#### **Image Neo4j**
```yaml
image: neo4j:5.13
```
- **`neo4j:5.13`** : Version 5.13 (Community Edition)
- Vous pouvez utiliser `neo4j:latest` pour la dernière version
- Pour l'Enterprise Edition (payante) : `neo4j:5.13-enterprise`

#### **Ports**
```yaml
ports:
  - "7474:7474"  # Interface web
  - "7687:7687"  # Connexions Bolt
```
- **7474** : Accès à **Neo4j Browser** (interface graphique web)
- **7687** : Port **Bolt** pour les applications (drivers Python, Java, JavaScript...)

#### **Authentification**
```yaml
NEO4J_AUTH: none
```
⚠️ **IMPORTANT** : `NEO4J_AUTH: none` désactive complètement l'authentification.

**Pour activer un mot de passe (recommandé), utilisez** :
```yaml
NEO4J_AUTH: neo4j/votre_mot_de_passe_securise
```
- Format : `utilisateur/mot_de_passe`
- Par défaut, l'utilisateur est `neo4j`

#### **Volumes**
- **`neo4j_data`** : Stocke les données du graphe (persistance)
- **`neo4j_logs`** : Logs d'activité
- **`neo4j_import`** : Dossier pour importer des fichiers CSV
- **`neo4j_plugins`** : Extensions Neo4j (APOC, GDS...)

---

## ▶️ Étape 3 : Démarrer Neo4j

Dans votre terminal, depuis le dossier contenant `docker-compose.yml` :

```bash
# Démarrer Neo4j en arrière-plan
docker-compose up -d
```

**Sortie attendue :**
```
Creating network "neo4j-docker_default" with the default driver
Creating volume "neo4j-docker_neo4j_data" with default driver
Creating volume "neo4j-docker_neo4j_logs" with default driver
Creating volume "neo4j-docker_neo4j_import" with default driver
Creating volume "neo4j-docker_neo4j_plugins" with default driver
Creating neo4j_dev ... done
```

### Vérifier que le conteneur tourne

```bash
docker-compose ps
```

**Résultat :**
```
    Name                  Command               State           Ports
-----------------------------------------------------------------------------
neo4j_dev   tini -g -- /startup/docker ...   Up      0.0.0.0:7474->7474/tcp,
                                                      0.0.0.0:7687->7687/tcp
```

### Voir les logs (optionnel)

```bash
# Suivre les logs en temps réel
docker-compose logs -f neo4j

# Arrêter le suivi avec Ctrl+C
```

**Attendez quelques secondes** que Neo4j démarre complètement (généralement 10-30 secondes).

---

## 🌐 Étape 4 : Accéder à Neo4j Browser

Neo4j Browser est l'**interface graphique web** pour interagir avec votre base de données.

### Ouvrir dans le navigateur

```
http://localhost:7474
```

### 🎨 Interface Neo4j Browser

![Capture Neo4j Browser](https://neo4j.com/docs/browser-manual/current/_images/browser-overview.png)

À votre première connexion :

1. **Connect URL** : `neo4j://localhost:7687` (pré-rempli)
2. **Authentication type** :
   - Si vous avez mis `NEO4J_AUTH: none` → Sélectionnez **"No authentication"**
   - Sinon → Utilisez `neo4j` / votre mot de passe
3. Cliquez sur **"Connect"**

---

## 🧪 Étape 5 : Premiers pas avec Cypher

**Cypher** est le langage de requête de Neo4j (équivalent de SQL pour les graphes).

### 1️⃣ Créer vos premiers nœuds

Dans Neo4j Browser, tapez et exécutez (Ctrl+Entrée ou cliquez sur le bouton ▶) :

```cypher
// Créer deux nœuds de type "Personne"
CREATE (alice:Personne {nom: 'Alice', age: 30})
CREATE (bob:Personne {nom: 'Bob', age: 25})
```

**Explication :**
- `CREATE` : Créer un élément
- `(alice:Personne ...)` : Nœud avec label `:Personne`
- `{nom: 'Alice', age: 30}` : Propriétés du nœud

**Résultat :** "Added 2 labels, created 2 nodes, set 4 properties, completed after X ms."

### 2️⃣ Créer une relation

```cypher
// Alice connaît Bob depuis 2020
MATCH (a:Personne {nom: 'Alice'}), (b:Personne {nom: 'Bob'})
CREATE (a)-[r:CONNAIT {depuis: 2020}]->(b)
RETURN a, r, b
```

**Explication :**
- `MATCH` : Rechercher des nœuds existants
- `CREATE (a)-[r:CONNAIT]-(b)` : Créer une relation `:CONNAIT`
- `RETURN` : Afficher le résultat

**Résultat visuel :** Vous verrez un graphe avec deux nœuds reliés ! 🎉

### 3️⃣ Afficher tout le graphe

```cypher
// Afficher tous les nœuds et relations
MATCH (n)
RETURN n
```

### 4️⃣ Rechercher avec des conditions

```cypher
// Trouver les personnes de plus de 28 ans
MATCH (p:Personne)
WHERE p.age > 28
RETURN p.nom, p.age
```

**Résultat :**
```
╒═════════╤═══════╕
│ p.nom   │ p.age │
╞═════════╪═══════╡
│ "Alice" │ 30    │
└─────────┴───────┘
```

### 5️⃣ Supprimer des données

```cypher
// Supprimer tous les nœuds et relations (ATTENTION !)
MATCH (n)
DETACH DELETE n
```

⚠️ **DETACH DELETE** supprime le nœud ET toutes ses relations.

---

## 📊 Commandes utiles Docker Compose

```bash
# Voir les logs
docker-compose logs -f neo4j

# Arrêter Neo4j (sans supprimer les données)
docker-compose stop

# Redémarrer Neo4j
docker-compose start

# Arrêter ET supprimer le conteneur (les données restent dans les volumes)
docker-compose down

# Supprimer TOUT (conteneur + volumes = perte de données)
docker-compose down -v
```

---

## 🔧 Configuration avancée (optionnelle)

### Activer l'authentification

Modifiez la section `environment` :

```yaml
environment:
  # Utilisateur : neo4j, Mot de passe : monMotDePasseSecurise123
  NEO4J_AUTH: neo4j/monMotDePasseSecurise123
  NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'
```

Relancez :
```bash
docker-compose down
docker-compose up -d
```

À la connexion, utilisez :
- **Utilisateur** : `neo4j`
- **Mot de passe** : `monMotDePasseSecurise123`

### Limiter la mémoire utilisée

Ajoutez ces variables d'environnement (exemple pour 2 Go de heap) :

```yaml
environment:
  NEO4J_AUTH: none
  NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'
  # Limiter la mémoire heap (tas Java)
  NEO4J_server_memory_heap_initial__size: '512m'
  NEO4J_server_memory_heap_max__size: '2G'
  # Limiter le cache des pages
  NEO4J_server_memory_pagecache_size: '512m'
```

---

## 🗑️ Nettoyage complet

Pour supprimer complètement Neo4j et ses données :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer les volumes (SUPPRIME LES DONNÉES)
docker volume rm neo4j-docker_neo4j_data
docker volume rm neo4j-docker_neo4j_logs
docker volume rm neo4j-docker_neo4j_import
docker volume rm neo4j-docker_neo4j_plugins

# Ou en une commande (depuis le dossier du projet)
docker-compose down -v
```

---

## 🐛 Dépannage

### Le conteneur ne démarre pas

**Vérifier les logs :**
```bash
docker-compose logs neo4j
```

**Problèmes courants :**
- **Manque de mémoire** : Neo4j nécessite au moins 4 Go de RAM disponibles
- **Port déjà utilisé** : Changez `7474:7474` en `7475:7474` (par exemple)

### Impossible de se connecter à Neo4j Browser

1. **Vérifier que le conteneur tourne** : `docker-compose ps`
2. **Attendre le démarrage complet** : 20-30 secondes après `up -d`
3. **Vérifier l'URL** : `http://localhost:7474` (pas `https://`)
4. **Tester le port Bolt** : `telnet localhost 7687`

### "Database not available"

Neo4j met du temps à démarrer. Attendez 30 secondes et rafraîchissez la page.

---

## 📚 Ressources complémentaires

- 📖 [Documentation officielle Neo4j](https://neo4j.com/docs/)
- 🎓 [Neo4j GraphAcademy (cours gratuits)](https://graphacademy.neo4j.com/)
- 📝 [Cypher Query Language](https://neo4j.com/docs/cypher-manual/current/)
- 🎮 [Neo4j Sandbox (tester en ligne)](https://sandbox.neo4j.com/)

---

## ✅ Récapitulatif

Vous avez appris à :

✅ Créer un fichier `docker-compose.yml` pour Neo4j
✅ Démarrer Neo4j avec Docker Compose
✅ Accéder à Neo4j Browser
✅ Exécuter vos premières requêtes Cypher
✅ Créer des nœuds, relations et requêter le graphe
✅ Gérer l'authentification et la configuration

**Prochaines étapes :**
- [Configuration avec IP fixe](02-config-ip-fixe.md)
- [Configuration avancée avec plugins](03-config-avec-plugins.md)
- [Import de données CSV](04-import-donnees-csv.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
