# Redis - Introduction et Guide

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que Redis ?

**Redis** (Remote Dictionary Server) est une base de données **NoSQL** de type **clé-valeur** qui fonctionne **entièrement en mémoire**. C'est l'une des bases de données les plus populaires au monde, utilisée par des millions d'applications.

### **Caractéristiques principales**

| Caractéristique | Description |
|----------------|-------------|
| 🚀 **Ultra-rapide** | Toutes les données sont en RAM (lecture/écriture en microsecondes) |
| 🔑 **Clé-Valeur** | Modèle simple : une clé → une valeur |
| 🛠️ **Structures riches** | Strings, Lists, Sets, Hashes, Sorted Sets, Bitmaps, HyperLogLogs, Streams |
| 💾 **Persistance** | Peut sauvegarder sur disque (RDB snapshots, AOF logs) |
| 🔄 **Pub/Sub** | Système de messagerie intégré |
| ⏰ **Expiration** | Clés avec TTL (time to live) automatique |
| 📊 **Transactions** | Support ACID avec MULTI/EXEC |
| 🌐 **Réplication** | Master-Slave pour haute disponibilité |
| 🔧 **Lua scripting** | Exécution de scripts côté serveur |
| 📖 **Open Source** | Licence BSD, communauté active |

---

## 🎯 Cas d'usage typiques

### **1. 🚀 Cache d'application**

Le cas d'usage le plus courant. Redis stocke temporairement des données fréquemment accédées pour éviter de solliciter la base de données principale.

**Exemple :**
```
Requête → Vérifier Redis → Si présent : retourner
                        → Si absent : requête BDD → stocker dans Redis → retourner
```

**Avantages :**
- ✅ Réduction de 80-90% de la charge sur la BDD
- ✅ Temps de réponse divisé par 10 ou plus
- ✅ Scalabilité améliorée

**Types de données cachées :**
- Résultats de requêtes SQL
- Pages HTML rendues
- Résultats d'API externes
- Calculs complexes

### **2. 📝 Gestion de sessions utilisateur**

Stocker les sessions PHP, Node.js, etc. pour permettre la distribution des requêtes entre plusieurs serveurs.

**Exemple :**
```
session:abc123 → { "user_id": 42, "role": "admin", "cart": [...] }
```

**Avantages :**
- ✅ Sessions partagées entre plusieurs serveurs web
- ✅ Persistence même en cas de redémarrage
- ✅ Expiration automatique (TTL)

### **3. 📨 File d'attente de tâches (Queue)**

Gérer des tâches asynchrones : envoi d'emails, génération de PDF, traitement d'images, etc.

**Exemple avec Lists :**
```redis
LPUSH queue:emails '{"to":"user@example.com","subject":"Welcome"}'
RPOP queue:emails  # Récupérer et traiter
```

**Avantages :**
- ✅ Découplement entre producteur et consommateur
- ✅ Gestion de la charge (rate limiting)
- ✅ Retry automatique en cas d'échec

### **4. 🏆 Classements et leaderboards**

Sorted Sets permettent de maintenir des classements en temps réel.

**Exemple - Top joueurs :**
```redis
ZADD leaderboard 1500 "Alice"
ZADD leaderboard 2300 "Bob"
ZADD leaderboard 1800 "Charlie"
ZREVRANGE leaderboard 0 9  # Top 10
```

**Avantages :**
- ✅ Mise à jour en O(log N)
- ✅ Récupération du classement instantanée
- ✅ Support de millions d'entrées

### **5. ⏱️ Rate Limiting (limitation de débit)**

Limiter le nombre de requêtes par utilisateur/IP.

**Exemple :**
```redis
INCR rate:user:123
EXPIRE rate:user:123 60  # Réinitialise après 60 secondes
GET rate:user:123  # Vérifier le compteur
```

**Avantages :**
- ✅ Protection contre le spam/abus
- ✅ Implémentation simple et rapide
- ✅ Distribué entre plusieurs serveurs

### **6. 📊 Compteurs et statistiques en temps réel**

Suivre des métriques : vues de pages, clics, événements, etc.

**Exemple :**
```redis
INCR page:home:views
INCRBY user:123:points 10
HINCRBY stats:2025-10-31 views 1
```

