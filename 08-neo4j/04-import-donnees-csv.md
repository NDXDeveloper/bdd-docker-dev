# 8.4 Import de données CSV

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

L'import de données CSV est l'une des **premières étapes** lorsqu'on travaille avec Neo4j. Que vous migriez depuis une base relationnelle, que vous ayez des exports Excel, ou des données issues d'API, le format CSV est universel et Neo4j offre plusieurs méthodes pour l'importer efficacement.

### 🤔 Pourquoi importer depuis des CSV ?

| Situation | Exemple |
|-----------|---------|
| **Migration de données** | Passer d'une base MySQL/PostgreSQL à Neo4j |
| **Import initial** | Charger un jeu de données de référence (villes, produits...) |
| **Intégration** | Importer des exports quotidiens depuis un ERP |
| **Prototypage** | Tester rapidement un modèle de graphe |
| **Données externes** | Charger des datasets publics (Kaggle, data.gouv.fr...) |

### 🎯 Ce que vous allez apprendre

1. Préparer des fichiers CSV pour Neo4j
2. Configurer Docker pour l'import de fichiers
3. Utiliser `LOAD CSV` (méthode native Neo4j)
4. Utiliser APOC pour des imports avancés
5. Gérer les erreurs et optimiser les performances

---

## 📊 Structure d'un CSV pour Neo4j

### Format recommandé

Un fichier CSV bien structuré pour Neo4j doit avoir :
- **Une ligne d'en-têtes** (headers) explicites
- **Un séparateur clair** (virgule `,` ou point-virgule `;`)
- **Des guillemets** pour les valeurs contenant le séparateur
- **Un encodage UTF-8** (pour les accents)

**Exemple : `personnes.csv`**
```csv
id,nom,age,ville
1,Alice,30,Paris
2,Bob,25,Lyon
3,Charlie,35,Marseille
```

**Exemple : `relations.csv`**
```csv
id_source,id_cible,type,depuis
1,2,CONNAIT,2020
2,3,TRAVAILLE_AVEC,2019
```

---

## 🚀 Configuration Docker pour l'import CSV

### Étape 1 : Créer le projet avec dossier d'import

```bash
mkdir neo4j-import-csv
cd neo4j-import-csv
mkdir import  # Dossier pour les fichiers CSV
```

### Étape 2 : Créer le fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.13
    container_name: neo4j_import_csv
    restart: unless-stopped

    ports:
      - "7474:7474"  # Neo4j Browser
      - "7687:7687"  # Bolt

    environment:
      NEO4J_AUTH: none
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

      # Activer APOC pour les imports avancés
      NEO4J_PLUGINS: '["apoc"]'
      NEO4J_dbms_security_procedures_unrestricted: 'apoc.*'

      # IMPORTANT : Autoriser l'import depuis des fichiers
      NEO4J_dbms_security_allow__csv__import__from__file__urls: 'true'

    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
      - neo4j_plugins:/plugins
      # Bind mount : partager le dossier local "import" avec le conteneur
      - ./import:/var/lib/neo4j/import

volumes:
  neo4j_data:
  neo4j_logs:
  neo4j_plugins:
```

### 🔍 Points clés de la configuration

#### **Volume import**
```yaml
- ./import:/var/lib/neo4j/import
```
- `./import` : Dossier local sur votre machine
- `/var/lib/neo4j/import` : Dossier dans le conteneur Neo4j
- Tout fichier placé dans `./import` sera accessible dans Neo4j

#### **Autorisation d'import**
```yaml
NEO4J_dbms_security_allow__csv__import__from__file__urls: 'true'
```
⚠️ **Important :** Sans cette ligne, Neo4j bloque les imports de fichiers locaux (sécurité).

### Étape 3 : Démarrer Neo4j

```bash
docker-compose up -d
```

Attendez 30 secondes pour le chargement d'APOC, puis ouvrez : `http://localhost:7474`

---

## 📥 Méthode 1 : Import avec `LOAD CSV` (natif Neo4j)

`LOAD CSV` est la commande **native** de Neo4j pour importer des CSV. Elle est idéale pour des fichiers de taille petite à moyenne (< 10 000 lignes).

