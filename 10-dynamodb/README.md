# 10. DynamoDB Local

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que DynamoDB ?

**DynamoDB** est une base de données NoSQL **entièrement gérée** créée par Amazon Web Services (AWS). C'est l'une des bases de données les plus populaires pour les applications cloud modernes.

### 🎯 Caractéristiques principales

| Caractéristique | Description |
|----------------|-------------|
| **Type** | NoSQL orientée documents (clé-valeur) |
| **Modèle** | Schéma flexible (comme MongoDB) |
| **Performance** | Latence de quelques millisecondes |
| **Scalabilité** | Automatique et illimitée |
| **Disponibilité** | 99.99% SLA (AWS) |
| **Facturation** | Pay-per-use (AWS) ou gratuit (Local) |

### 🌟 Points forts

✅ **Ultra-rapide** : Temps de réponse en millisecondes
✅ **Scalable** : De quelques requêtes à des millions par seconde
✅ **Serverless** : Pas de serveur à gérer (sur AWS)
✅ **Flexible** : Schéma dynamique, ajout d'attributs à la volée
✅ **Fiable** : Réplication automatique multi-zones (AWS)
✅ **Compatible** : SDK pour tous les langages (Python, JavaScript, Java, etc.)

### ⚠️ Limitations

❌ **Requêtes limitées** : Pas de jointures SQL complexes
❌ **Coût élevé** : Sur AWS, peut être cher avec beaucoup de données
❌ **Courbe d'apprentissage** : Modélisation différente du SQL
❌ **Pas de SQL standard** : Syntaxe propriétaire AWS

---

## 🔄 DynamoDB Local vs DynamoDB AWS

### Qu'est-ce que DynamoDB Local ?

**DynamoDB Local** est une version **gratuite** de DynamoDB que vous pouvez exécuter sur votre ordinateur. C'est l'outil parfait pour :

- 🧪 **Développer** sans frais AWS
- 📚 **Apprendre** DynamoDB gratuitement
- 🚀 **Tester** vos applications localement
- 🔬 **Expérimenter** sans risque

### Comparaison

| Fonctionnalité | DynamoDB Local | DynamoDB AWS |
|---------------|----------------|--------------|
| **Coût** | ✅ Gratuit | 💰 Payant |
| **Installation** | Docker (facile) | Aucune (cloud) |
| **Performance** | Limitée (PC) | ⚡ Ultra-rapide |
| **Scalabilité** | ❌ Non | ✅ Automatique |
| **Disponibilité** | Locale uniquement | 🌍 Mondiale |
| **Fonctionnalités** | ~90% compatibles | 100% |
| **Persistance** | Via volumes Docker | ✅ Garantie |
| **Backups** | Manuels | ✅ Automatiques |
| **Monitoring** | ❌ Non | ✅ CloudWatch |
| **Sécurité** | ❌ Aucune | ✅ IAM, VPC |

### 🎯 Quand utiliser quoi ?

**Utilisez DynamoDB Local pour :**
- ✅ Développement local
- ✅ Tests unitaires et d'intégration
- ✅ Apprentissage et prototypage
- ✅ CI/CD (tests automatisés)
- ✅ Démonstrations sans connexion Internet

**Utilisez DynamoDB AWS pour :**
- ✅ Production
- ✅ Applications en ligne
- ✅ Haute disponibilité requise
- ✅ Scalabilité automatique nécessaire
- ✅ Réplication multi-régions

---

## 🗂️ Concepts fondamentaux de DynamoDB

### 1. Tables

Une **table** est l'équivalent d'une collection MongoDB ou d'une table SQL. C'est le conteneur principal de vos données.

**Exemple :** Table `users` qui contient tous vos utilisateurs.

### 2. Items (Éléments)

Un **item** est l'équivalent d'un document MongoDB ou d'une ligne SQL. C'est un objet JSON avec des attributs.

**Exemple d'item :**
```json
{
  "userId": "user001",
  "name": "Alice Dupont",
  "email": "alice@example.com",
  "age": 28,
  "premium": true
}
```

### 3. Attributs

Les **attributs** sont les champs d'un item (comme les colonnes SQL).

