# Configuration de MariaDB avec fichier my.cnf

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous apprend à personnaliser la configuration de votre serveur MariaDB en utilisant un fichier `my.cnf`. Ce fichier permet de modifier le comportement de MariaDB sans avoir à reconstruire le conteneur.

**Ce que vous allez apprendre :**
- Comprendre le rôle du fichier `my.cnf`
- Créer et structurer votre fichier de configuration personnalisé
- Intégrer le fichier `my.cnf` à Docker Compose
- Modifier des paramètres courants (encodage, performances, fonctionnalités)
- Vérifier que vos modifications sont appliquées

**Durée estimée :** 15 minutes

---

## 🎯 Prérequis

Avant de commencer, vous devez :

- ✅ Avoir suivi la [fiche 1.1 - Configuration basique](01-config-basique-docker-compose.md)
- ✅ Comprendre les bases de Docker Compose
- ✅ Savoir éditer des fichiers texte

---

## 🤔 Pourquoi utiliser un fichier my.cnf ?

### Sans fichier my.cnf
Vous êtes limité aux paramètres par défaut de MariaDB ou devez passer des arguments dans la ligne de commande.

### Avec fichier my.cnf
Vous pouvez :
- ✅ Activer/désactiver des fonctionnalités (comme l'Event Scheduler)
- ✅ Modifier les limites de ressources (mémoire, connexions)
- ✅ Configurer l'encodage des caractères
- ✅ Ajuster les performances selon vos besoins
- ✅ Partager votre configuration avec votre équipe

---

## 📝 Étape 1 : Créer la structure du projet

### 1.1 Créer le dossier du projet

```bash
# Créer le dossier principal
mkdir mariadb-mycnf

# Se déplacer dans le dossier
cd mariadb-mycnf

# Créer un sous-dossier pour les données (optionnel mais recommandé)
mkdir data
```

Votre structure sera :
```
mariadb-mycnf/
├── docker-compose.yml    (à créer)
├── my.cnf                (à créer)
└── data/                 (pour les données)
```

---

## 📄 Étape 2 : Créer le fichier my.cnf

### 2.1 Créer le fichier

Créez un fichier nommé `my.cnf` dans votre dossier `mariadb-mycnf` avec le contenu suivant :

```ini
# ==========================================
# Configuration personnalisée MariaDB
# ==========================================

[mysqld]
# Section pour le serveur MariaDB

# ----- Encodage des caractères -----
# UTF-8 pour supporter tous les caractères (émojis, accents, etc.)
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# ----- Event Scheduler (Tâches planifiées) -----
# Active le planificateur d'événements pour les tâches récurrentes
# Valeurs possibles : ON, OFF, DISABLED
event_scheduler = ON

# ----- Limites de connexions -----
# Nombre maximum de connexions simultanées
max_connections = 200

# ----- Tailles de buffer (Performances) -----
# Mémoire allouée pour les requêtes de tri
sort_buffer_size = 2M

# Mémoire pour les jointures
join_buffer_size = 2M

# ----- Logs (Débogage et monitoring) -----
# Active les logs généraux (toutes les requêtes SQL)
# ⚠️ À désactiver en production (impact performances)
# general_log = ON
# general_log_file = /var/log/mysql/general.log

# Active les logs des requêtes lentes (>2 secondes)
slow_query_log = ON
slow_query_log_file = /var/log/mysql/slow-query.log
long_query_time = 2

# ----- Timezone -----
# Définir le fuseau horaire du serveur
default-time-zone = '+01:00'

[mysql]
# Section pour le client MySQL (commande mysql)

# Encodage par défaut pour le client
default-character-set = utf8mb4

[client]
# Section pour tous les clients MariaDB

# Port par défaut
port = 3306

# Encodage par défaut
default-character-set = utf8mb4
```

### 2.2 Comprendre la structure du fichier

Le fichier `my.cnf` est organisé en **sections** délimitées par `[nom_section]` :

| Section | Description |
|---------|-------------|
| `[mysqld]` | Configuration du serveur MariaDB (la plus importante) |
| `[mysql]` | Configuration du client en ligne de commande `mysql` |
| `[client]` | Configuration pour tous les clients (HeidiSQL, DBeaver, etc.) |

**Format des paramètres :**
```ini
# Commentaire (ligne ignorée)
nom_parametre = valeur
```

### 2.3 Paramètres expliqués

#### 🔤 Encodage des caractères

```ini
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

- `utf8mb4` : Encodage UTF-8 complet (supporte les émojis 😊)
- `utf8mb4_unicode_ci` : Règles de comparaison insensibles à la casse

**Pourquoi c'est important ?** Sans cela, vous pourriez avoir des problèmes d'affichage avec les caractères spéciaux.

#### ⏰ Event Scheduler

```ini
event_scheduler = ON
```

Active le planificateur d'événements qui permet de créer des tâches SQL automatiques (comme les CRON).

**Exemple d'usage :**
```sql
-- Créer un événement qui nettoie les vieilles données chaque jour
CREATE EVENT nettoyer_logs
ON SCHEDULE EVERY 1 DAY
DO
  DELETE FROM logs WHERE date < DATE_SUB(NOW(), INTERVAL 30 DAY);
```

#### 🔌 Limites de connexions

```ini
max_connections = 200
```

Nombre maximum de clients pouvant se connecter simultanément. Augmentez si vous avez beaucoup d'utilisateurs.

#### 🚀 Buffers de performance

```ini
sort_buffer_size = 2M
join_buffer_size = 2M
```

Mémoire allouée pour certaines opérations. Des valeurs plus élevées = meilleures performances pour les grosses requêtes.

**⚠️ Attention :** Ne mettez pas des valeurs trop élevées si vous avez peu de RAM !

#### 📝 Logs des requêtes lentes

```ini
slow_query_log = ON
long_query_time = 2
```

Enregistre toutes les requêtes qui prennent plus de 2 secondes. Utile pour identifier les problèmes de performances.

---

## 🐳 Étape 3 : Créer le docker-compose.yml

Créez un fichier `docker-compose.yml` dans le même dossier :

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_custom_config
    restart: unless-stopped

    environment:
      # Mot de passe root (⚠️ CHANGEZ-LE !)
      MYSQL_ROOT_PASSWORD: mon_mot_de_passe_securise

      # Base de données initiale (optionnel)
      MYSQL_DATABASE: ma_base_dev

      # Utilisateur supplémentaire (optionnel)
      MYSQL_USER: dev_user
      MYSQL_PASSWORD: mot_de_passe_user

    ports:
      - "3306:3306"

    volumes:
      # Données persistantes (bind mount vers dossier local)
      - ./data:/var/lib/mysql

      # 🔑 LIGNE CLÉ : Montage du fichier my.cnf
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf

    # Vérification de santé (optionnel mais recommandé)
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 3
```

### 📖 Point clé : Montage du fichier de configuration

```yaml
volumes:
  - ./my.cnf:/etc/mysql/conf.d/custom.cnf
```

**Explication :**
- `./my.cnf` : Fichier sur votre machine hôte
- `:` : Séparateur
- `/etc/mysql/conf.d/custom.cnf` : Emplacement dans le conteneur

MariaDB lit automatiquement **tous les fichiers `.cnf`** dans `/etc/mysql/conf.d/`. Votre fichier sera donc pris en compte !

**💡 Pourquoi "custom.cnf" et pas "my.cnf" ?**
Pour éviter les conflits avec le `my.cnf` principal. Vous pouvez utiliser n'importe quel nom avec l'extension `.cnf`.

---

## ▶️ Étape 4 : Démarrer MariaDB avec la configuration

### 4.1 Lancer le conteneur

```bash
# Depuis le dossier mariadb-mycnf
docker-compose up -d
```

### 4.2 Vérifier les logs

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Vous devriez voir des lignes comme :
# mariadb_custom_config | [...] Event scheduler loaded successfully
```

Appuyez sur `Ctrl+C` pour quitter l'affichage des logs.

---

## ✅ Étape 5 : Vérifier que la configuration est appliquée

### 5.1 Se connecter à MariaDB

```bash
docker exec -it mariadb_custom_config mariadb -u root -p
```

Entrez votre mot de passe root.

### 5.2 Vérifier l'encodage

```sql
-- Afficher la configuration des caractères
SHOW VARIABLES LIKE 'character_set%';
SHOW VARIABLES LIKE 'collation%';
```

Vous devriez voir `utf8mb4` pour les variables importantes.

### 5.3 Vérifier l'Event Scheduler

```sql
-- Vérifier si l'Event Scheduler est activé
SHOW VARIABLES LIKE 'event_scheduler';
```

Résultat attendu :
```
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| event_scheduler | ON    |
+-----------------+-------+
```

### 5.4 Vérifier d'autres paramètres

```sql
-- Nombre maximum de connexions
SHOW VARIABLES LIKE 'max_connections';

-- Taille des buffers
SHOW VARIABLES LIKE '%buffer%';

-- Timezone
SHOW VARIABLES LIKE 'time_zone';

-- Quitter
EXIT;
```

---

## 🔄 Étape 6 : Modifier la configuration

### 6.1 Éditer le fichier my.cnf

Si vous souhaitez modifier un paramètre, éditez simplement le fichier `my.cnf` sur votre machine.

**Exemple :** Augmenter le nombre de connexions max

```ini
[mysqld]
max_connections = 500  # Avant : 200
```

### 6.2 Redémarrer le conteneur

Pour appliquer les modifications :

```bash
docker-compose restart
```

### 6.3 Vérifier la nouvelle valeur

```bash
docker exec -it mariadb_custom_config mariadb -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"
```

---

## 🎨 Exemples de configurations courantes

### Configuration pour le développement

```ini
[mysqld]
# Encodage complet
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Event Scheduler activé
event_scheduler = ON

# Logs de débogage activés
general_log = ON
slow_query_log = ON
long_query_time = 1

# Connexions suffisantes pour les tests
max_connections = 100
```

### Configuration orientée performance

```ini
[mysqld]
# Encodage
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Buffers augmentés
innodb_buffer_pool_size = 512M
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 2M

# Plus de connexions
max_connections = 500

# Pas de logs généraux (performances)
general_log = OFF
slow_query_log = ON
long_query_time = 2
```

### Configuration minimaliste

```ini
[mysqld]
# Juste l'essentiel
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
event_scheduler = ON
```

---

## 🛑 Étape 7 : Gérer le conteneur

### Commandes courantes

```bash
# Arrêter
docker-compose stop

# Redémarrer
docker-compose restart

# Voir les logs
docker-compose logs -f

# Arrêter et supprimer (données conservées)
docker-compose down

# Supprimer tout (⚠️ SUPPRIME LES DONNÉES)
docker-compose down
rm -rf ./data
```

---

## 🧹 Nettoyage complet

Pour tout supprimer proprement :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer les données (⚠️ IRRÉVERSIBLE)
rm -rf ./data

# 3. Supprimer les fichiers de config (optionnel)
rm docker-compose.yml my.cnf
```

---

## ✅ Récapitulatif

Vous avez appris à :

- ✅ Créer un fichier `my.cnf` personnalisé
- ✅ Comprendre la structure des sections `[mysqld]`, `[mysql]`, `[client]`
- ✅ Monter le fichier de configuration dans Docker Compose
- ✅ Modifier et appliquer des paramètres courants
- ✅ Vérifier que vos modifications sont prises en compte
- ✅ Adapter la configuration selon vos besoins (dev, performance)

**Fichiers créés :**
- `docker-compose.yml` : Configuration Docker
- `my.cnf` : Configuration personnalisée MariaDB
- `data/` : Données persistantes (bind mount)

---

## 🚀 Prochaines étapes

Maintenant que vous maîtrisez la configuration personnalisée, vous pouvez explorer :

- **[1.3 Configuration avec IP fixe](03-config-ip-fixe.md)** - Assigner une adresse IP statique
- **[1.4 Gestion des utilisateurs](04-gestion-utilisateurs.md)** - Créer et gérer les utilisateurs SQL
- **[1.5 Accès réseau local](05-acces-reseau-local.md)** - Accéder depuis d'autres machines

---

## 📚 Ressources complémentaires

- [Documentation MariaDB - Options du serveur](https://mariadb.com/kb/en/server-system-variables/)
- [Documentation MariaDB - my.cnf](https://mariadb.com/kb/en/configuring-mariadb-with-option-files/)
- [Documentation Docker - Volumes](https://docs.docker.com/storage/volumes/)

---

## ❓ FAQ - Questions fréquentes

**Q : Mes modifications ne sont pas prises en compte, pourquoi ?**
R : Vérifiez que :
1. Le fichier `my.cnf` est bien monté (vérifiez le chemin dans `volumes:`)
2. Vous avez redémarré le conteneur (`docker-compose restart`)
3. Il n'y a pas d'erreur de syntaxe dans `my.cnf` (pas de faute de frappe)

**Q : Comment voir tous les paramètres disponibles ?**
R : Connectez-vous à MariaDB et exécutez :
```sql
SHOW VARIABLES;
```

**Q : Puis-je avoir plusieurs fichiers .cnf ?**
R : Oui ! Montez plusieurs fichiers dans `/etc/mysql/conf.d/` :
```yaml
volumes:
  - ./my.cnf:/etc/mysql/conf.d/custom.cnf
  - ./performance.cnf:/etc/mysql/conf.d/performance.cnf
```

**Q : Les paramètres dans my.cnf écrasent-ils ceux par défaut ?**
R : Oui, les fichiers dans `/etc/mysql/conf.d/` ont priorité sur `/etc/mysql/my.cnf` (configuration par défaut).

**Q : Comment savoir si un paramètre peut être modifié à chaud ?**
R : Consultez la [documentation MariaDB](https://mariadb.com/kb/en/server-system-variables/). Les paramètres marqués "Dynamic" peuvent être changés sans redémarrage via `SET GLOBAL`.

**Q : Quelle est la différence entre utf8 et utf8mb4 ?**
R : `utf8mb4` est la version complète d'UTF-8 qui supporte les émojis et caractères sur 4 octets. Utilisez toujours `utf8mb4` !

**Q : Le fichier my.cnf fonctionne-t-il pour MySQL aussi ?**
R : Oui ! MariaDB et MySQL utilisent le même format de fichier de configuration. La plupart des paramètres sont compatibles.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