### Exemple 1 : Import simple de nœuds

**Fichier : `import/personnes.csv`**
```csv
id,nom,age,ville
1,Alice,30,Paris
2,Bob,25,Lyon
3,Charlie,35,Marseille
4,Diana,28,Toulouse
```

**Requête Cypher :**
```cypher
// Charger le CSV et créer des nœuds Personne
LOAD CSV WITH HEADERS FROM 'file:///personnes.csv' AS row
CREATE (p:Personne {
  id: toInteger(row.id),
  nom: row.nom,
  age: toInteger(row.age),
  ville: row.ville
})
RETURN p
```

**Explication ligne par ligne :**

| Élément | Signification |
|---------|---------------|
| `LOAD CSV WITH HEADERS` | Charge le CSV en utilisant la première ligne comme en-têtes |
| `FROM 'file:///personnes.csv'` | Chemin du fichier (3 slashes = chemin absolu dans `/import`) |
| `AS row` | Chaque ligne devient une variable `row` |
| `toInteger(row.id)` | Convertit le texte en nombre entier |
| `CREATE (p:Personne {...})` | Crée un nœud avec les données de la ligne |

**Résultat :**
```
Added 4 labels, created 4 nodes, set 16 properties, completed after X ms.
```

### Exemple 2 : Import de relations

**Fichier : `import/relations.csv`**
```csv
id_source,id_cible,type_relation,depuis
1,2,CONNAIT,2020-01-15
2,3,TRAVAILLE_AVEC,2019-06-01
3,4,CONNAIT,2021-03-10
1,4,MANAGE,2022-08-20
```

**Requête Cypher :**
```cypher
// Charger les relations entre personnes existantes
LOAD CSV WITH HEADERS FROM 'file:///relations.csv' AS row
MATCH (source:Personne {id: toInteger(row.id_source)})
MATCH (cible:Personne {id: toInteger(row.id_cible)})
CREATE (source)-[r:CONNAIT {depuis: row.depuis}]->(cible)
RETURN source.nom, cible.nom, r.depuis
```

**Explication :**
1. `MATCH` : Trouve les nœuds source et cible déjà créés
2. `CREATE (source)-[r:CONNAIT]-(cible)` : Crée la relation
3. `{depuis: row.depuis}` : Ajoute une propriété à la relation

⚠️ **Note :** Cette requête crée toutes les relations avec le type `CONNAIT`. Pour gérer plusieurs types, voir l'exemple suivant.

### Exemple 3 : Relations avec types dynamiques

Si votre CSV contient différents types de relations :

```cypher
LOAD CSV WITH HEADERS FROM 'file:///relations.csv' AS row
MATCH (source:Personne {id: toInteger(row.id_source)})
MATCH (cible:Personne {id: toInteger(row.id_cible)})
CALL apoc.create.relationship(source, row.type_relation, {depuis: row.depuis}, cible)
YIELD rel
RETURN source.nom, type(rel), cible.nom
```

**Résultat :**
```
╒═════════════╤══════════════════╤═════════════╕
│ source.nom  │ type(rel)        │ cible.nom   │
╞═════════════╪══════════════════╪═════════════╡
│ "Alice"     │ "CONNAIT"        │ "Bob"       │
│ "Bob"       │ "TRAVAILLE_AVEC" │ "Charlie"   │
│ "Charlie"   │ "CONNAIT"        │ "Diana"     │
│ "Alice"     │ "MANAGE"         │ "Diana"     │
└─────────────┴──────────────────┴─────────────┘
```

---

## 🚀 Méthode 2 : Import avec APOC (pour gros volumes)

APOC offre des procédures optimisées pour l'import de **gros fichiers CSV** (> 10 000 lignes).

### Avantages d'APOC

| Fonctionnalité | LOAD CSV | APOC |
|----------------|----------|------|
| **Gros volumes** | ❌ Lent (> 10k lignes) | ✅ Optimisé (millions de lignes) |
| **Transactions par batch** | ❌ Non | ✅ Oui (configurable) |
| **Import parallèle** | ❌ Non | ✅ Oui |
| **URLs directes** | ⚠️ Limité | ✅ HTTP, HTTPS, FTP |
| **Gestion d'erreurs** | ⚠️ Basique | ✅ Avancée |

