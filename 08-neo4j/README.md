# Neo4j - Base de données orientée graphes

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que Neo4j ?

**Neo4j** est une base de données orientée **graphes** (Graph Database), leader mondial dans sa catégorie. Contrairement aux bases de données relationnelles qui stockent les données dans des tables, Neo4j représente les informations sous forme de **nœuds** (nodes) et de **relations** (relationships), formant ainsi un réseau interconnecté.

### 🌐 Le paradigme du graphe

Imaginons un réseau social simplifié :

```
(Alice)-[:CONNAIT]->(Bob)-[:TRAVAILLE_AVEC]->(Charlie)
   |                                              |
   +---------------[:MANAGE]----------------------+
```

Dans Neo4j, ce schéma se traduit naturellement :
- **Alice, Bob, Charlie** = Nœuds (nodes)
- **CONNAIT, TRAVAILLE_AVEC, MANAGE** = Relations (relationships)
- Chaque élément peut avoir des **propriétés** (nom, âge, date...)

---

## 🆚 Différence avec les bases relationnelles

### Base de données relationnelle (SQL)

```sql
-- Table : personnes
id | nom     | age
1  | Alice   | 30
2  | Bob     | 25
3  | Charlie | 35

-- Table : relations (jointure nécessaire)
id_source | id_cible | type
1         | 2        | CONNAIT
2         | 3        | TRAVAILLE_AVEC
1         | 3        | MANAGE

-- Requête : Qui sont les amis d'Alice ?
SELECT p2.nom
FROM personnes p1
JOIN relations r ON p1.id = r.id_source
JOIN personnes p2 ON r.id_cible = p2.id
WHERE p1.nom = 'Alice' AND r.type = 'CONNAIT';
```

**Problèmes :**
- ❌ Jointures multiples = lent
- ❌ Complexité croissante avec la profondeur
- ❌ Difficile de représenter des réseaux complexes

### Base de données graphe (Neo4j)

```cypher
// Création directe du graphe
CREATE (alice:Personne {nom: 'Alice', age: 30})
CREATE (bob:Personne {nom: 'Bob', age: 25})
CREATE (alice)-[:CONNAIT]->(bob)

// Requête : Qui sont les amis d'Alice ?
MATCH (alice:Personne {nom: 'Alice'})-[:CONNAIT]->(ami)
RETURN ami.nom
```

**Avantages :**
- ✅ Structure naturelle et intuitive
- ✅ Requêtes ultra-rapides sur les relations
- ✅ Profondeur illimitée sans perte de performance
- ✅ Modélisation proche de la réalité

---

## 🎯 Cas d'usage idéaux pour Neo4j

### 1️⃣ Réseaux sociaux

**Exemple :** Facebook, LinkedIn, Twitter

**Fonctionnalités :**
- Amis, followers, recommandations d'amis
- "Vous connaissez peut-être..."
- Influence et centralité dans le réseau

**Requête type :**
```cypher
// Amis d'amis (recommandations)
MATCH (moi:Utilisateur {id: 123})-[:AMI]->()-[:AMI]->(recommandation)
WHERE NOT (moi)-[:AMI]->(recommandation)
RETURN recommandation.nom
LIMIT 10
```

### 2️⃣ Systèmes de recommandation

**Exemple :** Amazon, Netflix, Spotify

**Fonctionnalités :**
- "Les clients qui ont acheté X ont aussi acheté Y"
- Recommandations basées sur les goûts similaires
- Découverte de contenus

**Requête type :**
```cypher
// Produits similaires
MATCH (p:Produit {id: 'laptop-123'})<-[:A_ACHETE]-(client)-[:A_ACHETE]->(autre:Produit)
RETURN autre.nom, COUNT(*) AS popularite
ORDER BY popularite DESC
LIMIT 5
```

### 3️⃣ Détection de fraude

**Exemple :** Banques, assurances, e-commerce

**Fonctionnalités :**
- Réseaux de transactions suspectes
- Détection de patterns anormaux
- Traçabilité des flux financiers

**Requête type :**
```cypher
// Trouver des comptes liés suspects
MATCH (compte:Compte {suspect: true})-[:TRANSFERT*1..3]-(autre:Compte)
WHERE autre.montant_total > 100000
RETURN autre
```

### 4️⃣ Gestion des connaissances (Knowledge Graphs)

**Exemple :** Wikipedia, Google Knowledge Graph

**Fonctionnalités :**
- Ontologies et taxonomies
- Relations sémantiques entre concepts
- Moteurs de recherche intelligents

**Requête type :**
```cypher
// Tous les concepts liés à "Intelligence Artificielle"
MATCH (ia:Concept {nom: 'Intelligence Artificielle'})-[r]-(concept)
RETURN concept.nom, type(r)
```

### 5️⃣ Logistique et routage

**Exemple :** GPS, livraisons, supply chain

**Fonctionnalités :**
- Calcul de chemin optimal
- Optimisation de tournées
- Gestion des dépendances

