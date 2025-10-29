# PostgreSQL - Configuration Avancée avec postgresql.conf

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Par défaut, PostgreSQL utilise une configuration générique qui convient à la plupart des cas. Cependant, pour optimiser les performances, activer certaines fonctionnalités ou adapter PostgreSQL à vos besoins spécifiques, vous devrez personnaliser le fichier `postgresql.conf`.

Cette fiche vous guide pour créer et utiliser votre propre configuration PostgreSQL dans Docker.

**Ce que vous allez apprendre :**
- Comprendre le fichier `postgresql.conf`
- Créer une configuration personnalisée
- Monter le fichier dans votre conteneur Docker
- Optimiser PostgreSQL pour le développement
- Activer des fonctionnalités avancées (logs, performances)

**Durée estimée :** 15-20 minutes

---

## 🎯 Objectif

À la fin de cette fiche, vous aurez :
- ✅ Un fichier `postgresql.conf` personnalisé
- ✅ PostgreSQL configuré selon vos besoins
- ✅ Des logs détaillés pour le debug
- ✅ Des performances optimisées pour le développement
- ✅ La compréhension des paramètres importants

---

## 📦 Prérequis

Avant de commencer :
- ✅ Avoir suivi la [Configuration basique](01-config-basique-docker-compose.md)
- ✅ Comprendre les [Concepts Docker](/00-introduction/02-concepts-docker.md)
- ✅ PostgreSQL doit fonctionner en configuration basique

---

## 🧠 Comprendre postgresql.conf

### Qu'est-ce que postgresql.conf ?

Le fichier `postgresql.conf` est le **fichier de configuration principal** de PostgreSQL. Il contient des centaines de paramètres qui contrôlent :
- Les performances (mémoire, cache, parallélisme)
- Les connexions (nombre max, timeouts)
- Les logs (niveau de détail, format)
- La sécurité (authentification, SSL)
- Les fonctionnalités (réplication, extensions)

### Où se Trouve-t-il ?

**Dans PostgreSQL "classique"** : `/etc/postgresql/{version}/main/postgresql.conf`

**Dans le conteneur Docker** : `/var/lib/postgresql/data/postgresql.conf`

### Pourquoi Utiliser un Fichier Personnalisé ?

**Sans fichier personnalisé** : PostgreSQL utilise les valeurs par défaut (génériques)

**Avec fichier personnalisé** :
- ✅ Optimisation pour votre usage (dev, test, production)
- ✅ Logs détaillés pour débugger
- ✅ Activation de fonctionnalités spécifiques
- ✅ Configuration versionnée (dans Git)
- ✅ Reproductible sur toutes les machines

---

## 🚀 Étape 1 : Préparation de la Structure

### Créer la Structure de Dossiers

```bash
# Depuis votre dossier postgres-docker
mkdir -p config
```

**Structure obtenue :**
```
postgres-docker/
├── config/
│   └── postgresql.conf    # À créer
├── data/                  # Données PostgreSQL
└── docker-compose.yml     # À modifier
```

---

## 📝 Étape 2 : Créer le Fichier postgresql.conf

Créez le fichier `config/postgresql.conf` avec cette configuration optimisée pour le développement :

