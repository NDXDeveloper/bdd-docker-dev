# 6.3 - Configuration avancée de Redis avec redis.conf

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📄 Qu'est-ce que redis.conf ?

**redis.conf** est le fichier de configuration principal de Redis. Il permet de personnaliser **tous les aspects** du serveur Redis :

- 🔒 **Sécurité** : Mots de passe, limitation d'accès, commandes désactivées
- 💾 **Persistance** : Comment et quand sauvegarder les données sur disque
- 🚀 **Performance** : Limites de mémoire, politique d'éviction, optimisations
- 🌐 **Réseau** : Ports, adresses IP, timeouts
- 📊 **Logging** : Niveau de détail des logs, fichiers de log
- ⚙️ **Comportement** : Nombre de bases de données, encodage, etc.

**Pourquoi utiliser redis.conf ?**
- ✅ Configuration complète et centralisée
- ✅ Reproductible (partage avec l'équipe)
- ✅ Versionnable (suivi des modifications)
- ✅ Plus professionnel que les variables d'environnement
- ✅ Accès à toutes les options avancées de Redis

---

## 🎯 Objectif de cette fiche

Vous allez apprendre à :
1. ✅ Créer et personnaliser un fichier `redis.conf`
2. ✅ Comprendre les paramètres essentiels
3. ✅ Configurer la persistance (RDB et AOF)
4. ✅ Optimiser les performances et la mémoire
5. ✅ Appliquer des mesures de sécurité avancées
6. ✅ Utiliser ce fichier avec Docker Compose

---

## 🚀 Configuration de base

### **Étape 1 : Créer le dossier du projet**

```bash
mkdir redis-advanced
cd redis-advanced
```

### **Étape 2 : Créer le fichier `redis.conf`**

Créez un fichier `redis.conf` avec cette configuration de base commentée :

```bash
# ============================================================================
# REDIS.CONF - Configuration Redis pour développement
# ============================================================================

# ----------------------------------------------------------------------------
# RÉSEAU
# ----------------------------------------------------------------------------

# Adresse(s) IP sur laquelle Redis écoute
# 127.0.0.1 = localhost uniquement (sécurisé)
# 0.0.0.0 = toutes les interfaces (attention en production !)
bind 127.0.0.1

# Port d'écoute (défaut : 6379)
port 6379

# Timeout de connexion inactive (0 = désactivé)
timeout 300

# Keepalive TCP (secondes)
tcp-keepalive 60

# ----------------------------------------------------------------------------
# SÉCURITÉ
# ----------------------------------------------------------------------------

# Mot de passe d'authentification
# ⚠️ Changez ce mot de passe !
requirepass VotreMotDePasseSecurise123!

# Mode protégé (refuse les connexions externes sans mot de passe)
protected-mode yes

# Renommer ou désactiver des commandes dangereuses
# Syntaxe : rename-command COMMANDE "NouveauNom"
# Syntaxe : rename-command COMMANDE "" (désactive)
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_SECRET_8574"

# ----------------------------------------------------------------------------
# LIMITES
# ----------------------------------------------------------------------------

# Nombre maximum de clients connectés simultanément
maxclients 10000

# Limite de mémoire (exemple : 256 MB)
# Formats acceptés : 1k, 1kb, 1m, 1mb, 1g, 1gb
maxmemory 256mb

# Politique d'éviction quand maxmemory est atteinte
# Options :
# - noeviction : retourne une erreur, n'évince rien (défaut)
# - allkeys-lru : évince les clés les moins récemment utilisées
# - volatile-lru : évince les clés avec TTL, les moins récemment utilisées
# - allkeys-random : évince des clés aléatoires
# - volatile-random : évince des clés aléatoires parmi celles avec TTL
# - volatile-ttl : évince les clés avec le TTL le plus court
maxmemory-policy allkeys-lru

# ----------------------------------------------------------------------------
# PERSISTANCE - SNAPSHOTS RDB
# ----------------------------------------------------------------------------

# Sauvegarde automatique (format : save <secondes> <modifications>)
# "Sauvegarde si au moins X modifications en Y secondes"
save 900 1      # Sauvegarde après 15 min si au moins 1 modification
save 300 10     # Sauvegarde après 5 min si au moins 10 modifications
save 60 10000   # Sauvegarde après 1 min si au moins 10 000 modifications

# Arrêter d'accepter les écritures si la dernière sauvegarde a échoué
stop-writes-on-bgsave-error yes

# Compresser les fichiers RDB avec LZF
rdbcompression yes

# Vérifier l'intégrité du fichier RDB avec checksum
rdbchecksum yes

# Nom du fichier de sauvegarde
dbfilename dump.rdb

# Dossier où sont stockés les fichiers de persistance
dir /data

# ----------------------------------------------------------------------------
# PERSISTANCE - APPEND ONLY FILE (AOF)
# ----------------------------------------------------------------------------

# Activer le mode AOF (log de toutes les écritures)
# Plus sûr que RDB mais plus lourd en I/O
appendonly no

# Nom du fichier AOF
appendfilename "appendonly.aof"

# Fréquence de synchronisation sur disque
# Options :
# - always : après chaque commande (très lent, très sûr)
# - everysec : toutes les secondes (bon compromis)
# - no : laisse le système décider (rapide, moins sûr)
appendfsync everysec

# Ne pas effectuer de fsync pendant un BGSAVE ou BGREWRITEAOF
no-appendfsync-on-rewrite no

# Réécriture automatique de l'AOF
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# ----------------------------------------------------------------------------
# LOGGING
# ----------------------------------------------------------------------------

# Niveau de log
# Options : debug, verbose, notice, warning
loglevel notice

# Fichier de log (vide = stdout)
logfile ""

# ----------------------------------------------------------------------------
# BASES DE DONNÉES
# ----------------------------------------------------------------------------

# Nombre de bases de données (0 à databases-1)
databases 16

# ----------------------------------------------------------------------------
# REPLICATION (pour le mode esclave)
# ----------------------------------------------------------------------------

# Désactivé par défaut
# replicaof <masterip> <masterport>

# ----------------------------------------------------------------------------
# PERFORMANCES
# ----------------------------------------------------------------------------

# Nombre de threads pour les opérations I/O (Redis 6+)
# 0 = auto-détection
io-threads 4

# Active le threading pour les lectures
io-threads-do-reads yes

# Limite de mémoire pour les clients
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# ----------------------------------------------------------------------------
# OPTIMISATIONS MÉMOIRE
# ----------------------------------------------------------------------------

# Fréquence de la tâche de fond (Hz)
# Plus élevé = plus de CPU mais meilleure réactivité
# Recommandé : entre 10 et 100
hz 10

# Active la défragmentation active de la mémoire (Redis 4+)
activedefrag yes

# ----------------------------------------------------------------------------
# OPTIONS AVANCÉES
# ----------------------------------------------------------------------------

# Active les notifications d'événements (clés expirées, etc.)
# Options : K=keyspace, E=keyevent, g=generic, $=string, etc.
notify-keyspace-events ""

# Compression des listes et hashs
list-max-ziplist-size -2
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```

### **Étape 3 : Créer le fichier `docker-compose.yml`**

```yaml
version: '3.8'

services:
  redis:
    # Image officielle Redis
    image: redis:7-alpine

    # Nom du conteneur
    container_name: redis_configured

    # Redémarrage automatique
    restart: unless-stopped

    # Utiliser notre fichier de configuration
    command: redis-server /usr/local/etc/redis/redis.conf

    # Exposition du port
    ports:
      - "6379:6379"

    # Volumes
    volumes:
      # Monter le fichier de configuration
      - ./redis.conf:/usr/local/etc/redis/redis.conf:ro
      # Persistance des données
      - redis_data:/data
      # Logs (optionnel)
      - ./logs:/var/log/redis

    # Variables d'environnement (optionnel)
    environment:
      - TZ=Europe/Paris

volumes:
  redis_data:
```

**Note sur les volumes :**
- `:ro` = read-only (Redis ne peut pas modifier le fichier de config)
- `redis_data:/data` = Volume Docker pour la persistance
- `./logs` = Dossier local pour les logs (créez-le avant : `mkdir logs`)

### **Étape 4 : Démarrer Redis**

```bash
# Créer le dossier pour les logs
mkdir logs

# Démarrer Redis
docker-compose up -d

# Vérifier les logs
docker-compose logs -f
```

---

## 🔐 Section Sécurité

### **1. Authentification par mot de passe**

```bash
# Mot de passe simple (Redis < 6)
requirepass VotreMotDePasseTresTresSecurise!

# ⚠️ Utilisez un mot de passe fort :
# - Au moins 32 caractères
# - Lettres, chiffres, symboles
# - Généré aléatoirement
```

**Générer un mot de passe fort :**
```bash
# Linux/macOS
openssl rand -base64 32

# Windows PowerShell
-join ((65..90) + (97..122) + (48..57) | Get-Random -Count 32 | % {[char]$_})
```

### **2. Limitation des adresses IP**

```bash
# Écouter uniquement sur localhost
bind 127.0.0.1

# Écouter sur localhost et une IP spécifique
bind 127.0.0.1 192.168.1.100

# Écouter sur toutes les interfaces (⚠️ dangereux !)
bind 0.0.0.0
```

### **3. Mode protégé**

```bash
# Active la protection (recommandé)
protected-mode yes
```

Avec `protected-mode yes`, Redis refuse les connexions externes si :
- Aucun `bind` n'est défini
- Aucun mot de passe n'est configuré

### **4. Désactivation de commandes dangereuses**

```bash
# Désactiver complètement une commande
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command KEYS ""
rename-command CONFIG ""
rename-command SHUTDOWN ""
rename-command BGREWRITEAOF ""
rename-command BGSAVE ""
rename-command SAVE ""
rename-command DEBUG ""

# Renommer une commande (secret)
rename-command CONFIG "CONFIG_SECRET_abc123"
```

**Utilisation d'une commande renommée :**
```redis
# Au lieu de : CONFIG GET requirepass
CONFIG_SECRET_abc123 GET requirepass
```

### **5. Timeout des connexions inactives**

```bash
# Fermer les connexions après 5 minutes d'inactivité (300 secondes)
timeout 300

# 0 = désactivé (connexions persistantes)
timeout 0
```

---

## 💾 Section Persistance

Redis propose **deux méthodes** de persistance :

### **1. RDB (Redis Database) - Snapshots**

**Avantages :**
- ✅ Fichiers compacts
- ✅ Rapide à charger
- ✅ Idéal pour les backups

**Inconvénients :**
- ❌ Peut perdre des données (entre deux snapshots)
- ❌ Peut bloquer temporairement Redis (grandes BDD)

**Configuration :**
```bash
# Sauvegardes automatiques
save 900 1      # Toutes les 15 min si au moins 1 modification
save 300 10     # Toutes les 5 min si au moins 10 modifications
save 60 10000   # Toutes les 1 min si au moins 10 000 modifications

# Pour désactiver les snapshots automatiques
# save ""

# Options
stop-writes-on-bgsave-error yes    # Arrêter les écritures si backup échoue
rdbcompression yes                  # Compresser les fichiers RDB
rdbchecksum yes                     # Vérifier l'intégrité
dbfilename dump.rdb                 # Nom du fichier
dir /data                           # Dossier de sauvegarde
```

**Déclenchement manuel :**
```redis
# Sauvegarde bloquante (attend la fin)
SAVE

# Sauvegarde en arrière-plan (non bloquante)
BGSAVE
```

### **2. AOF (Append Only File) - Log des écritures**

**Avantages :**
- ✅ Perte de données minimale
- ✅ Fichier en texte lisible
- ✅ Récupération automatique en cas de corruption

**Inconvénients :**
- ❌ Fichiers plus volumineux
- ❌ Plus lent que RDB
- ❌ Plus d'I/O disque

**Configuration :**
```bash
# Activer AOF
appendonly yes

# Nom du fichier
appendfilename "appendonly.aof"

# Fréquence de synchronisation
appendfsync everysec   # Recommandé (bon compromis)
# appendfsync always   # Très sûr mais très lent
# appendfsync no       # Rapide mais risqué

# Réécriture automatique de l'AOF (compactage)
auto-aof-rewrite-percentage 100   # Réécrire si 100% plus gros
auto-aof-rewrite-min-size 64mb    # Minimum 64 MB avant réécriture
```

**Déclenchement manuel de la réécriture :**
```redis
BGREWRITEAOF
```

### **3. Quelle méthode choisir ?**

| Scénario | Recommandation |
|----------|---------------|
| **Cache** (perte acceptable) | RDB uniquement |
| **Sessions** (perte minimale) | AOF avec `appendfsync everysec` |
| **Données critiques** | RDB + AOF (les deux) |
| **Maximum de performances** | RDB uniquement |
| **Maximum de sécurité** | AOF avec `appendfsync always` |

**Configuration recommandée (compromis) :**
```bash
# RDB pour les backups
save 900 1
save 300 10
save 60 10000

# AOF pour la durabilité
appendonly yes
appendfsync everysec
```

---

## 🚀 Section Performances

### **1. Limites de mémoire**

```bash
# Définir une limite de mémoire (exemple : 2 GB)
maxmemory 2gb

# Politique quand la limite est atteinte
maxmemory-policy allkeys-lru

# Échantillon pour LRU (plus élevé = plus précis mais plus lent)
maxmemory-samples 5
```

**Politiques d'éviction :**

| Politique | Description | Cas d'usage |
|-----------|-------------|-------------|
| `noeviction` | Retourne une erreur | Production avec monitoring |
| `allkeys-lru` | Évince les clés LRU | Cache général |
| `allkeys-lfu` | Évince les clés LFU | Cache avec patterns d'accès |
| `allkeys-random` | Évince aléatoirement | Tests, développement |
| `volatile-lru` | Évince LRU parmi clés avec TTL | Sessions avec expiration |
| `volatile-lfu` | Évince LFU parmi clés avec TTL | Données temporaires |
| `volatile-ttl` | Évince clés avec TTL le plus court | Files d'attente |
| `volatile-random` | Évince aléatoirement parmi TTL | Tests spécifiques |

### **2. Threading (Redis 6+)**

```bash
# Activer le multi-threading I/O
io-threads 4

# Activer pour les lectures aussi (pas seulement les écritures)
io-threads-do-reads yes
```

**Recommandations :**
- 2 threads pour machines à 2-4 cœurs
- 4 threads pour machines à 4-8 cœurs
- 6+ threads pour serveurs puissants

### **3. Optimisations diverses**

```bash
# Fréquence des tâches de fond (10-100 Hz)
hz 10

# Défragmentation active de la mémoire
activedefrag yes

# TCP backlog (nombre de connexions en attente)
tcp-backlog 511

# Désactiver Transparent Huge Pages (Linux)
# (Nécessite configuration système)
```

### **4. Buffers de sortie clients**

```bash
# Limite pour les clients normaux
client-output-buffer-limit normal 0 0 0

# Limite pour les réplicas (esclaves)
client-output-buffer-limit replica 256mb 64mb 60

# Limite pour les canaux pub/sub
client-output-buffer-limit pubsub 32mb 8mb 60
```

**Format :** `<hard-limit> <soft-limit> <soft-seconds>`
- Si `hard-limit` atteinte : déconnexion immédiate
- Si `soft-limit` atteinte pendant `soft-seconds` : déconnexion

---

## 📊 Section Logging et Monitoring

### **1. Niveau de log**

```bash
# Niveau de détail
loglevel notice

# Options :
# - debug : Très verbeux (développement uniquement)
# - verbose : Beaucoup d'informations
# - notice : Informations modérées (recommandé)
# - warning : Seulement les avertissements
```

### **2. Fichier de log**

```bash
# Écrire dans un fichier
logfile "/var/log/redis/redis.log"

# Écrire sur stdout (défaut Docker)
logfile ""
```

### **3. Slow log (requêtes lentes)**

```bash
# Enregistrer les requêtes qui prennent plus de X microsecondes
slowlog-log-slower-than 10000   # 10 ms

# Nombre maximum d'entrées dans le slow log
slowlog-max-len 128
```

**Consulter le slow log :**
```redis
# Voir les 10 dernières requêtes lentes
SLOWLOG GET 10

# Voir le nombre d'entrées
SLOWLOG LEN

# Réinitialiser le slow log
SLOWLOG RESET
```

### **4. Latency monitoring**

```bash
# Seuil de latence en millisecondes
latency-monitor-threshold 100
```

**Consulter les latences :**
```redis
LATENCY LATEST
LATENCY HISTORY command
LATENCY DOCTOR
```

---

## 🔧 Commandes de gestion de la configuration

### **Voir la configuration actuelle**

```bash
# Se connecter
docker exec -it redis_configured redis-cli -a VotreMotDePasse

# Voir toute la configuration
CONFIG GET *

# Voir un paramètre spécifique
CONFIG GET maxmemory
CONFIG GET requirepass
CONFIG GET save
```

### **Modifier la configuration à chaud**

```redis
# Changer un paramètre (temporaire, perdu au redémarrage)
CONFIG SET maxmemory 512mb
CONFIG SET loglevel debug

# Sauvegarder la config actuelle dans redis.conf
CONFIG REWRITE
```

⚠️ **Attention :** `CONFIG REWRITE` ne fonctionne que si Redis a les droits d'écriture sur `redis.conf`.

### **Recharger la configuration**

Pour appliquer les modifications du fichier `redis.conf` :

```bash
# Méthode 1 : Redémarrer le conteneur
docker-compose restart

# Méthode 2 : Reload à chaud (limité)
docker exec redis_configured redis-cli -a VotreMotDePasse CONFIG REWRITE
```

---

## 📋 Template redis.conf minimal

Pour démarrer rapidement, voici une configuration minimale mais solide :

```bash
# Configuration Redis minimale pour développement

# Réseau
bind 127.0.0.1
port 6379
timeout 300

# Sécurité
requirepass ChangeMe123456789!
protected-mode yes
rename-command FLUSHALL ""
rename-command FLUSHDB ""

# Mémoire
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistance
save 900 1
save 300 10
save 60 10000
dbfilename dump.rdb
dir /data

# Logging
loglevel notice
logfile ""

# Performances
hz 10
databases 16
```

---

## 🧪 Tester la configuration

### **1. Vérifier que Redis démarre**

```bash
docker-compose up -d
docker-compose logs
```

Recherchez :
```
Ready to accept connections
```

### **2. Vérifier les paramètres**

```bash
docker exec redis_configured redis-cli -a VotreMotDePasse CONFIG GET requirepass
docker exec redis_configured redis-cli -a VotreMotDePasse CONFIG GET maxmemory
docker exec redis_configured redis-cli -a VotreMotDePasse CONFIG GET save
```

### **3. Tester la persistance**

```bash
# Se connecter
docker exec -it redis_configured redis-cli -a VotreMotDePasse

# Stocker des données
SET test:persistence "Ceci est un test"
SET test:nombre 42

# Forcer une sauvegarde
BGSAVE

# Vérifier que la sauvegarde est terminée
LASTSAVE

# Redémarrer le conteneur
docker-compose restart

# Se reconnecter et vérifier les données
docker exec -it redis_configured redis-cli -a VotreMotDePasse
GET test:persistence
# Résultat : "Ceci est un test"
```

### **4. Tester les limites de mémoire**

```bash
# Voir la mémoire utilisée
INFO memory

# Voir la politique d'éviction
CONFIG GET maxmemory-policy
```

---

## ❓ Questions fréquentes

### **Où trouver le fichier redis.conf par défaut ?**

Téléchargez-le depuis GitHub :
```bash
wget https://raw.githubusercontent.com/redis/redis/7.0/redis.conf
```

Ou copiez-le depuis le conteneur :
```bash
docker run --rm redis:7-alpine cat /etc/redis/redis.conf > redis.conf
```

### **Comment savoir si ma config est correcte ?**

```bash
# Redis affiche des erreurs au démarrage si la config est invalide
docker-compose logs | grep -i error
docker-compose logs | grep -i warning
```

### **Puis-je avoir plusieurs fichiers de configuration ?**

Oui, avec `include` :
```bash
# Dans redis.conf
include /usr/local/etc/redis/redis-base.conf
include /usr/local/etc/redis/redis-security.conf
```

### **CONFIG REWRITE ne fonctionne pas, pourquoi ?**

Deux raisons possibles :
1. Le fichier `redis.conf` est monté en lecture seule (`:ro`)
2. Redis n'a pas les permissions d'écriture

Solution : Montez sans `:ro` :
```yaml
volumes:
  - ./redis.conf:/usr/local/etc/redis/redis.conf
```

### **Comment sauvegarder ma configuration ?**

```bash
# Copier le fichier de config
cp redis.conf redis.conf.backup

# Ou exporter la config actuelle
docker exec redis_configured redis-cli -a VotreMotDePasse CONFIG GET '*' > config-backup.txt
```

### **Redis ignore certains paramètres de mon redis.conf, pourquoi ?**

Vérifiez :
1. Les commentaires (`#`) ne sont pas à l'intérieur des valeurs
2. Pas d'espaces en début de ligne
3. Le format est correct (pas de guillemets sauf si nécessaire)
4. Les unités sont valides (k, kb, m, mb, g, gb)

---

## 🛡️ Configuration de production vs développement

### **Développement**

```bash
# Sécurité light
requirepass SimplePassword123
protected-mode yes

# Persistance basique
save 900 1
appendonly no

# Logs verbeux
loglevel debug

# Pas de limite de mémoire
maxmemory 0
```

### **Production**

```bash
# Sécurité renforcée
requirepass GeneratedPassword-32chars-Random!
protected-mode yes
bind 10.0.1.5   # IP spécifique
rename-command FLUSHALL ""
rename-command CONFIG "SECRET_CONFIG_xyz"

# Persistance robuste
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec

# Logs modérés
loglevel notice

# Limite de mémoire stricte
maxmemory 4gb
maxmemory-policy allkeys-lru

# Monitoring
slowlog-log-slower-than 10000
latency-monitor-threshold 100
```

---

## 📚 Ressources complémentaires

- 📖 [Documentation redis.conf officielle](https://redis.io/docs/management/config/)
- 📄 [Fichier redis.conf exemple complet](https://raw.githubusercontent.com/redis/redis/7.0/redis.conf)
- 🔧 [Redis Configuration Generator](https://redis.io/docs/manual/config/)
- 📊 [Redis Best Practices](https://redis.io/docs/manual/patterns/)

---

## 🎉 Récapitulatif

Vous savez maintenant :
- ✅ Créer et personnaliser un fichier `redis.conf` complet
- ✅ Configurer la sécurité (mots de passe, IP, commandes)
- ✅ Choisir et paramétrer la persistance (RDB, AOF)
- ✅ Optimiser les performances (mémoire, threading, buffers)
- ✅ Configurer le logging et le monitoring
- ✅ Appliquer des configurations différentes selon l'environnement
- ✅ Tester et valider votre configuration

**Prochaines étapes :**
- 🌐 [6.4 - Configuration avec IP fixe](04-config-ip-fixe.md)
- 🖥️ [6.5 - Redis avec Redis Commander (GUI)](05-redis-commander.md)
- 📚 [Annexe D - Sécurité et bonnes pratiques](../annexes/D-securite-bonnes-pratiques.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
