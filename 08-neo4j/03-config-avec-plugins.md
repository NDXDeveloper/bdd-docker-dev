# 8.3 Configuration avancée avec plugins

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction aux plugins Neo4j

Neo4j, dans sa version de base, offre déjà de nombreuses fonctionnalités. Cependant, les **plugins** (ou **extensions**) permettent d'étendre considérablement ses capacités.

### 🤔 Qu'est-ce qu'un plugin Neo4j ?

Un plugin est un fichier `.jar` (Java Archive) qui ajoute de nouvelles **procédures** et **fonctions** à Neo4j. Ces extensions peuvent être appelées depuis vos requêtes Cypher pour réaliser des tâches complexes.

**Exemple simple :**
```cypher
// Sans plugin : calcul manuel du chemin le plus court
MATCH path = shortestPath((a:Personne {nom: 'Alice'})-[*]-(b:Personne {nom: 'Bob'}))
RETURN path

// Avec plugin APOC : fonctions avancées de traitement
CALL apoc.path.expandConfig(startNode, {relationshipFilter: 'CONNAIT>'})
YIELD path
RETURN path
```

---

## 🔌 Les plugins Neo4j les plus populaires

### 1️⃣ **APOC (Awesome Procedures On Cypher)**

**Le plugin incontournable** - Plus de 450 procédures et fonctions utilitaires.

| Catégorie | Exemples de fonctionnalités |
|-----------|---------------------------|
| **Manipulation de données** | Création en masse, fusion, clonage de nœuds |
| **Import/Export** | JSON, CSV, XML, fichiers |
| **Algorithmes de graphes** | Chemins, centralité, communautés |
| **Conversion de types** | Dates, textes, listes, maps |
| **Triggers** | Actions automatiques sur événements |
| **Cypher dynamique** | Construire et exécuter des requêtes à la volée |