**Avantages :**
- ✅ Opérations atomiques (pas de race conditions)
- ✅ Très rapide (millions d'incréments/seconde)
- ✅ Agrégation facile

### **7. 🔔 Système Pub/Sub (messagerie)**

Communication en temps réel entre applications.

**Exemple - Chat en temps réel :**
```redis
SUBSCRIBE chat:room:42
PUBLISH chat:room:42 "Nouveau message !"
```

**Avantages :**
- ✅ Latence très faible
- ✅ Multiple subscribers
- ✅ Pas de persistance (léger)

### **8. 🔐 Gestion de tokens et authentification**

Stocker des tokens JWT, sessions OAuth, codes de vérification, etc.

**Exemple :**
```redis
SET token:xyz123 "user_id:42" EX 3600  # Expire dans 1h
GET token:xyz123  # Vérifier la validité
```

**Avantages :**
- ✅ Expiration automatique
- ✅ Révocation instantanée
- ✅ Distribué (multi-serveurs)

---

## 🏗️ Architecture et concepts

### **Modèle de données**

Redis utilise un modèle **clé-valeur** simple :

```
Clé (string) → Valeur (type varié)
```

**Exemple :**
```redis
"user:123:name" → "Alice"
"user:123:email" → "alice@example.com"
"counters:visits" → 42
"session:abc" → {"user_id": 123, "role": "admin"}
```

### **Types de données supportés**

| Type | Description | Cas d'usage |
|------|-------------|-------------|
| **String** | Valeur simple (texte, nombre, JSON) | Cache, compteurs, flags |
| **List** | Liste ordonnée (doubly linked list) | Files d'attente, historiques, feeds |
| **Set** | Ensemble non ordonné (unique) | Tags, membres, relations |
| **Sorted Set** | Ensemble trié par score | Classements, priorités, timelines |
| **Hash** | Map clé-valeur (objet) | Objets, profils utilisateurs |
| **Bitmap** | Tableau de bits | Statistiques binaires, bloom filters |
| **HyperLogLog** | Compteur probabiliste | Comptage unique approximatif |
| **Stream** | Log append-only | Event sourcing, logs, time series |
| **Geospatial** | Coordonnées GPS | Géolocalisation, recherche de proximité |

### **Persistance**

Redis propose **deux méthodes** de sauvegarde :

#### **1. RDB (Redis Database Backup)**

Snapshots périodiques de toutes les données.

**Avantages :**
- ✅ Fichiers compacts
- ✅ Rapide à charger
- ✅ Idéal pour backups

**Inconvénients :**
- ❌ Perte possible entre deux snapshots
- ❌ Peut bloquer temporairement

**Configuration :**
```redis
SAVE 900 1      # Sauvegarde après 15 min si au moins 1 modification
SAVE 300 10     # Sauvegarde après 5 min si au moins 10 modifications
SAVE 60 10000   # Sauvegarde après 1 min si au moins 10 000 modifications
```

#### **2. AOF (Append Only File)**

Log de toutes les commandes d'écriture.

**Avantages :**
- ✅ Perte minimale de données
- ✅ Fichier en texte lisible
- ✅ Récupération automatique

**Inconvénients :**
- ❌ Fichiers plus volumineux
- ❌ Plus lent que RDB

**Configuration :**
```redis
appendonly yes
appendfsync everysec  # Synchronise toutes les secondes
```

**Recommandation :** Utilisez les **deux** (RDB + AOF) pour un maximum de sécurité.

### **Bases de données multiples**

Redis supporte **16 bases de données** (0-15 par défaut), isolées les unes des autres.

**Sélection :**
```redis
SELECT 0  # Base par défaut
SELECT 1  # Base 1
```

**Utilisation typique :**
- DB 0 : Production
- DB 1 : Tests
- DB 2 : Développement

⚠️ **Attention :** Ce n'est pas un système de multi-tenancy. Préférez plusieurs instances Redis pour isoler les environnements.

---

## 🚀 Pourquoi utiliser Docker pour Redis ?

### **Avantages de Redis sur Docker**

| Avantage | Description |
|----------|-------------|
| 🎯 **Installation rapide** | Démarrez Redis en 30 secondes |
| 🧹 **Environnement propre** | Pas de pollution du système |
| 🔄 **Versions multiples** | Testez différentes versions facilement |
| 📦 **Portable** | Même config sur tous les environnements |
| 🔧 **Configuration simple** | Tout dans docker-compose.yml |
| 🗑️ **Suppression propre** | Nettoyage en une commande |
| 🌐 **Réseaux isolés** | Sécurité et organisation |
| 💾 **Volumes persistants** | Données préservées |

### **Comparaison : Installation native vs Docker**

| Aspect | Installation native | Docker |
|--------|-------------------|--------|
| **Installation** | 15-30 minutes | 30 secondes |
| **Configuration** | Fichiers système | docker-compose.yml |
| **Mise à jour** | Manuelle, risquée | Changement de version d'image |
| **Multi-versions** | Complexe | Facile (plusieurs conteneurs) |
| **Nettoyage** | Difficile | `docker-compose down -v` |
| **Portabilité** | Configuration manuelle | Partage du docker-compose.yml |

---

## 📊 Redis vs autres bases de données

### **Redis vs Memcached**

| Critère | Redis | Memcached |
|---------|-------|-----------|
| **Types de données** | 9+ types | String uniquement |
| **Persistance** | Oui (RDB, AOF) | Non |
| **Réplication** | Oui (Master-Slave) | Non natif |
| **Pub/Sub** | Oui | Non |
| **Transactions** | Oui | Non |
| **Lua scripting** | Oui | Non |
| **Complexité** | Plus riche | Plus simple |
| **Performance** | Excellent | Légèrement plus rapide |
| **Cas d'usage** | Cache + BDD + Queues | Cache pur |

**Verdict :** Redis est plus polyvalent, Memcached est plus simple si vous n'avez besoin que de cache.

### **Redis vs bases SQL (MySQL, PostgreSQL)**

| Critère | Redis | SQL |
|---------|-------|-----|
| **Stockage** | RAM (+ disque optionnel) | Disque |
| **Performance** | Microsecondes | Millisecondes |
| **Structure** | Clé-Valeur | Tables relationnelles |
| **Requêtes** | Commandes simples | SQL complexe |
| **Relations** | Manuelles | Natives (JOIN) |
| **Transactions** | Basique (MULTI/EXEC) | ACID complet |
| **Durabilité** | Moyenne | Élevée |
| **Scalabilité** | Horizontale facile | Verticale principalement |
| **Cas d'usage** | Cache, sessions, temps réel | Données structurées, relations |

**Verdict :** Redis complète les BDD SQL, ne les remplace pas. Utilisez les deux ensemble !

### **Redis vs MongoDB**

| Critère | Redis | MongoDB |
|---------|-------|---------|
| **Modèle** | Clé-Valeur | Document (JSON) |
| **Stockage** | RAM | Disque |
| **Performance** | Très rapide | Rapide |
| **Requêtes** | Par clé | Complexes (agrégations) |
| **Persistance** | Optionnelle | Par défaut |
| **Schéma** | Sans | Sans (flexible) |
| **Cas d'usage** | Cache, temps réel | Documents, CMS, API |

**Verdict :** Redis pour les données éphémères et le temps réel, MongoDB pour les documents persistants.

---

## 🛠️ Redis et Docker : Concepts à connaître

### **Volumes Docker**

Les volumes permettent de **persister** les données Redis même après suppression du conteneur.

**Types de volumes :**

1. **Volume nommé** (recommandé) :
```yaml
volumes:
  - redis_data:/data
```

2. **Bind mount** (développement) :
```yaml
volumes:
  - ./data:/data
```

### **Réseaux Docker**

Les réseaux isolent les conteneurs et permettent la communication.

**Exemple :**
```yaml
networks:
  redis_net:
    driver: bridge
```

### **Ports**

Exposition du port Redis pour accès depuis l'hôte.

```yaml
ports:
  - "6379:6379"
```

- `6379` (gauche) : Port sur votre machine
- `6379` (droite) : Port dans le conteneur

---

## 📚 Structure de ce guide

Ce guide Redis est organisé en **5 fiches progressives** :

### **6.1 - Configuration basique**
🎯 **Objectif :** Démarrer Redis en moins de 2 minutes
📖 **Contenu :**
- Installation simple avec docker-compose
- Premiers tests avec redis-cli
- Connexion depuis Python/Node.js
- Commandes Redis de base

### **6.2 - Configuration avec mot de passe**
🎯 **Objectif :** Sécuriser Redis avec authentification
📖 **Contenu :**
- Activation du mot de passe
- Connexion avec authentification
- Fichier .env pour les secrets
- Bonnes pratiques de sécurité

### **6.3 - Configuration avancée avec redis.conf**
🎯 **Objectif :** Maîtriser toutes les options de Redis
📖 **Contenu :**
- Fichier redis.conf complet et commenté
- Persistance (RDB vs AOF)
- Optimisation des performances
- Gestion de la mémoire

### **6.4 - Configuration avec IP fixe**
🎯 **Objectif :** Assigner une adresse IP statique
📖 **Contenu :**
- Création de réseaux Docker
- Attribution d'IP fixe
- Configuration multi-conteneurs
- Communication inter-services

### **6.5 - Redis Commander (GUI)**
🎯 **Objectif :** Gérer Redis visuellement
📖 **Contenu :**
- Installation de Redis Commander
- Interface web complète
- Gestion visuelle des données
- Multi-instances Redis

---

## 🎓 Pour qui est ce guide ?

### **✅ Vous êtes au bon endroit si :**
- Vous voulez apprendre Redis rapidement
- Vous développez des applications web modernes
- Vous cherchez à améliorer les performances de votre app
- Vous voulez utiliser Docker pour le développement
- Vous êtes étudiant, développeur ou architecte

### **🎯 Prérequis**
- Docker installé (version 20.10+)
- Docker Compose (version 2.0+)
- Notions de ligne de commande
- Bases de programmation (Python, Node.js, PHP...)

### **📖 Niveau**
- **Débutant** : Toutes les commandes sont expliquées
- **Intermédiaire** : Concepts avancés (persistance, sécurité)
- **Avancé** : Optimisation et production

---

## 🚦 Par où commencer ?

### **🆕 Débutant avec Redis et Docker ?**

1. Lisez cette introduction (vous y êtes !)
2. Commencez par **[6.1 - Configuration basique](01-config-basique-docker-compose.md)**
3. Sécurisez avec **[6.2 - Configuration avec mot de passe](02-config-avec-password.md)**
4. Explorez l'interface avec **[6.5 - Redis Commander](05-redis-commander.md)**

### **🔧 Vous connaissez déjà Redis ?**

1. Allez directement à **[6.3 - Configuration avancée](03-config-avec-redisconf.md)**
2. Configurez des **[6.4 - IPs fixes](04-config-ip-fixe.md)** pour vos projets
3. Consultez les **[Annexes](../annexes/)** pour référence

### **🚀 Vous voulez un environnement complet ?**

Consultez les **[Cas pratiques](../cas-pratiques/)** :
- Stack LAMP (Apache + MariaDB + PHP + Redis)
- Stack MEAN (MongoDB + Express + Node + Redis)
- Environnement de développement multi-BDD

---

## 📖 Ressources complémentaires

### **Documentation officielle**
- 🌐 [Redis.io](https://redis.io/) - Site officiel
- 📚 [Documentation Redis](https://redis.io/docs/) - Guide complet
- 🎓 [Redis University](https://university.redis.com/) - Cours gratuits
- 🔧 [Commandes Redis](https://redis.io/commands/) - Référence complète

### **Outils**
- 🐳 [Image Docker officielle](https://hub.docker.com/_/redis)
- 🖥️ [RedisInsight](https://redis.com/redis-enterprise/redis-insight/) - GUI officiel
- 📊 [Redis Commander](https://joeferner.github.io/redis-commander/) - GUI web

### **Communauté**
- 💬 [Redis Discord](https://discord.gg/redis)
- 🐙 [Redis GitHub](https://github.com/redis/redis)
- 📰 [Redis Blog](https://redis.io/blog/)

---

## ⚠️ Avertissements

### **Ce guide est pour le DÉVELOPPEMENT**

Les configurations présentées sont optimisées pour le **développement local**, pas pour la **production**.

**En production, vous devez :**
- 🔒 Activer SSL/TLS
- 🔐 Utiliser des mots de passe très forts
- 🛡️ Configurer un pare-feu
- 📊 Mettre en place du monitoring
- 💾 Planifier des backups réguliers
- 🌐 Utiliser la réplication (Master-Slave)
- 📈 Optimiser les performances
- 🔍 Auditer la sécurité

Consultez l'**[Annexe D - Sécurité](../annexes/D-securite-bonnes-pratiques.md)** avant de passer en production.

---

## 🎉 Prêt à commencer ?

Vous avez maintenant une vue d'ensemble de Redis et de ce que vous allez apprendre. Il est temps de passer à la pratique !

**➡️ Commencez par : [6.1 - Configuration basique avec docker-compose](01-config-basique-docker-compose.md)**

Ou explorez les autres fiches selon vos besoins. Bon apprentissage ! 🚀

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