**Important :** DynamoDB est **schemaless** ! Chaque item peut avoir des attributs différents :

```json
// Item 1
{
  "userId": "user001",
  "name": "Alice",
  "age": 28
}

// Item 2 (attributs différents, c'est OK !)
{
  "userId": "user002",
  "name": "Bob",
  "city": "Paris",
  "hobbies": ["sport", "lecture"]
}
```

### 4. Types de données

DynamoDB supporte plusieurs types :

| Type | Symbole | Exemple | Description |
|------|---------|---------|-------------|
| **String** | S | `"Alice"` | Chaîne de caractères |
| **Number** | N | `28` | Nombre (entier ou décimal) |
| **Boolean** | BOOL | `true` / `false` | Booléen |
| **Null** | NULL | `null` | Valeur nulle |
| **Binary** | B | `[base64]` | Données binaires |
| **List** | L | `[1, "a", true]` | Tableau |
| **Map** | M | `{"key": "value"}` | Objet imbriqué |
| **String Set** | SS | `["a", "b"]` | Ensemble de chaînes |
| **Number Set** | NS | `[1, 2, 3]` | Ensemble de nombres |

### 5. Clés primaires (Primary Keys)

**Chaque item DOIT avoir une clé primaire unique.** Il existe deux types :

#### A. Clé simple (Hash Key / Partition Key)

Un seul attribut identifie l'item de manière unique.

```
userId (Hash Key)
```

**Exemple :**
```json
{
  "userId": "user001",  // ← Clé primaire
  "name": "Alice"
}
```

#### B. Clé composée (Hash Key + Range Key)

Deux attributs identifient l'item :
- **Hash Key** (Partition Key) : Groupe les items
- **Range Key** (Sort Key) : Trie les items dans un groupe

```
customerId (Hash Key) + orderDate (Range Key)
```

**Exemple :**
```json
{
  "customerId": "user001",  // ← Hash Key
  "orderDate": "2025-11-01", // ← Range Key
  "amount": 149.99
}
```

**Avantage :** Permet de requêter efficacement tous les items d'un groupe.

**Exemple de requête :** "Toutes les commandes du client user001"

### 6. Index secondaires

Les **index** permettent de requêter sur d'autres attributs que la clé primaire.

#### Local Secondary Index (LSI)

- Même Hash Key que la table principale
- Range Key différent
- Créé lors de la création de la table (ne peut pas être ajouté après)

#### Global Secondary Index (GSI)

- Hash Key et Range Key complètement différents
- Peut être ajouté après la création de la table
- Table "virtuelle" avec ses propres capacités

**Exemple :** Table `users` avec clé primaire `userId`, GSI sur `email`.

---

## 🔍 Opérations principales

### 1. Scan

**Lit TOUS les items d'une table.**

```
Scan → Lit toute la table → Filtre optionnel → Retourne les résultats
```

**Performance :** ❌ Lent (lit toute la table)
**Coût (AWS) :** 💰💰💰 Élevé
**Usage :** Dev/Debug uniquement

**Exemple :** "Donne-moi tous les utilisateurs"

### 2. Query

**Recherche des items basés sur la clé primaire.**

```
Query → Utilise la clé primaire → Retourne les résultats
```

**Performance :** ✅ Rapide (index optimisé)
**Coût (AWS) :** 💰 Faible
**Usage :** Production recommandée

**Exemple :** "Donne-moi l'utilisateur avec userId = user001"

### 3. Get Item

**Récupère UN item spécifique** en utilisant la clé primaire complète.

**Performance :** ⚡ Très rapide
**Coût (AWS) :** 💰 Minimal

### 4. Put Item

**Insère ou remplace** un item complet.

**Note :** Si l'item existe, il est **écrasé** (pas de merge).

### 5. Update Item

**Met à jour des attributs spécifiques** sans remplacer l'item entier.

**Avantage :** Plus efficace que Put si vous ne modifiez que quelques attributs.

### 6. Delete Item

**Supprime un item** basé sur sa clé primaire.

---

## 📊 Modélisation de données

### Différence avec SQL