📖 [Documentation APOC](https://neo4j.com/labs/apoc/)

### 2️⃣ **Graph Data Science (GDS)**

**Algorithmes de science des données** pour l'analyse de graphes à grande échelle.

| Algorithme | Usage |
|------------|-------|
| **PageRank** | Mesurer l'importance des nœuds |
| **Louvain** | Détection de communautés |
| **Node Similarity** | Recommandations |
| **Shortest Path** | Routage optimal |
| **Betweenness Centrality** | Identifier les nœuds "ponts" |

📖 [Documentation GDS](https://neo4j.com/docs/graph-data-science/current/)

### 3️⃣ **Neosemantics (n10s)**

**Import et gestion de données RDF/OWL** - Pour les ontologies et le web sémantique.

📖 [Documentation n10s](https://neo4j.com/labs/neosemantics/)

### 4️⃣ **Streams**

**Intégration avec Apache Kafka** - Pour les flux de données en temps réel.

📖 [Documentation Streams](https://neo4j.com/labs/kafka/)

---

## 🚀 Configuration de base avec APOC

Nous allons installer **APOC**, le plugin le plus utilisé et le plus pratique pour débuter.

### Étape 1 : Créer le projet

```bash
mkdir neo4j-avec-apoc
cd neo4j-avec-apoc
```

### Étape 2 : Créer le fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.13
    container_name: neo4j_avec_apoc
    restart: unless-stopped

    ports:
      - "7474:7474"  # Neo4j Browser
      - "7687:7687"  # Bolt

    environment:
      NEO4J_AUTH: none
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

      # IMPORTANT : Activer APOC
      # Sans cette ligne, APOC ne sera pas chargé même s'il est installé
      NEO4J_PLUGINS: '["apoc"]'

      # Autoriser toutes les procédures APOC (pour le dev)
      # En production, restreindre aux procédures nécessaires
      NEO4J_dbms_security_procedures_unrestricted: 'apoc.*'

      # Autoriser les imports depuis n'importe quel chemin (pour le dev)
      NEO4J_dbms_security_allow__csv__import__from__file__urls: 'true'

    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
      - neo4j_import:/var/lib/neo4j/import
      - neo4j_plugins:/plugins

volumes:
  neo4j_data:
  neo4j_logs:
  neo4j_import:
  neo4j_plugins:
```

### Étape 3 : Démarrer Neo4j

```bash
docker-compose up -d
```

**⏱️ Attention :** Le premier démarrage prend **1-2 minutes** car Neo4j doit :
1. Télécharger le plugin APOC depuis Internet
2. L'installer dans `/plugins`
3. Charger toutes les procédures

### Étape 4 : Vérifier les logs

```bash
docker-compose logs -f neo4j
```

**Lignes à chercher :**
```
INFO  Loaded APOC (Awesome Procedures On Cypher) ...
INFO  Started.
```

Une fois `Started.` affiché, c'est prêt ! (Ctrl+C pour quitter les logs)

---

## 🔍 Explication de la configuration APOC

### 1️⃣ Activation du plugin

```yaml
NEO4J_PLUGINS: '["apoc"]'
```

**Format :**
- **JSON array** entre quotes : `'["plugin1", "plugin2"]'`
- Pour plusieurs plugins : `'["apoc", "graph-data-science"]'`

**Plugins disponibles via cette méthode :**
- `apoc` (Core + Extended)
- `graph-data-science`
- `bloom` (Neo4j Bloom - licence Enterprise)

### 2️⃣ Autorisation des procédures

```yaml
NEO4J_dbms_security_procedures_unrestricted: 'apoc.*'
```

**Signification :**
- `apoc.*` : Autorise **toutes** les procédures APOC
- Par défaut, seules certaines procédures sont autorisées (sécurité)

**Pour restreindre (production) :**
```yaml
# Autoriser seulement certaines procédures
NEO4J_dbms_security_procedures_unrestricted: 'apoc.load.*, apoc.create.*'
```

### 3️⃣ Import de fichiers CSV

```yaml
NEO4J_dbms_security_allow__csv__import__from__file__urls: 'true'
```

**Pourquoi cette ligne ?**
- Par défaut, Neo4j bloque les imports depuis des URLs ou chemins arbitraires
- `true` : Permet d'importer des CSV depuis n'importe où
- ⚠️ **En production**, utilisez `false` et placez les fichiers dans `/import`

---

## ✅ Vérifier l'installation d'APOC

### Méthode 1 : Lister toutes les procédures APOC

Ouvrez Neo4j Browser (`http://localhost:7474`) et exécutez :

```cypher
CALL dbms.procedures()
YIELD name, signature
WHERE name STARTS WITH 'apoc'
RETURN name, signature
LIMIT 10
```

**Résultat attendu :**
```
╒═══════════════════════╤═══════════════════════════════╕
│ name                  │ signature                     │
╞═══════════════════════╪═══════════════════════════════╡
│ apoc.create.node      │ apoc.create.node(...)         │
│ apoc.load.json        │ apoc.load.json(...)           │
│ apoc.merge.node       │ apoc.merge.node(...)          │
│ ...                   │ ...                           │
└───────────────────────┴───────────────────────────────┘
```

Si vous voyez des procédures `apoc.*`, c'est installé ! 🎉

### Méthode 2 : Tester une procédure simple

```cypher
// Obtenir la version d'APOC
RETURN apoc.version() AS version
```

**Résultat :**
```
╒═════════╕
│ version │
╞═════════╡
│ "5.13.0"│
└─────────┘
```

### Méthode 3 : Vérifier les fichiers

```bash
# Lister le contenu du dossier plugins
docker exec neo4j_avec_apoc ls -lh /var/lib/neo4j/plugins
```

**Résultat attendu :**
```
-rw-r--r-- 1 neo4j neo4j  15M Oct 31 12:00 apoc-5.13.0-core.jar
-rw-r--r-- 1 neo4j neo4j  8.5M Oct 31 12:00 apoc-5.13.0-extended.jar
```

---

## 🧪 Exemples d'utilisation d'APOC

### 1️⃣ Créer plusieurs nœuds rapidement

Sans APOC (répétitif) :
```cypher
CREATE (:Ville {nom: 'Paris'})
CREATE (:Ville {nom: 'Lyon'})
CREATE (:Ville {nom: 'Marseille'})
```

Avec APOC (en une seule requête) :
```cypher
CALL apoc.create.nodes(['Ville', 'Ville', 'Ville'],
  [
    {nom: 'Paris'},
    {nom: 'Lyon'},
    {nom: 'Marseille'}
  ]
)
YIELD node
RETURN node
```

### 2️⃣ Charger des données JSON depuis une URL

```cypher
// Charger les utilisateurs depuis une API REST
CALL apoc.load.json('https://jsonplaceholder.typicode.com/users')
YIELD value
MERGE (u:Utilisateur {id: value.id})
SET u.nom = value.name, u.email = value.email
RETURN u
LIMIT 5
```

### 3️⃣ Exécuter du Cypher dynamique

```cypher
// Construire une requête dynamiquement
CALL apoc.cypher.run(
  'MATCH (n:Personne) WHERE n.age > $age RETURN n',
  {age: 25}
)
YIELD value
RETURN value.n AS personne
```

### 4️⃣ Générer un UUID

```cypher
// Créer un identifiant unique
CREATE (p:Produit {id: apoc.create.uuid(), nom: 'Laptop'})
RETURN p
```

### 5️⃣ Convertir une date

```cypher
// Parser une date depuis un texte
RETURN apoc.date.parse('2025-10-31', 'ms', 'yyyy-MM-dd') AS timestamp
```

**Résultat :**
```
╒═══════════════╕
│ timestamp     │
╞═══════════════╡
│ 1730332800000 │
└───────────────┘
```

---

## 🔬 Configuration avec Graph Data Science (GDS)

GDS est idéal pour l'analyse de graphes complexes. **Attention :** GDS est plus lourd qu'APOC (~150 Mo).

### Fichier `docker-compose.yml` avec GDS

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.13
    container_name: neo4j_avec_gds
    restart: unless-stopped

    ports:
      - "7474:7474"
      - "7687:7687"

    environment:
      NEO4J_AUTH: none
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

      # Installer APOC ET GDS
      NEO4J_PLUGINS: '["apoc", "graph-data-science"]'

      # Autoriser APOC et GDS
      NEO4J_dbms_security_procedures_unrestricted: 'apoc.*, gds.*'

      # GDS nécessite plus de mémoire
      NEO4J_server_memory_heap_initial__size: '1G'
      NEO4J_server_memory_heap_max__size: '4G'
      NEO4J_server_memory_pagecache_size: '1G'

    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
      - neo4j_import:/var/lib/neo4j/import
      - neo4j_plugins:/plugins

volumes:
  neo4j_data:
  neo4j_logs:
  neo4j_import:
  neo4j_plugins:
```

### Vérifier l'installation de GDS

```cypher
// Lister les procédures GDS
CALL dbms.procedures()
YIELD name
WHERE name STARTS WITH 'gds'
RETURN name
LIMIT 5
```

### Exemple : Algorithme PageRank avec GDS

```cypher
// 1. Créer un graphe exemple
CREATE (a:Page {nom: 'A'})
CREATE (b:Page {nom: 'B'})
CREATE (c:Page {nom: 'C'})
CREATE (a)-[:LIEN]->(b)
CREATE (b)-[:LIEN]->(c)
CREATE (c)-[:LIEN]->(a)

// 2. Créer une projection de graphe en mémoire
CALL gds.graph.project('monGraphe', 'Page', 'LIEN')

// 3. Exécuter PageRank
CALL gds.pageRank.stream('monGraphe')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).nom AS page, score
ORDER BY score DESC
```

**Résultat :**
```
╒══════╤═══════════════╕
│ page │ score         │
╞══════╪═══════════════╡
│ "A"  │ 0.3333333333  │
│ "B"  │ 0.3333333333  │
│ "C"  │ 0.3333333333  │
└──────┴───────────────┘
```

---

## 🔧 Configuration avancée : Plugins personnalisés

Si vous avez développé votre propre plugin ou téléchargé un `.jar` non officiel.

### Méthode 1 : Via bind mount (recommandé pour le dev)

**Structure du projet :**
```
neo4j-custom-plugin/
├── docker-compose.yml
└── plugins/
    └── mon-plugin-1.0.0.jar
```

**docker-compose.yml :**
```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.13
    container_name: neo4j_custom_plugin
    restart: unless-stopped

    ports:
      - "7474:7474"
      - "7687:7687"

    environment:
      NEO4J_AUTH: none
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

      # Autoriser votre plugin
      NEO4J_dbms_security_procedures_unrestricted: 'com.monentreprise.*'

    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
      # Bind mount du dossier local contenant le plugin
      - ./plugins:/var/lib/neo4j/plugins

volumes:
  neo4j_data:
  neo4j_logs:
```

**Utilisation :**
1. Placez votre fichier `.jar` dans le dossier `plugins/`
2. `docker-compose up -d`
3. Le plugin sera chargé au démarrage

### Méthode 2 : Copier manuellement dans le conteneur

```bash
# Conteneur en cours d'exécution
docker cp mon-plugin.jar neo4j_avec_apoc:/var/lib/neo4j/plugins/

# Redémarrer Neo4j pour charger le plugin
docker-compose restart
```

---

## 📊 Comparaison des plugins

| Plugin | Taille | Cas d'usage | Niveau |
|--------|--------|-------------|--------|
| **APOC** | ~25 Mo | Utilitaires généraux, indispensable | Débutant |
| **GDS** | ~150 Mo | Algorithmes de graphes, ML | Intermédiaire |
| **Neosemantics** | ~15 Mo | Données RDF/OWL, web sémantique | Avancé |
| **Streams** | ~40 Mo | Kafka, temps réel | Avancé |

---

## 🐛 Dépannage

### Le plugin ne se charge pas

**1. Vérifier les logs :**
```bash
docker-compose logs neo4j | grep -i apoc
```

**Erreurs courantes :**
- `Plugin not found` : Vérifiez l'orthographe dans `NEO4J_PLUGINS`
- `Download failed` : Problème de connexion Internet

**2. Vérifier le téléchargement :**
```bash
docker exec neo4j_avec_apoc ls -lh /var/lib/neo4j/plugins
```

Si le dossier est vide, le téléchargement a échoué.

**3. Téléchargement manuel (solution de secours) :**

```bash
# 1. Télécharger APOC depuis GitHub
wget https://github.com/neo4j/apoc/releases/download/5.13.0/apoc-5.13.0-core.jar

# 2. Copier dans le conteneur
docker cp apoc-5.13.0-core.jar neo4j_avec_apoc:/var/lib/neo4j/plugins/

# 3. Redémarrer
docker-compose restart
```

### Procédure non autorisée

**Erreur :**
```
There is no procedure with the name `apoc.create.node` registered
```

**Cause :** La ligne `NEO4J_dbms_security_procedures_unrestricted` est manquante ou incorrecte.

**Solution :**
```yaml
environment:
  NEO4J_dbms_security_procedures_unrestricted: 'apoc.*'
```

### Neo4j ne démarre pas après installation de GDS

**Cause :** Manque de mémoire.

**Solution :** Augmentez la heap Java :
```yaml
environment:
  NEO4J_server_memory_heap_max__size: '4G'  # Minimum 2G pour GDS
```

Vérifiez la mémoire disponible sur votre machine :
```bash
docker stats
```

---

## 🔒 Sécurité et bonnes pratiques

### ⚠️ Pour le développement

```yaml
# OK pour le dev
NEO4J_dbms_security_procedures_unrestricted: 'apoc.*, gds.*'
NEO4J_dbms_security_allow__csv__import__from__file__urls: 'true'
```

### ✅ Pour la production

```yaml
# Restreindre aux procédures nécessaires
NEO4J_dbms_security_procedures_unrestricted: 'apoc.load.json, apoc.create.uuid'

# Bloquer les imports arbitraires
NEO4J_dbms_security_allow__csv__import__from__file__urls: 'false'

# Activer l'authentification
NEO4J_AUTH: 'neo4j/UnMotDePasseTresTresSecurise123!'
```

---

## 📚 Ressources complémentaires

### Documentation officielle

- 📖 [APOC Documentation](https://neo4j.com/labs/apoc/5/)
- 📖 [GDS Documentation](https://neo4j.com/docs/graph-data-science/current/)
- 📖 [Neo4j Plugin API](https://neo4j.com/docs/java-reference/current/extending-neo4j/procedures-and-functions/)

### Tutoriels et exemples

- 🎓 [APOC Examples](https://neo4j.com/labs/apoc/5/overview/apoc.load/)
- 🎓 [GDS Tutorials](https://neo4j.com/docs/graph-data-science/current/tutorials/)
- 💻 [GitHub - Exemples APOC](https://github.com/neo4j/apoc)

### Développer son propre plugin

- 🛠️ [Neo4j Plugin Template](https://github.com/neo4j-examples/neo4j-procedure-template)
- 📝 [Writing Custom Procedures](https://neo4j.com/docs/java-reference/current/extending-neo4j/procedures/)

---

## ✅ Récapitulatif

Vous avez appris à :

✅ Comprendre l'utilité des plugins Neo4j
✅ Installer APOC via Docker Compose
✅ Configurer les autorisations de sécurité
✅ Vérifier l'installation des plugins
✅ Utiliser des procédures APOC courantes
✅ Installer Graph Data Science (GDS)
✅ Charger des plugins personnalisés
✅ Diagnostiquer les problèmes d'installation
✅ Appliquer les bonnes pratiques de sécurité

**Prochaine étape :**
- [Import de données CSV](04-import-donnees-csv.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