```ini
# ========================================
# POSTGRESQL.CONF - Configuration Développement
# ========================================

# -------------------
# CONNEXIONS
# -------------------

# Nombre maximum de connexions simultanées
max_connections = 100

# Port d'écoute (laissez 5432, le mapping se fait dans docker-compose.yml)
# port = 5432

# Adresses d'écoute (0.0.0.0 = toutes les interfaces)
listen_addresses = '*'

# -------------------
# MÉMOIRE
# -------------------

# Mémoire partagée pour le cache (25% de la RAM disponible)
# Pour un PC avec 8 GB RAM → 2 GB
# Pour un PC avec 16 GB RAM → 4 GB
shared_buffers = 1GB

# Mémoire par opération de tri/hash
# Augmente les perfs pour les ORDER BY, GROUP BY, JOIN
work_mem = 16MB

# Mémoire pour les opérations de maintenance (VACUUM, CREATE INDEX)
maintenance_work_mem = 256MB

# Mémoire pour les connexions (max_connections * work_mem ne doit pas dépasser la RAM)
effective_cache_size = 3GB

# -------------------
# WRITE-AHEAD LOG (WAL)
# -------------------

# Niveau de log pour la réplication et la récupération
# minimal = performances max, replica = permet réplication
wal_level = minimal

# Taille des segments WAL
# Plus grand = moins d'I/O mais plus d'espace disque
# min_wal_size = 80MB
# max_wal_size = 1GB

# -------------------
# LOGS ET DÉBOGAGE
# -------------------

# Niveau de log : debug5, debug4, debug3, debug2, debug1, info, notice, warning, error, log, fatal, panic
log_min_messages = notice
log_min_error_statement = error

# Log toutes les requêtes (utile en développement, lourd en production)
log_statement = 'all'

# Log la durée de chaque requête
log_duration = on

# Log les requêtes lentes (en millisecondes, 0 = désactivé)
# 1000 = log si requête > 1 seconde
log_min_duration_statement = 1000

# Préfixe des logs avec timestamp
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

# Destination des logs (stderr = sortie standard Docker)
log_destination = 'stderr'

# Format des logs (stderr, csvlog, syslog, eventlog)
logging_collector = off

# -------------------
# PERFORMANCES
# -------------------

# Coût estimé d'une lecture aléatoire (SSD = 1.0, HDD = 4.0)
random_page_cost = 1.1

# Coût estimé d'une lecture séquentielle
seq_page_cost = 1.0

# Activer le parallélisme des requêtes
max_parallel_workers_per_gather = 2
max_parallel_workers = 4
max_worker_processes = 4

# -------------------
# AUTOVACUUM (Nettoyage automatique)
# -------------------

# Activer l'autovacuum (recommandé)
autovacuum = on

# Paramètres d'autovacuum (valeurs par défaut adaptées)
# autovacuum_max_workers = 3
# autovacuum_naptime = 1min

# -------------------
# STATISTIQUES
# -------------------

# Collecter les statistiques pour l'optimiseur de requêtes
track_activities = on
track_counts = on
track_io_timing = on
track_functions = all

# -------------------
# TIMEZONE ET LOCALE
# -------------------

# Timezone par défaut (Europe/Paris pour la France)
timezone = 'Europe/Paris'

# Locale par défaut
lc_messages = 'en_US.utf8'
lc_monetary = 'en_US.utf8'
lc_numeric = 'en_US.utf8'
lc_time = 'en_US.utf8'

# Encodage par défaut
default_text_search_config = 'pg_catalog.english'

# -------------------
# EXTENSIONS
# -------------------

# Bibliothèques à précharger (ex: pg_stat_statements pour stats de requêtes)
# shared_preload_libraries = 'pg_stat_statements'

# -------------------
# DÉVELOPPEMENT
# -------------------

# Désactiver fsync pour plus de vitesse (DANGER : uniquement en dev !)
# ⚠️ Ne JAMAIS utiliser en production (risque de corruption)
# fsync = off

# Checkpoint moins fréquents = plus rapide (dev uniquement)
# checkpoint_timeout = 15min
```

### 📖 Explication des Sections Principales

| Section | Description | Impact |
|---------|-------------|--------|
| **Connexions** | Nombre max de clients, ports | Capacité d'utilisation |
| **Mémoire** | Allocation RAM, cache | **Performances** |
| **WAL** | Logs de transactions | Fiabilité / Réplication |
| **Logs** | Niveau de détail, format | **Debug et monitoring** |
| **Performances** | Parallélisme, coûts | **Vitesse des requêtes** |
| **Autovacuum** | Nettoyage automatique | Maintenance |
| **Statistiques** | Collecte d'infos | Optimisation |
| **Timezone/Locale** | Formats dates/nombres | Affichage |

---

## 🔧 Étape 3 : Modifier docker-compose.yml

Modifiez votre `docker-compose.yml` pour utiliser le fichier de configuration personnalisé :

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: postgres_dev
    restart: unless-stopped

    environment:
      POSTGRES_PASSWORD: changez_moi_123!
      POSTGRES_DB: ma_base
      POSTGRES_USER: mon_utilisateur

    ports:
      - "5432:5432"

    volumes:
      # Données persistantes
      - ./data:/var/lib/postgresql/data

      # ✨ NOUVEAU : Configuration personnalisée
      # Le :ro signifie "read-only" (PostgreSQL ne peut pas modifier ce fichier)
      - ./config/postgresql.conf:/etc/postgresql/postgresql.conf:ro

    # ✨ NOUVEAU : Commande pour utiliser notre fichier de config
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
```

### 📖 Explication des Modifications

**Ligne ajoutée 1 :**
```yaml
- ./config/postgresql.conf:/etc/postgresql/postgresql.conf:ro
```
- Monte votre fichier `postgresql.conf` dans le conteneur
- `:ro` = read-only (sécurité : PostgreSQL ne peut pas écrire dedans)

**Ligne ajoutée 2 :**
```yaml
command: postgres -c config_file=/etc/postgresql/postgresql.conf
```
- Dit à PostgreSQL d'utiliser votre fichier au lieu du fichier par défaut
- `-c config_file=...` = paramètre de ligne de commande

---

## ▶️ Étape 4 : Appliquer la Configuration

### Si PostgreSQL n'est Pas Encore Lancé

```bash
docker-compose up -d
```

### Si PostgreSQL Tourne Déjà

Vous devez recréer le conteneur pour appliquer les changements :

```bash
# Arrêter le conteneur actuel
docker-compose down