### Exemple 1 : Import simple avec APOC

**Fichier : `import/produits.csv`**
```csv
sku,nom,prix,categorie
P001,Laptop,999.99,Informatique
P002,Souris,29.99,Accessoires
P003,Clavier,79.99,Accessoires
P004,Écran,299.99,Informatique
```

**Requête avec `apoc.load.csv` :**
```cypher
CALL apoc.load.csv('file:///produits.csv', {
  header: true,
  sep: ',',
  ignore: []
}) YIELD map AS row
CREATE (p:Produit {
  sku: row.sku,
  nom: row.nom,
  prix: toFloat(row.prix),
  categorie: row.categorie
})
RETURN p
```

**Paramètres :**
- `header: true` : Utilise la première ligne comme en-têtes
- `sep: ','` : Séparateur (virgule)
- `ignore: []` : Liste de colonnes à ignorer (aucune ici)

### Exemple 2 : Import par batch (performances)

Pour importer **100 000 lignes ou plus**, utilisez le mode batch :

```cypher
// Import par lots de 1000 lignes
CALL apoc.periodic.iterate(
  // Requête 1 : Charger les données
  "CALL apoc.load.csv('file:///gros_fichier.csv', {header: true}) YIELD map RETURN map",

  // Requête 2 : Créer les nœuds
  "CREATE (p:Personne {
    id: toInteger(map.id),
    nom: map.nom,
    age: toInteger(map.age)
  })",

  // Configuration
  {batchSize: 1000, parallel: false}
)
YIELD batches, total, timeTaken
RETURN batches, total, timeTaken
```

**Résultat :**
```
╒═════════╤═══════╤════════════╕
│ batches │ total │ timeTaken  │
╞═════════╪═══════╪════════════╡
│ 100     │ 100000│ 45         │ (secondes)
└─────────┴───────┴────────────┘
```

**Paramètres :**
- `batchSize: 1000` : Traite 1000 lignes par transaction
- `parallel: false` : Import séquentiel (mettre `true` si plusieurs CPU)

---

## 🌐 Import depuis une URL

Neo4j peut charger des CSV directement depuis Internet.

### Exemple : Dataset public

```cypher
// Charger des données depuis GitHub
LOAD CSV WITH HEADERS
FROM 'https://raw.githubusercontent.com/neo4j-graph-examples/movies/main/data/movies.csv'
AS row
MERGE (m:Film {id: row.movieId})
SET m.titre = row.title, m.annee = toInteger(row.year)
RETURN m
LIMIT 10
```

**Avec APOC (plus performant) :**
```cypher
CALL apoc.load.csv(
  'https://raw.githubusercontent.com/neo4j-graph-examples/movies/main/data/movies.csv',
  {header: true}
) YIELD map
MERGE (m:Film {id: map.movieId})
SET m.titre = map.title, m.annee = toInteger(map.year)
RETURN m
LIMIT 10
```

---

## 🛠️ Bonnes pratiques d'import

### 1️⃣ Utiliser `MERGE` au lieu de `CREATE`

```cypher
// ❌ Mauvais : Crée des doublons si on relance la requête
CREATE (p:Personne {id: toInteger(row.id), nom: row.nom})

// ✅ Bon : Crée uniquement si le nœud n'existe pas
MERGE (p:Personne {id: toInteger(row.id)})
SET p.nom = row.nom, p.age = toInteger(row.age)
```

### 2️⃣ Créer des index AVANT l'import

Les index accélèrent les `MATCH` et `MERGE` :

```cypher
// Créer un index sur l'ID des personnes
CREATE INDEX personne_id FOR (p:Personne) ON (p.id)

// Créer un index composite
CREATE INDEX personne_nom_ville FOR (p:Personne) ON (p.nom, p.ville)
```

Ensuite, importez vos données. La différence de performance peut être énorme (x10 à x100).

### 3️⃣ Séparer nœuds et relations