| SQL (Relationnel) | DynamoDB (NoSQL) |
|-------------------|------------------|
| Normalisation (3NF) | Dénormalisation |
| Jointures | Pas de jointures |
| Tables multiples liées | Souvent une seule table |
| Requêtes flexibles | Requêtes optimisées à l'avance |

### Principe clé : Design pour vos requêtes

**En SQL :** Vous modélisez vos données, puis écrivez des requêtes.
**En DynamoDB :** Vous identifiez vos requêtes, puis modélisez vos données.

### Exemple : Blog

**Mauvais (pensée SQL) :**
```
Table users : userId, name, email
Table posts : postId, userId, title, content
Table comments : commentId, postId, userId, text
```
→ Nécessite 3 scans ou imports depuis l'application = lent

**Bon (pensée NoSQL) :**
```
Table unique : blogData

Item 1 (User)
PK: USER#user001
SK: PROFILE
name: Alice
email: alice@example.com

Item 2 (Post)
PK: USER#user001
SK: POST#2025-11-01#post001
title: Mon article
content: ...

Item 3 (Comment)
PK: POST#post001
SK: COMMENT#2025-11-02#comment001
author: Bob
text: Super article !
```

**Avantage :** Une seule Query retourne tous les posts d'un utilisateur !

---

## 🛠️ Outils et écosystème

### 1. AWS CLI

**Ligne de commande officielle** pour interagir avec DynamoDB.