**Requête type :**
```cypher
// Plus court chemin entre deux villes
MATCH path = shortestPath(
  (paris:Ville {nom: 'Paris'})-[:ROUTE*]-(marseille:Ville {nom: 'Marseille'})
)
RETURN path, length(path) AS distance
```

### 6️⃣ Gestion d'identité et accès (IAM)

**Exemple :** Active Directory, systèmes de permissions

**Fonctionnalités :**
- Hiérarchies d'organisations
- Gestion des rôles et permissions
- Traçabilité des accès

**Requête type :**
```cypher
// Tous les accès d'un utilisateur
MATCH (user:User {id: 'alice'})-[:MEMBRE_DE*]->(group)-[:A_ACCES]->(resource)
RETURN DISTINCT resource.nom
```

---

## ⚡ Pourquoi Neo4j est rapide sur les relations

### Le secret : Index-Free Adjacency

**Dans une base SQL :**
```
Alice -> [Recherche dans table relations] -> [Jointure] -> Bob
Temps : O(log n) par relation
```

**Dans Neo4j :**
```
Alice -> [Pointeur direct] -> Bob
Temps : O(1) par relation
```

**Conséquence :**
- Traverser 1 relation : même temps
- Traverser 1000 relations : **Neo4j reste constant**, SQL explose

**Exemple concret :**
```cypher
// Trouver tous les amis jusqu'à 5 niveaux de profondeur
MATCH (alice:Personne {nom: 'Alice'})-[:CONNAIT*1..5]->(ami)
RETURN ami.nom
```

En SQL, cette requête nécessiterait 5 jointures auto-référentielles (cauchemar de performances) !

---

## 🏗️ Architecture de Neo4j

### Composants principaux

```
┌─────────────────────────────────────────┐
│          Neo4j Browser (Web UI)         │
│        http://localhost:7474            │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         Bolt Protocol (7687)            │
│    Protocole binaire pour drivers       │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         Neo4j Database Engine           │
│  - Query Engine (Cypher)                │
│  - Transaction Manager                  │
│  - Storage Engine                       │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         Persistance (Disque)            │
│  /data (graphes, index, transactions)   │
└─────────────────────────────────────────┘
```

### Ports utilisés

| Port | Protocole | Usage |
|------|-----------|-------|
| **7474** | HTTP | Neo4j Browser (interface web) |
| **7687** | Bolt | Connexions des applications (drivers) |
| 7473 | HTTPS | Neo4j Browser sécurisé (optionnel) |

---

## 💻 Cypher : Le langage de requête

**Cypher** est le langage de requête de Neo4j, conçu pour être **visuel** et **intuitif**.

### Philosophie : "ASCII Art"

```cypher
// Les parenthèses () représentent les nœuds
(alice:Personne {nom: 'Alice'})

// Les crochets [] représentent les relations
-[:CONNAIT]->

// Tout ensemble forme un pattern
(alice)-[:CONNAIT]->(bob)
```

### Commandes de base

| Opération | Commande Cypher | Équivalent SQL |
|-----------|-----------------|----------------|
| Créer | `CREATE` | `INSERT` |
| Lire | `MATCH` | `SELECT` |
| Mettre à jour | `SET` | `UPDATE` |
| Supprimer | `DELETE` | `DELETE` |
| Fusionner | `MERGE` | `INSERT ... ON CONFLICT` |

### Exemples de syntaxe

```cypher
// Créer un nœud
CREATE (p:Personne {nom: 'Alice', age: 30})

// Chercher un nœud
MATCH (p:Personne {nom: 'Alice'})
RETURN p

// Créer une relation
MATCH (a:Personne {nom: 'Alice'})
MATCH (b:Personne {nom: 'Bob'})
CREATE (a)-[:CONNAIT {depuis: 2020}]->(b)

// Requête complexe
MATCH (alice:Personne {nom: 'Alice'})-[:CONNAIT*1..3]-(ami)
WHERE ami.age > 25
RETURN ami.nom, ami.age
ORDER BY ami.age DESC
LIMIT 10
```

---

## 📦 Versions de Neo4j

### Community vs Enterprise

| Fonctionnalité | Community (Gratuit) | Enterprise (Payant) |
|----------------|---------------------|---------------------|
| **Base de données** | ✅ Complète | ✅ Complète |
| **Cypher** | ✅ Complet | ✅ Complet |
| **Neo4j Browser** | ✅ Oui | ✅ Oui |
| **APOC (plugins)** | ✅ Oui | ✅ Oui |
| **Clustering** | ❌ Non | ✅ Oui |
| **Réplication** | ❌ Non | ✅ Oui |
| **Hot backups** | ❌ Non | ✅ Oui |
| **Monitoring avancé** | ❌ Limité | ✅ Complet |
| **Support** | ❌ Communauté | ✅ Professionnel |

**💡 Pour le développement :** La version **Community** suffit amplement !

### Versions disponibles

- **Neo4j 5.x** : Version actuelle (recommandée)
- **Neo4j 4.x** : Ancienne version stable
- **Neo4j 3.x** : Legacy (déprécié)

---

## 🐳 Neo4j avec Docker : Avantages

### ✅ Pourquoi utiliser Docker pour Neo4j ?