# Redémarrer avec la nouvelle configuration
docker-compose up -d
```

**⚠️ Important** : Les données dans `./data` sont conservées.

### Vérifier que la Configuration est Appliquée

```bash
# Se connecter à PostgreSQL
docker exec -it postgres_dev psql -U mon_utilisateur -d ma_base

# Vérifier quelques paramètres
SHOW shared_buffers;
SHOW work_mem;
SHOW log_statement;
SHOW timezone;

# Quitter
\q
```

**Résultats attendus :**
```
shared_buffers    | 1GB
work_mem          | 16MB
log_statement     | all
timezone          | Europe/Paris
```

---

## 📊 Étape 5 : Tester et Observer

### Voir les Logs en Temps Réel

```bash
docker-compose logs -f
```

**Avec `log_statement = 'all'`, vous verrez toutes les requêtes :**
```
postgres_dev  | 2024-10-29 14:23:45.123 UTC [123]: [1-1] user=mon_utilisateur,db=ma_base,app=psql,client=172.18.0.1 LOG:  statement: SELECT * FROM utilisateurs;
```

### Tester les Performances

Créez une table avec des données :

```sql
-- Se connecter
docker exec -it postgres_dev psql -U mon_utilisateur -d ma_base

-- Créer une table de test
CREATE TABLE test_perf (
    id SERIAL PRIMARY KEY,
    nom VARCHAR(100),
    valeur INT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insérer 100 000 lignes
INSERT INTO test_perf (nom, valeur)
SELECT
    'Ligne ' || i,
    (random() * 1000)::int
FROM generate_series(1, 100000) AS i;

-- Créer un index
CREATE INDEX idx_valeur ON test_perf(valeur);

-- Tester une requête lente (> 1 seconde)
SELECT pg_sleep(2);

-- Analyser les performances
EXPLAIN ANALYZE SELECT * FROM test_perf WHERE valeur > 500 ORDER BY created_at;
```

**Dans les logs**, vous verrez :
- La durée de chaque requête
- Les requêtes qui ont pris plus de 1 seconde
- Les détails de connexion

---

## 🎛️ Personnalisation Selon Vos Besoins

### Scénario 1 : Développement Rapide (Performances Max)

**Objectif :** Vitesse maximale, peu importe la sécurité

```ini
# ⚠️ DÉVELOPPEMENT UNIQUEMENT - DANGER EN PRODUCTION

# Désactiver fsync (gain de vitesse énorme)
fsync = off

# Désactiver synchronous_commit (moins de latence)
synchronous_commit = off

# Checkpoints moins fréquents
checkpoint_timeout = 30min
checkpoint_completion_target = 0.9

# Augmenter la mémoire
shared_buffers = 2GB
work_mem = 32MB
maintenance_work_mem = 512MB
```

**⚠️ DANGER** : Avec `fsync = off`, vous risquez une corruption de données en cas de crash. **UNIQUEMENT pour le développement !**

### Scénario 2 : Debug Intensif (Logs Détaillés)

**Objectif :** Voir absolument tout pour débugger

```ini
# Log TOUTES les requêtes sans exception
log_statement = 'all'

# Log même les requêtes rapides
log_min_duration_statement = 0

# Niveau de log très détaillé
log_min_messages = debug1

# Log les connexions/déconnexions
log_connections = on
log_disconnections = on

# Log les checkpoints
log_checkpoints = on

# Log les locks (verrous)
log_lock_waits = on
deadlock_timeout = 1s

# Détails des requêtes lentes
log_min_duration_statement = 100
auto_explain.log_min_duration = 500
```

**⚠️ Attention** : Les logs deviennent TRÈS volumineux. Limitez la taille avec Docker (voir [Bonnes Pratiques](/00-introduction/03-bonnes-pratiques.md)).

### Scénario 3 : Production-Like (Sécurité et Stabilité)

**Objectif :** Simuler un environnement de production

```ini
# Sécurité maximale
fsync = on
synchronous_commit = on
full_page_writes = on

# Logs modérés
log_statement = 'ddl'  # Uniquement CREATE, ALTER, DROP
log_min_duration_statement = 5000  # Seulement si > 5 secondes

# Connexions limitées
max_connections = 50

# Mémoire conservatrice
shared_buffers = 512MB
work_mem = 8MB
maintenance_work_mem = 128MB

# WAL pour réplication
wal_level = replica
max_wal_senders = 3
```

---

## 🔧 Paramètres Importants à Connaître

### Mémoire (Performance)

| Paramètre | Recommandation Dev | Description |
|-----------|-------------------|-------------|
| `shared_buffers` | 25% de la RAM | Cache partagé entre toutes les connexions |
| `work_mem` | 16-64 MB | Mémoire par opération (tri, jointure) |
| `maintenance_work_mem` | 256-512 MB | Pour VACUUM, CREATE INDEX |
| `effective_cache_size` | 50-75% de la RAM | Indique à l'optimiseur la RAM totale disponible |

**Calcul simple pour 8 GB de RAM :**
```ini
shared_buffers = 2GB          # 25% de 8 GB
work_mem = 32MB              # Pour ~60 connexions
maintenance_work_mem = 512MB  # 6% de 8 GB
effective_cache_size = 6GB    # 75% de 8 GB
```

### Connexions

| Paramètre | Valeur Dev | Valeur Prod | Description |
|-----------|-----------|-------------|-------------|
| `max_connections` | 100 | 200-500 | Nombre max de clients simultanés |
| `superuser_reserved_connections` | 3 | 3-5 | Connexions réservées admin |

**Note** : Plus de connexions = plus de RAM utilisée (`work_mem` × `max_connections`)

### Logs

| Paramètre | Valeur | Utilité |
|-----------|--------|---------|
| `log_statement` | `'all'` | Log toutes les requêtes (debug) |
| `log_statement` | `'ddl'` | Log uniquement les CREATE, ALTER, DROP |
| `log_statement` | `'none'` | Désactive (production légère) |
| `log_min_duration_statement` | `1000` | Log si requête > 1 seconde |
| `log_line_prefix` | `'%t [%p]...'` | Format des logs |

### WAL (Write-Ahead Log)

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| `wal_level` | `minimal` | Dev simple (pas de réplication) |
| `wal_level` | `replica` | Permet réplication/streaming |
| `wal_level` | `logical` | Permet réplication logique |

---

## 🔍 Vérifier et Diagnostiquer

### Voir Toutes les Configurations Actives

```sql
-- Se connecter
docker exec -it postgres_dev psql -U mon_utilisateur -d ma_base

-- Voir TOUS les paramètres
SHOW ALL;

-- Voir un paramètre spécifique
SHOW shared_buffers;
SHOW work_mem;

-- Voir la source du paramètre (fichier, défaut, ligne de commande)
SELECT name, setting, source, sourcefile
FROM pg_settings
WHERE name IN ('shared_buffers', 'work_mem', 'log_statement');
```

### Voir les Statistiques de Mémoire

```sql
-- Mémoire utilisée par PostgreSQL
SELECT pg_size_pretty(pg_database_size('ma_base')) AS taille_base;

-- Voir le cache
SELECT * FROM pg_stat_database WHERE datname = 'ma_base';
```

### Recharger la Configuration Sans Redémarrer

Certains paramètres peuvent être rechargés sans redémarrer PostgreSQL :

```bash
# Éditer votre postgresql.conf
nano config/postgresql.conf

# Recharger (depuis psql)
docker exec -it postgres_dev psql -U mon_utilisateur -d ma_base -c "SELECT pg_reload_conf();"

# Ou via signal
docker exec postgres_dev pg_ctl reload
```

**Paramètres nécessitant un redémarrage :**
- `shared_buffers`
- `max_connections`
- `wal_level`
- `max_worker_processes`

**Paramètres rechargeables à chaud :**
- `work_mem`
- `log_statement`
- `log_min_duration_statement`
- La plupart des paramètres de logs

---

## 🧹 Maintenance de la Configuration

### Sauvegarder Votre Configuration

```bash
# Votre fichier est déjà dans ./config/
# Versionnez-le dans Git

git add config/postgresql.conf
git commit -m "feat(postgres): configuration optimisée pour dev"
```

### Revenir à la Configuration Par Défaut

Supprimez simplement les lignes ajoutées dans `docker-compose.yml` :

```yaml
# Supprimer ces lignes
volumes:
  - ./config/postgresql.conf:/etc/postgresql/postgresql.conf:ro
command: postgres -c config_file=/etc/postgresql/postgresql.conf

# Recréer le conteneur
docker-compose down
docker-compose up -d
```

### Tester Plusieurs Configurations

Créez plusieurs fichiers :
```
config/
├── postgresql.conf          # Par défaut
├── postgresql-dev.conf      # Développement
├── postgresql-prod.conf     # Production-like
└── postgresql-debug.conf    # Debug intensif
```

Dans `docker-compose.yml`, changez le fichier monté :
```yaml
- ./config/postgresql-debug.conf:/etc/postgresql/postgresql.conf:ro
```

---

## ⚠️ Erreurs Courantes

### Erreur : "Configuration file does not exist"

**Cause** : Le chemin vers `postgresql.conf` est incorrect.

**Solution** :
```bash
# Vérifier que le fichier existe
ls -la config/postgresql.conf

# Vérifier les permissions
chmod 644 config/postgresql.conf
```

### Erreur : PostgreSQL ne démarre pas après changement

**Cause** : Erreur de syntaxe dans `postgresql.conf`.

**Solution** :
```bash
# Voir les logs d'erreur
docker-compose logs

# Tester la configuration
docker exec postgres_dev postgres --config-file=/etc/postgresql/postgresql.conf -C shared_buffers
```

### Erreur : "out of shared memory"

**Cause** : `shared_buffers` trop élevé pour la RAM disponible.

**Solution** : Réduisez `shared_buffers` dans `postgresql.conf` :
```ini
shared_buffers = 512MB  # Au lieu de 2GB
```

### Logs Trop Volumineux

**Cause** : `log_statement = 'all'` génère énormément de logs.

**Solution** : Limitez la taille des logs dans `docker-compose.yml` :
```yaml
services:
  postgres:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 📊 Comparaison des Configurations

| Aspect | Config par Défaut | Config Dev Optimisée | Config Production-Like |
|--------|------------------|---------------------|----------------------|
| **Performances** | Moyennes | ⚡ Très rapides | Stables |
| **Sécurité** | Bonne | ⚠️ Réduite | 🔒 Maximale |
| **Logs** | Minimaux | 📋 Détaillés | Modérés |
| **RAM** | Faible | Élevée | Moyenne |
| **Risque corruption** | Très faible | ⚠️ Moyen (si fsync=off) | Très faible |
| **Use Case** | Découverte | **Développement** | Tests pré-prod |

---

## ✅ Checklist de Configuration

Avant de considérer votre configuration prête :

### Fichiers
- [ ] `config/postgresql.conf` créé et syntaxe correcte
- [ ] `docker-compose.yml` modifié pour monter le fichier
- [ ] Fichier versionné dans Git (si applicable)

### Paramètres Essentiels
- [ ] `shared_buffers` adapté à votre RAM
- [ ] `work_mem` calculé selon `max_connections`
- [ ] `timezone` configurée (Europe/Paris, America/New_York...)
- [ ] `log_statement` adapté à vos besoins (all/ddl/none)

### Tests
- [ ] PostgreSQL démarre sans erreur
- [ ] `SHOW shared_buffers;` retourne la bonne valeur
- [ ] Les logs apparaissent dans `docker-compose logs`
- [ ] Les requêtes lentes sont loggées

### Performance
- [ ] Requêtes plus rapides qu'avant (si optimisation faite)
- [ ] Logs exploitables pour débugger
- [ ] RAM utilisée raisonnable (`docker stats`)

---

## 🚀 Prochaines Étapes

Maintenant que votre PostgreSQL est configuré :

1. **Configurer une IP fixe** → [Configuration IP fixe](03-config-ip-fixe.md)
2. **Créer des utilisateurs** → [Gestion des utilisateurs et BDD](04-gestion-utilisateurs-bdd.md)
3. **Ajouter pgAdmin** → [Configuration avec pgAdmin](05-config-avec-pgadmin.md)
4. **Optimiser encore plus** → [Annexe D - Sécurité](/annexes/D-securite-bonnes-pratiques.md)

---

## 📚 Ressources pour Aller Plus Loin

### Documentation Officielle
- **PostgreSQL Configuration** : [https://www.postgresql.org/docs/current/runtime-config.html](https://www.postgresql.org/docs/current/runtime-config.html)
- **PostgreSQL Tuning** : [https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)

### Outils d'Optimisation
- **PGTune** (génère config optimale) : [https://pgtune.leopard.in.ua/](https://pgtune.leopard.in.ua/)
- **PG Config** (calculateur) : [https://pgconfig.rustprooflabs.com/](https://pgconfig.rustprooflabs.com/)

### Articles et Guides
- **PostgreSQL Memory Settings** : [https://postgresqlco.nf/doc/en/param/](https://postgresqlco.nf/doc/en/param/)
- **PostgreSQL Logging Best Practices** : Recherchez sur le blog PostgreSQL officiel

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