**Ordre recommandé :**
1. Importer tous les **nœuds** d'abord
2. Créer les **index**
3. Importer les **relations**

**Pourquoi ?**
- Les relations nécessitent que les nœuds existent (`MATCH`)
- Les index accélèrent la recherche des nœuds

### 4️⃣ Gérer les valeurs nulles

```cypher
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
CREATE (p:Personne {
  id: toInteger(row.id),
  nom: row.nom,
  // Si age est vide, utiliser null au lieu de convertir
  age: CASE WHEN row.age IS NULL OR row.age = ''
            THEN null
            ELSE toInteger(row.age)
       END
})
```

### 5️⃣ Nettoyer les espaces

```cypher
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
CREATE (p:Personne {
  nom: trim(row.nom),  // Supprime les espaces avant/après
  ville: trim(row.ville)
})
```

---

## 🔧 Gestion des formats de dates

### Format ISO (recommandé)

**CSV avec dates ISO :**
```csv
id,nom,date_naissance
1,Alice,1995-06-15
2,Bob,1998-12-03
```

**Import :**
```cypher
LOAD CSV WITH HEADERS FROM 'file:///dates.csv' AS row
CREATE (p:Personne {
  id: toInteger(row.id),
  nom: row.nom,
  dateNaissance: date(row.date_naissance)
})
```

### Format personnalisé avec APOC

**CSV avec format français :**
```csv
id,nom,date_naissance
1,Alice,15/06/1995
2,Bob,03/12/1998
```

**Import avec conversion :**
```cypher
LOAD CSV WITH HEADERS FROM 'file:///dates.csv' AS row
CREATE (p:Personne {
  id: toInteger(row.id),
  nom: row.nom,
  dateNaissance: apoc.date.parse(row.date_naissance, 'ms', 'dd/MM/yyyy')
})
```

---

## 📂 Structure de projet complète

### Organisation recommandée

```
neo4j-import-csv/
├── docker-compose.yml
├── import/
│   ├── 01-personnes.csv
│   ├── 02-entreprises.csv
│   ├── 03-relations-travail.csv
│   └── 04-relations-amis.csv
└── scripts/
    ├── 01-create-indexes.cypher
    ├── 02-import-personnes.cypher
    ├── 03-import-entreprises.cypher
    └── 04-import-relations.cypher
```

### Script d'import complet

**`scripts/import-all.cypher`**
```cypher
// Étape 1 : Créer les index
CREATE INDEX personne_id IF NOT EXISTS FOR (p:Personne) ON (p.id);
CREATE INDEX entreprise_id IF NOT EXISTS FOR (e:Entreprise) ON (e.id);

// Étape 2 : Importer les personnes
LOAD CSV WITH HEADERS FROM 'file:///01-personnes.csv' AS row
MERGE (p:Personne {id: toInteger(row.id)})
SET p.nom = row.nom, p.age = toInteger(row.age);

// Étape 3 : Importer les entreprises
LOAD CSV WITH HEADERS FROM 'file:///02-entreprises.csv' AS row
MERGE (e:Entreprise {id: toInteger(row.id)})
SET e.nom = row.nom, e.secteur = row.secteur;

// Étape 4 : Importer les relations
LOAD CSV WITH HEADERS FROM 'file:///03-relations-travail.csv' AS row
MATCH (p:Personne {id: toInteger(row.id_personne)})
MATCH (e:Entreprise {id: toInteger(row.id_entreprise)})
MERGE (p)-[r:TRAVAILLE_POUR]->(e)
SET r.depuis = row.annee_debut;
```

### Exécuter le script depuis Docker

```bash
# Copier le script dans le conteneur
docker cp scripts/import-all.cypher neo4j_import_csv:/var/lib/neo4j/

# Exécuter le script
docker exec -it neo4j_import_csv cypher-shell -f /var/lib/neo4j/import-all.cypher
```

---

## 🐛 Dépannage

### Erreur : "Couldn't load the external resource"

**Cause :** Neo4j ne trouve pas le fichier CSV.

**Solutions :**

1. **Vérifier que le fichier existe :**
   ```bash
   docker exec neo4j_import_csv ls -lh /var/lib/neo4j/import
   ```