| Avantage | Description |
|----------|-------------|
| **Installation rapide** | 2 minutes vs 30+ minutes en installation native |
| **Isolation** | Ne pollue pas votre système |
| **Reproductibilité** | Même environnement sur tous les postes |
| **Versioning facile** | Tester plusieurs versions en parallèle |
| **Nettoyage simple** | Suppression complète en une commande |
| **Développement** | Idéal pour prototypage et tests |

### 📋 Ce que vous trouverez dans ce dossier

```
08-neo4j/
├── README.md (ce fichier)
├── 01-config-basique-docker-compose.md
│   └── Configuration de base, premiers pas avec Cypher
├── 02-config-ip-fixe.md
│   └── Réseau Docker personnalisé et IP statique
├── 03-config-avec-plugins.md
│   └── APOC, Graph Data Science, plugins personnalisés
└── 04-import-donnees-csv.md
    └── Import de données CSV, optimisations
```

---

## 🎓 Ressources d'apprentissage

### Documentation officielle

- 📖 [Neo4j Documentation](https://neo4j.com/docs/)
- 📖 [Cypher Manual](https://neo4j.com/docs/cypher-manual/current/)
- 📖 [Docker Hub - Neo4j](https://hub.docker.com/_/neo4j)

### Formations gratuites

- 🎓 [Neo4j GraphAcademy](https://graphacademy.neo4j.com/) - Cours interactifs gratuits
- 🎓 [Neo4j Certification](https://graphacademy.neo4j.com/categories/certification/) - Certification gratuite

### Tutoriels et exemples

- 💻 [Neo4j Examples](https://neo4j.com/developer/example-data/) - Jeux de données
- 💻 [GitHub - Neo4j Examples](https://github.com/neo4j-graph-examples) - Projets complets
- 📺 [YouTube - Neo4j](https://www.youtube.com/neo4j) - Vidéos tutorielles

### Communauté

- 💬 [Neo4j Community Forum](https://community.neo4j.com/)
- 💬 [Stack Overflow - neo4j](https://stackoverflow.com/questions/tagged/neo4j)
- 💬 [Discord Neo4j](https://discord.gg/neo4j)

---

## 🚀 Démarrage rapide

Prêt à commencer ? Suivez les guides dans l'ordre :

1. **[Configuration basique](01-config-basique-docker-compose.md)** ⭐ Commencez ici
   - Installation en 5 minutes
   - Premiers pas avec Cypher
   - Interface Neo4j Browser

2. **[Configuration IP fixe](02-config-ip-fixe.md)**
   - Réseau Docker personnalisé
   - Communication inter-conteneurs
   - Configuration avancée

3. **[Plugins (APOC, GDS)](03-config-avec-plugins.md)**
   - APOC : 450+ fonctions utilitaires
   - Graph Data Science : Algorithmes ML
   - Plugins personnalisés

4. **[Import de données CSV](04-import-donnees-csv.md)**
   - LOAD CSV natif
   - APOC pour gros volumes
   - Optimisations et bonnes pratiques

---

## 💡 Neo4j en un coup d'œil

### Quand utiliser Neo4j ?

✅ **OUI si :**
- Vos données ont beaucoup de **relations interconnectées**
- Vous devez faire des **traversées de graphes** (amis d'amis, chemins)
- Vous avez besoin de **requêtes complexes sur les relations**
- Vous travaillez sur des **réseaux** (social, routage, supply chain)
- Vous faites de la **détection de patterns** ou **recommandations**

❌ **NON si :**
- Vos données sont tabulaires simples (préférez PostgreSQL/MySQL)
- Vous faites principalement des agrégations (préférez bases colonnes)
- Vous avez besoin de transactions distribuées complexes
- Vos requêtes sont principalement de type CRUD simple
- Vous stockez des documents JSON (préférez MongoDB)

### Tableau comparatif rapide

| Critère | Neo4j | PostgreSQL | MongoDB |
|---------|-------|------------|---------|
| **Structure** | Graphe | Tables | Documents |
| **Relations** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **Requêtes complexes** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Scalabilité horizontale** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| **Transactions ACID** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Courbe d'apprentissage** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

---

## 🎯 Objectifs après ce chapitre

Après avoir suivi les 4 fiches de ce dossier, vous serez capable de :

✅ Installer et configurer Neo4j avec Docker
✅ Utiliser Neo4j Browser et écrire des requêtes Cypher
✅ Créer des nœuds, relations et parcourir le graphe
✅ Configurer un réseau Docker avec IP fixe
✅ Installer et utiliser APOC et Graph Data Science
✅ Importer des données CSV (petits et gros volumes)
✅ Optimiser les performances d'import
✅ Diagnostiquer et résoudre les problèmes courants

**Temps estimé :** 2-3 heures pour parcourir l'ensemble des fiches.

---

## 🔗 Liens vers les autres bases de données

- ⬅️ [Cassandra](../07-cassandra/README.md) - Base NoSQL colonnes
- ➡️ [InfluxDB](../09-influxdb/README.md) - Base de séries temporelles

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