```bash
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

**Avantages :**
- ✅ Automatisation facile (scripts)
- ✅ Compatible Local et AWS
- ✅ Toutes les fonctionnalités disponibles

### 2. Admin UI (dynamodb-admin)

**Interface graphique web** pour gérer DynamoDB Local.

```
http://localhost:8001
```

**Avantages :**
- ✅ Visuel et intuitif
- ✅ Parfait pour débuter
- ✅ Création de tables en quelques clics

### 3. NoSQL Workbench

**Outil officiel AWS** (application desktop).

**Avantages :**
- ✅ Modélisation visuelle
- ✅ Génération de code
- ✅ Compatible Local et AWS

**Inconvénient :**
- ❌ Installation lourde

### 4. SDK (Software Development Kits)

Bibliothèques pour tous les langages :

**JavaScript/Node.js :**
```javascript
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({
  endpoint: 'http://localhost:8000'
});
```

**Python (boto3) :**
```python
import boto3
dynamodb = boto3.client(
    'dynamodb',
    endpoint_url='http://localhost:8000'
)
```

**Java, .NET, PHP, Ruby, Go...** : Tous supportés !

---

## 🚀 Cas d'usage typiques

### 1. Applications web/mobile

**Exemple :** Application de réseau social

- Table `users` : Profils utilisateurs
- Table `posts` : Publications
- Table `sessions` : Sessions utilisateurs (avec TTL)

**Avantage :** Latence ultra-faible, scalabilité automatique.

### 2. Gaming

**Exemple :** Jeu en ligne multijoueur

- Table `players` : Statistiques des joueurs
- Table `leaderboards` : Classements
- Table `game-state` : État des parties en cours

**Avantage :** Performance élevée pour les lectures/écritures fréquentes.

### 3. IoT (Internet of Things)

**Exemple :** Capteurs connectés

- Table `sensor-data` : Données de capteurs
- Clé composée : `deviceId` + `timestamp`

**Avantage :** Gère des millions d'écritures par seconde.

### 4. E-commerce

**Exemple :** Boutique en ligne

- Table `products` : Catalogue
- Table `carts` : Paniers d'achat
- Table `orders` : Commandes

**Avantage :** Sessions utilisateurs ultra-rapides.

### 5. Cache/Session store

**Exemple :** Cache applicatif

- Alternative à Redis pour du cache persistant
- TTL automatique pour expiration

**Avantage :** Persistance garantie + scalabilité.

---

## 📈 Avantages de DynamoDB

### Pour les développeurs

✅ **Pas de gestion de serveur** (serverless)
✅ **Schéma flexible** (ajout d'attributs à la volée)
✅ **SDK dans tous les langages**
✅ **Intégration AWS** (Lambda, API Gateway, etc.)
✅ **Local pour développer** sans frais

### Pour les applications

✅ **Performance prévisible** (latence constante)
✅ **Scalabilité illimitée** (des milliers aux millions de requêtes)
✅ **Haute disponibilité** (99.99% SLA)
✅ **Backups automatiques** (AWS)
✅ **Réplication multi-régions** (Global Tables)

### Pour les entreprises

✅ **Pay-per-use** (pas de serveur inactif)
✅ **Conformité** (SOC, ISO, PCI-DSS)
✅ **Monitoring intégré** (CloudWatch)
✅ **Sécurité** (chiffrement, IAM)

---

## ⚠️ Quand NE PAS utiliser DynamoDB

### Cas inadaptés

❌ **Requêtes ad-hoc complexes** (multiples filtres arbitraires)
→ Utilisez Elasticsearch ou une BDD SQL

❌ **Jointures complexes** entre plusieurs tables
→ Utilisez PostgreSQL ou MySQL

❌ **Analytics/BI** (agrégations, GROUP BY)
→ Utilisez Redshift, BigQuery ou ClickHouse

❌ **Transactions ACID complexes** multi-tables
→ Utilisez PostgreSQL avec support transactionnel

❌ **Budget serré avec gros volume de données**
→ DynamoDB AWS peut être coûteux

❌ **Besoin de SQL standard**
→ Utilisez une BDD relationnelle

---

## 🎓 Concepts à maîtriser

Avant de commencer les fiches pratiques, assurez-vous de comprendre :

### Niveau débutant

- [ ] Qu'est-ce qu'une base NoSQL ?
- [ ] Différence entre DynamoDB Local et AWS
- [ ] Qu'est-ce qu'une table, un item, un attribut ?
- [ ] Les types de données (String, Number, Boolean, etc.)
- [ ] Qu'est-ce qu'une clé primaire (Hash Key) ?

### Niveau intermédiaire

- [ ] Clé composée (Hash + Range Key)
- [ ] Différence entre Scan et Query
- [ ] Comment modéliser sans jointures
- [ ] Index secondaires (LSI et GSI)
- [ ] Opérations CRUD (Create, Read, Update, Delete)

### Niveau avancé

- [ ] Stratégies de modélisation NoSQL
- [ ] Optimisation des coûts (AWS)
- [ ] Transactions DynamoDB
- [ ] Streams DynamoDB (événements)
- [ ] Global Tables (réplication multi-régions)

---

## 📚 Structure de ce module

Ce module contient **4 fiches pratiques** progressives :

### 🟢 Fiches débutant

**[10.1 - Configuration basique avec docker-compose](01-config-basique-docker-compose.md)**
- Installation de DynamoDB Local
- Premier démarrage
- Vérifications de base
- 📊 Difficulté : ⭐☆☆☆☆

### 🟡 Fiches intermédiaire

**[10.2 - Configuration avec IP fixe](02-config-ip-fixe.md)**
- Création de réseaux Docker personnalisés
- Attribution d'IP statiques
- Communication inter-conteneurs
- 📊 Difficulté : ⭐⭐☆☆☆

**[10.3 - DynamoDB avec Admin UI](03-dynamodb-admin-ui.md)**
- Installation de dynamodb-admin
- Interface graphique web
- Gestion visuelle des tables
- 📊 Difficulté : ⭐⭐☆☆☆

### 🔴 Fiches avancé

**[10.4 - Utilisation avec AWS CLI](04-utilisation-aws-cli.md)**
- Installation et configuration d'AWS CLI
- Toutes les opérations en ligne de commande
- Scripts d'automatisation
- Migration vers AWS
- 📊 Difficulté : ⭐⭐⭐☆☆

---

## 🎯 Parcours d'apprentissage recommandé

### Pour les débutants complets

1. ✅ Lisez cette introduction
2. ✅ **[Fiche 10.1](01-config-basique-docker-compose.md)** - Démarrez DynamoDB Local
3. ✅ **[Fiche 10.3](03-dynamodb-admin-ui.md)** - Explorez avec l'interface graphique
4. ⏭️ **[Fiche 10.4](04-utilisation-aws-cli.md)** - Apprenez AWS CLI (optionnel)

### Pour les développeurs confirmés

1. ✅ **[Fiche 10.1](01-config-basique-docker-compose.md)** - Setup rapide
2. ✅ **[Fiche 10.4](04-utilisation-aws-cli.md)** - Maîtrisez AWS CLI
3. ⏭️ **[Fiche 10.2](02-config-ip-fixe.md)** - Si besoin de microservices

### Pour la production

1. ✅ Testez sur DynamoDB Local (fiches 10.1 à 10.4)
2. ✅ Lisez la [documentation AWS officielle](https://docs.aws.amazon.com/dynamodb/)
3. ✅ Consultez l'[Annexe D - Sécurité](../annexes/D-securite-bonnes-pratiques.md)
4. ✅ Suivez le [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

## 💡 Conseils avant de commencer

### 1. Oubliez (temporairement) le SQL

DynamoDB n'est **PAS** du SQL. Les concepts sont différents :
- Pas de `SELECT * FROM table WHERE column = value`
- Pas de jointures
- Pensez "accès par clé" plutôt que "recherche flexible"

### 2. Commencez simple

Ne cherchez pas à tout comprendre tout de suite :
- ✅ Commencez avec une table simple (Hash Key seule)
- ✅ Insérez quelques items avec Admin UI
- ✅ Lisez-les avec AWS CLI
- ⏭️ Complexifiez progressivement

### 3. Pratiquez localement

DynamoDB Local est **gratuit** :
- ✅ Créez, supprimez, recréez autant que vous voulez
- ✅ Expérimentez sans crainte
- ✅ Cassez des choses et apprenez

### 4. Documentez votre modèle

DynamoDB n'a pas de schéma imposé, mais **vous** devez en avoir un :
- ✅ Notez vos structures de clés
- ✅ Documentez vos patterns d'accès
- ✅ Listez vos index secondaires

### 5. Pensez "requêtes d'abord"

**Avant de modéliser :**
1. Listez **toutes** vos requêtes applicatives
2. Identifiez les accès fréquents (hot paths)
3. Modélisez pour optimiser ces accès
4. Ajoutez des index pour les requêtes secondaires

---

## 📖 Ressources complémentaires

### Documentation officielle

- 📖 [AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)
- 📖 [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- 📖 [DynamoDB Data Modeling](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-modeling-nosql.html)

### Outils

- 🛠️ [DynamoDB Local Download](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html)
- 🛠️ [NoSQL Workbench](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)
- 🛠️ [dynamodb-admin (GitHub)](https://github.com/aaronshaf/dynamodb-admin)

### Tutoriels et cours

- 🎥 [AWS DynamoDB Tutorial (YouTube)](https://www.youtube.com/results?search_query=aws+dynamodb+tutorial)
- 📚 [DynamoDB Guide (Alex DeBrie)](https://www.dynamodbguide.com/)
- 🎓 [AWS Skill Builder (gratuit)](https://explore.skillbuilder.aws/)

### Communauté

- 💬 [AWS re:Post](https://repost.aws/)
- 💬 [Stack Overflow - dynamodb](https://stackoverflow.com/questions/tagged/amazon-dynamodb)
- 💬 [Reddit r/aws](https://www.reddit.com/r/aws/)

---

## ✅ Checklist avant de commencer

Avant de passer aux fiches pratiques, vérifiez que vous avez :

- [ ] Docker installé et fonctionnel
- [ ] Docker Compose installé
- [ ] Compris les concepts de base (table, item, clé primaire)
- [ ] Compris la différence entre DynamoDB Local et AWS
- [ ] 30 minutes devant vous pour suivre la première fiche

---

## 🚀 Prêt à commencer ?

Maintenant que vous comprenez les concepts de base de DynamoDB, passez à la pratique !

➡️ **[Commencez par la fiche 10.1 - Configuration basique](01-config-basique-docker-compose.md)**

Vous y apprendrez à :
- Démarrer DynamoDB Local en 5 minutes
- Vérifier que tout fonctionne
- Comprendre la configuration Docker Compose

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
➡️ Fiche suivante : **[10.1 - Configuration basique avec docker-compose](01-config-basique-docker-compose.md)**

---

*Dernière mise à jour : Novembre 2025*