2. **Vérifier le chemin :**
   ```cypher
   // ❌ Mauvais
   FROM 'file://personnes.csv'

   // ✅ Bon (3 slashes)
   FROM 'file:///personnes.csv'
   ```

3. **Vérifier les permissions :**
   ```bash
   chmod 644 import/personnes.csv
   ```

### Erreur : "Type mismatch"

**Cause :** Tentative de conversion d'une valeur invalide.

**Exemple :**
```csv
id,age
1,30
2,inconnu  <- Problème !
```

**Solution :** Gérer les valeurs manquantes :
```cypher
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
CREATE (p:Personne {
  id: toInteger(row.id),
  age: CASE
         WHEN row.age IS NULL OR row.age = '' OR row.age = 'inconnu'
         THEN null
         ELSE toInteger(row.age)
       END
})
```

### Import très lent

**Causes possibles :**

1. **Pas d'index :** Créez des index avant l'import
2. **Fichier trop gros :** Utilisez APOC avec batches
3. **Trop de `MATCH` :** Importez d'abord tous les nœuds, puis les relations

**Optimisation :**
```cypher
// Au lieu de ça (lent) :
LOAD CSV WITH HEADERS FROM 'file:///relations.csv' AS row
MATCH (a:Personne {id: toInteger(row.id_a)})
MATCH (b:Personne {id: toInteger(row.id_b)})
CREATE (a)-[:CONNAIT]->(b)

// Faites ça (rapide avec APOC) :
CALL apoc.periodic.iterate(
  "LOAD CSV WITH HEADERS FROM 'file:///relations.csv' AS row RETURN row",
  "MATCH (a:Personne {id: toInteger(row.id_a)})
   MATCH (b:Personne {id: toInteger(row.id_b)})
   CREATE (a)-[:CONNAIT]->(b)",
  {batchSize: 1000, parallel: false}
)
```

### Caractères accentués illisibles

**Cause :** Encodage incorrect du CSV.

**Solution :**

1. **Sauvegarder le CSV en UTF-8** (depuis Excel : "CSV UTF-8")
2. **Forcer l'encodage avec APOC :**
   ```cypher
   CALL apoc.load.csv('file:///data.csv', {
     header: true,
     charset: 'UTF-8'
   })
   ```

---

## 📊 Tableau récapitulatif des méthodes

| Méthode | Fichiers | Taille max | Complexité | Performances |
|---------|----------|-----------|------------|--------------|
| **LOAD CSV** | Locaux, URLs | < 10 000 lignes | ⭐ Simple | ⭐⭐ Moyen |
| **APOC load.csv** | Locaux, URLs, FTP | Illimité | ⭐⭐ Moyen | ⭐⭐⭐ Bon |
| **APOC periodic.iterate** | Tous | Millions | ⭐⭐⭐ Avancé | ⭐⭐⭐⭐ Excellent |
| **neo4j-admin import** | Locaux | Milliards | ⭐⭐⭐⭐ Expert | ⭐⭐⭐⭐⭐ Exceptionnel |

💡 **Recommandation :**
- **< 10 000 lignes** : `LOAD CSV`
- **10 000 - 1 million** : `APOC load.csv`
- **> 1 million** : `APOC periodic.iterate` avec batches
- **> 100 millions** : `neo4j-admin import` (hors Docker, import offline)

---

## ✅ Récapitulatif

Vous avez appris à :

✅ Préparer des fichiers CSV pour Neo4j
✅ Configurer Docker pour partager des fichiers CSV
✅ Importer des nœuds avec `LOAD CSV`
✅ Importer des relations entre nœuds
✅ Utiliser APOC pour des imports optimisés
✅ Importer depuis des URLs
✅ Gérer les types de données (dates, nombres, nulls)
✅ Optimiser les performances (index, batches)
✅ Diagnostiquer et corriger les erreurs courantes
✅ Organiser un projet d'import complexe

**Prochaines étapes :**
- Explorer les [Cas pratiques](../../cas-pratiques/)
- Approfondir avec [APOC Documentation](https://neo4j.com/labs/apoc/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
