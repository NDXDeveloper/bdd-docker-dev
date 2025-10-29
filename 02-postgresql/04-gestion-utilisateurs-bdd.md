# PostgreSQL - Gestion des Utilisateurs et Bases de Données

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans PostgreSQL, la gestion des utilisateurs et des bases de données est cruciale pour la sécurité et l'organisation de vos projets. Contrairement à MySQL/MariaDB, PostgreSQL a un système de permissions plus strict et plus granulaire.

Cette fiche vous guide pour créer et gérer des utilisateurs, des bases de données, et configurer les permissions de manière sécurisée.

**Ce que vous allez apprendre :**
- Comprendre les rôles et utilisateurs PostgreSQL
- Créer des bases de données
- Créer des utilisateurs avec différents privilèges
- Gérer les permissions (GRANT/REVOKE)
- Configurer des utilisateurs par projet
- Bonnes pratiques de sécurité

**Durée estimée :** 20-25 minutes

---

## 🎯 Objectif

À la fin de cette fiche, vous aurez :
- ✅ Créé plusieurs bases de données
- ✅ Créé des utilisateurs avec des rôles spécifiques
- ✅ Configuré des permissions appropriées
- ✅ Compris le principe du moindre privilège
- ✅ Sécurisé votre environnement PostgreSQL

---

## 📦 Prérequis

Avant de commencer :
- ✅ PostgreSQL installé et fonctionnel ([Configuration basique](01-config-basique-docker-compose.md))
- ✅ Capacité de se connecter en tant que super-utilisateur
- ✅ Comprendre les bases de SQL

---

## 🧠 Concepts Fondamentaux

### Utilisateurs vs Rôles dans PostgreSQL

**Important** : Dans PostgreSQL, **utilisateurs** et **rôles** sont pratiquement synonymes.

| Terme | Définition | Différence |
|-------|------------|------------|
| **Rôle** | Entité pouvant posséder objets et permissions | Terme générique |
| **Utilisateur** | Rôle avec le privilège LOGIN | Peut se connecter |
| **Groupe** | Rôle sans LOGIN, contient d'autres rôles | Pour organiser |

**En pratique :**
```sql
-- Ces deux commandes sont équivalentes
CREATE USER alice WITH PASSWORD 'pass123';
CREATE ROLE alice WITH LOGIN PASSWORD 'pass123';
```

### Hiérarchie des Utilisateurs

```
postgres (super-utilisateur)
    ├── Propriétaire de base 1 (admin de la base)
    │   ├── Utilisateur lecture seule
    │   └── Utilisateur lecture/écriture
    │
    └── Propriétaire de base 2 (admin de la base)
        ├── Utilisateur application
        └── Utilisateur de test
```

### Bases de Données par Défaut

Lors de l'installation, PostgreSQL crée automatiquement :

| Base | Description | À Supprimer ? |
|------|-------------|---------------|
| `postgres` | Base système par défaut | ❌ Non |
| `template0` | Template "propre", jamais modifié | ❌ Non |
| `template1` | Template pour nouvelles bases | ❌ Non (mais modifiable) |

**Vos bases personnalisées** s'ajoutent à cette liste.

---

## 🗄️ Gestion des Bases de Données

### Se Connecter à PostgreSQL

```bash
# En tant que super-utilisateur (défini dans docker-compose.yml)
docker exec -it postgres_dev psql -U mon_utilisateur -d postgres
```

**Invite de commande attendue :**
```
postgres=#
```

Le `#` indique que vous êtes super-utilisateur.

### Créer une Base de Données

```sql
-- Syntaxe basique
CREATE DATABASE ma_nouvelle_base;

-- Avec propriétaire spécifique
CREATE DATABASE blog_db
    OWNER alice
    ENCODING 'UTF8'
    LC_COLLATE = 'en_US.utf8'
    LC_CTYPE = 'en_US.utf8'
    TEMPLATE = template0;
```

**Paramètres expliqués :**

| Paramètre | Description | Valeur Recommandée |
|-----------|-------------|-------------------|
| `OWNER` | Propriétaire de la base | Utilisateur applicatif |
| `ENCODING` | Encodage des caractères | `UTF8` (Unicode) |
| `LC_COLLATE` | Ordre de tri | `en_US.utf8` ou `fr_FR.utf8` |
| `LC_CTYPE` | Classification des caractères | `en_US.utf8` ou `fr_FR.utf8` |
| `TEMPLATE` | Base modèle | `template0` (propre) |

### Lister les Bases de Données

```sql
-- Commande méta psql
\l

-- Ou en SQL
SELECT datname, datdba::regrole AS owner, encoding
FROM pg_database
WHERE datistemplate = false;
```

**Résultat :**
```
    datname     |  owner   | encoding
----------------+----------+----------
 postgres       | postgres |        6
 ma_base        | alice    |        6
 blog_db        | alice    |        6
```

### Se Connecter à une Base Spécifique

```sql
-- Depuis psql
\c blog_db

-- Vous verrez
You are now connected to database "blog_db" as user "postgres".
blog_db=#
```

### Supprimer une Base de Données

```sql
-- ⚠️ ATTENTION : Action irréversible !
DROP DATABASE blog_db;

-- Avec vérification d'existence
DROP DATABASE IF EXISTS blog_db;
```

⚠️ **DANGER** : Toutes les données de la base sont définitivement perdues !

### Renommer une Base de Données

```sql
-- Vous devez être déconnecté de la base
\c postgres

-- Renommer
ALTER DATABASE ancienne_base RENAME TO nouvelle_base;
```

---

## 👥 Gestion des Utilisateurs (Rôles)

### Créer un Utilisateur

#### Utilisateur Simple (Lecture Seule)

```sql
-- Créer l'utilisateur
CREATE USER lecteur_blog WITH PASSWORD 'mot_de_passe_fort_123!';

-- Se connecter à la base concernée
\c blog_db

-- Accorder les permissions de lecture
GRANT CONNECT ON DATABASE blog_db TO lecteur_blog;
GRANT USAGE ON SCHEMA public TO lecteur_blog;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lecteur_blog;

-- Accorder SELECT automatiquement sur les futures tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO lecteur_blog;
```

#### Utilisateur Application (Lecture/Écriture)

```sql
-- Créer l'utilisateur
CREATE USER app_blog WITH PASSWORD 'app_pass_secure_456!';

-- Permissions complètes sur la base
\c blog_db
GRANT CONNECT ON DATABASE blog_db TO app_blog;
GRANT USAGE ON SCHEMA public TO app_blog;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_blog;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_blog;

-- Pour les futures tables et séquences
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_blog;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO app_blog;
```

**Explication des permissions :**

| Permission | Description | Use Case |
|------------|-------------|----------|
| `CONNECT` | Connexion à la base | Obligatoire |
| `USAGE` | Utilisation du schéma | Obligatoire pour accéder aux objets |
| `SELECT` | Lire les données | Lecture |
| `INSERT` | Ajouter des données | Création |
| `UPDATE` | Modifier les données | Modification |
| `DELETE` | Supprimer des données | Suppression |
| `TRUNCATE` | Vider une table | Nettoyage complet |
| `REFERENCES` | Créer des clés étrangères | Relations entre tables |
| `TRIGGER` | Créer des triggers | Automatisation |

#### Utilisateur Admin de Base (Pas Super-User)

```sql
-- Créer l'utilisateur avec privilèges étendus
CREATE USER admin_blog WITH PASSWORD 'admin_pass_789!' CREATEDB;

-- Faire de lui le propriétaire de la base
ALTER DATABASE blog_db OWNER TO admin_blog;

-- Il peut maintenant tout faire sur cette base (mais pas les autres)
```

#### Super-Utilisateur (À Éviter)

```sql
-- ⚠️ DANGEREUX : Tous les privilèges sur toutes les bases
CREATE USER super_admin WITH PASSWORD 'super_pass!' SUPERUSER;
```

⚠️ **À éviter** : Ne créez des super-utilisateurs qu'en cas de besoin absolu.

### Lister les Utilisateurs

```sql
-- Commande méta psql
\du

-- Ou en SQL (plus détaillé)
SELECT
    rolname AS username,
    rolsuper AS is_superuser,
    rolcreatedb AS can_create_db,
    rolcreaterole AS can_create_role,
    rolcanlogin AS can_login
FROM pg_roles
WHERE rolcanlogin = true
ORDER BY rolname;
```

**Résultat :**
```
    username     | is_superuser | can_create_db | can_create_role | can_login
-----------------+--------------+---------------+-----------------+-----------
 admin_blog      | f            | t             | f               | t
 app_blog        | f            | f             | f               | t
 lecteur_blog    | f            | f             | f               | t
 mon_utilisateur | t            | t             | t               | t
```

### Modifier un Utilisateur

```sql
-- Changer le mot de passe
ALTER USER app_blog WITH PASSWORD 'nouveau_mot_de_passe!';

-- Renommer
ALTER USER app_blog RENAME TO application_blog;

-- Ajouter des privilèges
ALTER USER lecteur_blog WITH CREATEDB;

-- Retirer des privilèges
ALTER USER lecteur_blog WITH NOCREATEDB;

-- Définir une date d'expiration
ALTER USER temp_user VALID UNTIL '2024-12-31';
```

### Supprimer un Utilisateur

```sql
-- Révoquer d'abord toutes les permissions
REVOKE ALL PRIVILEGES ON DATABASE blog_db FROM lecteur_blog;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM lecteur_blog;

-- Supprimer l'utilisateur
DROP USER lecteur_blog;

-- Ou avec vérification
DROP USER IF EXISTS lecteur_blog;
```

**⚠️ Attention** : Impossible de supprimer un utilisateur qui possède des objets (tables, vues...).

**Solution si l'utilisateur possède des objets :**
```sql
-- Voir ce qu'il possède
SELECT * FROM pg_tables WHERE tableowner = 'lecteur_blog';

-- Transférer la propriété à un autre utilisateur
REASSIGN OWNED BY lecteur_blog TO admin_blog;

-- Puis supprimer les dépendances restantes
DROP OWNED BY lecteur_blog;

-- Maintenant on peut supprimer
DROP USER lecteur_blog;
```

---

## 🔐 Gestion des Permissions (GRANT/REVOKE)

### Schéma de Permissions par Type d'Utilisateur

#### 1. Utilisateur Lecture Seule (Analyste, BI)

```sql
-- Créer l'utilisateur
CREATE USER analyst WITH PASSWORD 'analyst_pass!';

-- Connexion à la base
GRANT CONNECT ON DATABASE blog_db TO analyst;

-- Accès au schéma
GRANT USAGE ON SCHEMA public TO analyst;

-- Lecture uniquement
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;

-- Pour les futures tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO analyst;
```

**Ce qu'il peut faire :**
- ✅ Se connecter
- ✅ Lire toutes les tables
- ❌ Modifier des données
- ❌ Créer/supprimer des tables

#### 2. Utilisateur Application (Backend)

```sql
CREATE USER backend WITH PASSWORD 'backend_pass!';

\c blog_db
GRANT CONNECT ON DATABASE blog_db TO backend;
GRANT USAGE ON SCHEMA public TO backend;

-- CRUD complet (Create, Read, Update, Delete)
GRANT SELECT, INSERT, UPDATE, DELETE
    ON ALL TABLES IN SCHEMA public
    TO backend;

-- Séquences (pour les AUTO_INCREMENT)
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO backend;

-- Futures tables et séquences
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO backend;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO backend;
```

**Ce qu'il peut faire :**
- ✅ Se connecter
- ✅ Lire, créer, modifier, supprimer des lignes
- ✅ Utiliser les séquences (IDs auto)
- ❌ Créer/supprimer des tables
- ❌ Modifier la structure

#### 3. Utilisateur Migration/Déploiement (DevOps)

```sql
CREATE USER deployer WITH PASSWORD 'deployer_pass!';

-- Créer des bases de données
ALTER USER deployer WITH CREATEDB;

-- Permissions complètes sur une base spécifique
\c blog_db
GRANT ALL PRIVILEGES ON DATABASE blog_db TO deployer;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO deployer;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO deployer;

-- Peut créer des objets
GRANT CREATE ON SCHEMA public TO deployer;
```

**Ce qu'il peut faire :**
- ✅ Tout sur la base blog_db
- ✅ Créer/modifier/supprimer tables, index, vues
- ✅ Exécuter des migrations
- ❌ Toucher aux autres bases (isolation)

#### 4. Utilisateur Temporaire (Test, Stagiaire)

```sql
-- Avec date d'expiration
CREATE USER stagiaire WITH
    PASSWORD 'temp_pass!'
    VALID UNTIL '2024-12-31';

-- Permissions limitées
\c blog_db
GRANT CONNECT ON DATABASE blog_db TO stagiaire;
GRANT USAGE ON SCHEMA public TO stagiaire;
GRANT SELECT ON TABLE articles, comments TO stagiaire;
-- Accès uniquement à certaines tables
```

### Révoquer des Permissions

```sql
-- Révoquer la connexion
REVOKE CONNECT ON DATABASE blog_db FROM lecteur_blog;

-- Révoquer SELECT
REVOKE SELECT ON ALL TABLES IN SCHEMA public FROM lecteur_blog;

-- Révoquer INSERT/UPDATE/DELETE
REVOKE INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public FROM app_blog;

-- Révoquer TOUTES les permissions
REVOKE ALL PRIVILEGES ON DATABASE blog_db FROM lecteur_blog;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM lecteur_blog;
```

### Voir les Permissions d'un Utilisateur

```sql
-- Permissions sur les bases
\l+

-- Permissions sur les tables
\dp

-- Ou en SQL
SELECT
    table_schema,
    table_name,
    privilege_type,
    grantee
FROM information_schema.table_privileges
WHERE grantee = 'app_blog';
```

---

## 🎯 Cas Pratiques : Configuration par Projet

### Projet 1 : Blog avec 3 Utilisateurs

```sql
-- 1. Créer la base de données
CREATE DATABASE blog_db ENCODING 'UTF8';

-- 2. Se connecter à la base
\c blog_db

-- 3. Créer les utilisateurs
CREATE USER blog_admin WITH PASSWORD 'admin_blog_2024!';
CREATE USER blog_app WITH PASSWORD 'app_blog_2024!';
CREATE USER blog_readonly WITH PASSWORD 'read_blog_2024!';

-- 4. Admin : Propriétaire de la base
ALTER DATABASE blog_db OWNER TO blog_admin;

-- 5. Application : CRUD
GRANT CONNECT ON DATABASE blog_db TO blog_app;
GRANT USAGE ON SCHEMA public TO blog_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO blog_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO blog_app;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO blog_app;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT USAGE, SELECT ON SEQUENCES TO blog_app;

-- 6. Lecture seule : Analytics
GRANT CONNECT ON DATABASE blog_db TO blog_readonly;
GRANT USAGE ON SCHEMA public TO blog_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO blog_readonly;

ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO blog_readonly;
```

### Projet 2 : E-Commerce avec Isolation

```sql
-- Bases séparées par environnement
CREATE DATABASE shop_dev OWNER shop_admin;
CREATE DATABASE shop_test OWNER shop_admin;
CREATE DATABASE shop_prod OWNER shop_admin;

-- Utilisateurs par environnement
CREATE USER shop_dev_app WITH PASSWORD 'dev_pass!';
CREATE USER shop_test_app WITH PASSWORD 'test_pass!';
CREATE USER shop_prod_app WITH PASSWORD 'prod_pass_ultra_secure!';

-- Permissions dev (tous droits)
\c shop_dev
GRANT ALL PRIVILEGES ON DATABASE shop_dev TO shop_dev_app;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO shop_dev_app;

-- Permissions prod (limité)
\c shop_prod
GRANT CONNECT ON DATABASE shop_prod TO shop_prod_app;
GRANT USAGE ON SCHEMA public TO shop_prod_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO shop_prod_app;
-- Pas de DROP, TRUNCATE, ALTER
```

### Projet 3 : Multi-Tenant (Plusieurs Clients)

```sql
-- Une base par client
CREATE DATABASE client_acme OWNER admin_acme;
CREATE DATABASE client_globex OWNER admin_globex;

-- Utilisateurs isolés
CREATE USER app_acme WITH PASSWORD 'acme_pass!';
CREATE USER app_globex WITH PASSWORD 'globex_pass!';

-- app_acme ne peut accéder qu'à client_acme
\c client_acme
GRANT CONNECT ON DATABASE client_acme TO app_acme;
GRANT ALL ON SCHEMA public TO app_acme;

-- app_globex ne peut accéder qu'à client_globex
\c client_globex
GRANT CONNECT ON DATABASE client_globex TO app_globex;
GRANT ALL ON SCHEMA public TO app_globex;

-- Aucun accès croisé = isolation parfaite
```

---

## 🔒 Sécurité et Bonnes Pratiques

### Principe du Moindre Privilège

**Règle d'or** : N'accordez QUE les permissions nécessaires.

| ❌ Mauvaise Pratique | ✅ Bonne Pratique |
|---------------------|-------------------|
| Tout en super-utilisateur | Un utilisateur par rôle |
| `GRANT ALL` partout | Permissions spécifiques |
| Même mot de passe pour tous | Mot de passe unique par user |
| Pas de date d'expiration | Expiration pour comptes temporaires |
| Super-user pour l'application | Utilisateur applicatif limité |

### Mots de Passe Sécurisés

```sql
-- ❌ MAUVAIS
CREATE USER app WITH PASSWORD '123456';
CREATE USER admin WITH PASSWORD 'admin';

-- ✅ BON
CREATE USER app WITH PASSWORD 'Ap9!xK2#mN8$qR5@vT7';
CREATE USER admin WITH PASSWORD 'AdM!n_S3cuR3_P@ssw0rd_2024!';
```

**Recommandations :**
- Au moins 16 caractères
- Mélange majuscules, minuscules, chiffres, symboles
- Pas de mots du dictionnaire
- Unique par utilisateur
- Stocké dans un gestionnaire de mots de passe

### Utiliser des Fichiers .env

Ne mettez JAMAIS les mots de passe dans `docker-compose.yml` !

**Fichier `.env` :**
```bash
# .env (à ne JAMAIS commiter)
POSTGRES_PASSWORD=super_secure_root_pass!
POSTGRES_USER=mon_utilisateur

DB_BLOG_ADMIN_PASS=admin_blog_secure_123!
DB_BLOG_APP_PASS=app_blog_secure_456!
DB_BLOG_READ_PASS=read_blog_secure_789!
```

**Dans `docker-compose.yml` :**
```yaml
environment:
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  POSTGRES_USER: ${POSTGRES_USER}
```

### Scripts d'Initialisation Automatique

Créez un script SQL exécuté au premier démarrage :

**Fichier `init/01-create-users.sql` :**
```sql
-- Script exécuté automatiquement au démarrage

-- Créer la base blog
CREATE DATABASE blog_db;

-- Se connecter à la base
\c blog_db

-- Créer les utilisateurs
CREATE USER blog_admin WITH PASSWORD 'change_me_admin';
CREATE USER blog_app WITH PASSWORD 'change_me_app';
CREATE USER blog_readonly WITH PASSWORD 'change_me_readonly';

-- Configurer les permissions
ALTER DATABASE blog_db OWNER TO blog_admin;

GRANT CONNECT ON DATABASE blog_db TO blog_app;
GRANT USAGE ON SCHEMA public TO blog_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO blog_app;

GRANT CONNECT ON DATABASE blog_db TO blog_readonly;
GRANT USAGE ON SCHEMA public TO blog_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO blog_readonly;
```

**Dans `docker-compose.yml` :**
```yaml
volumes:
  - ./data:/var/lib/postgresql/data
  - ./init:/docker-entrypoint-initdb.d:ro
```

Les fichiers `.sql` dans `/docker-entrypoint-initdb.d` sont exécutés automatiquement au premier démarrage.

### Audit des Connexions

Activez le logging des connexions dans `postgresql.conf` :

```ini
# Logs des connexions/déconnexions
log_connections = on
log_disconnections = on

# Log des tentatives d'authentification échouées
log_min_messages = warning

# Détails des connexions
log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a,client=%h '
```

---

## 🔍 Commandes Utiles d'Inspection

### Voir Qui Est Connecté

```sql
-- Connexions actives
SELECT
    pid,
    usename AS username,
    datname AS database,
    client_addr AS client_ip,
    application_name,
    backend_start,
    state
FROM pg_stat_activity
WHERE datname IS NOT NULL
ORDER BY backend_start DESC;
```

### Voir les Objets Appartenant à un Utilisateur

```sql
-- Tables
SELECT schemaname, tablename, tableowner
FROM pg_tables
WHERE tableowner = 'blog_app';

-- Vues
SELECT schemaname, viewname, viewowner
FROM pg_views
WHERE viewowner = 'blog_app';

-- Séquences
SELECT schemaname, sequencename, sequenceowner
FROM pg_sequences
WHERE sequenceowner = 'blog_app';
```

### Taille des Bases de Données

```sql
-- Taille de chaque base
SELECT
    datname AS database_name,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
WHERE datistemplate = false
ORDER BY pg_database_size(datname) DESC;
```

### Historique des Commandes (psql)

```sql
-- Voir l'historique
\s

-- Sauvegarder l'historique
\s /tmp/history.txt

-- Exécuter à nouveau la commande N
\g N
```

---

## ⚠️ Erreurs Courantes

### Erreur : "permission denied for table"

**Cause** : L'utilisateur n'a pas les permissions sur la table.

**Solution** :
```sql
-- Se connecter en super-user
\c blog_db postgres

-- Accorder les permissions
GRANT SELECT, INSERT, UPDATE, DELETE
    ON TABLE articles
    TO blog_app;
```

### Erreur : "database is being accessed by other users"

**Cause** : Impossible de supprimer une base si quelqu'un est connecté.

**Solution** :
```sql
-- Voir qui est connecté
SELECT * FROM pg_stat_activity WHERE datname = 'blog_db';

-- Déconnecter tous les utilisateurs
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'blog_db' AND pid <> pg_backend_pid();

-- Maintenant on peut supprimer
DROP DATABASE blog_db;
```

### Erreur : "role cannot be dropped because some objects depend on it"

**Cause** : L'utilisateur possède des objets (tables, vues...).

**Solution** :
```sql
-- Voir ce qu'il possède
\dt

-- Transférer la propriété
REASSIGN OWNED BY old_user TO new_user;

-- Supprimer les dépendances restantes
DROP OWNED BY old_user;

-- Supprimer l'utilisateur
DROP USER old_user;
```

### Erreur : "password authentication failed"

**Cause** : Mauvais mot de passe ou utilisateur inexistant.

**Solution** :
```sql
-- Vérifier que l'utilisateur existe
\du

-- Réinitialiser le mot de passe
ALTER USER blog_app WITH PASSWORD 'nouveau_pass!';
```

---

## 📊 Tableau Récapitulatif des Permissions

| Permission | SELECT | INSERT | UPDATE | DELETE | TRUNCATE | CREATE | DROP | ALTER |
|------------|--------|--------|--------|--------|----------|--------|------|-------|
| **Lecture seule** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Application (CRUD)** | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Déploiement** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Admin de base** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Super-utilisateur** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## ✅ Checklist de Configuration

Avant de considérer votre configuration utilisateurs prête :

### Bases de Données
- [ ] Bases créées pour chaque projet
- [ ] Propriétaires assignés
- [ ] Nommage cohérent (projet_env)

### Utilisateurs
- [ ] Un utilisateur par rôle/application
- [ ] Mots de passe forts et uniques
- [ ] Dates d'expiration pour comptes temporaires
- [ ] Pas de super-utilisateur inutile

### Permissions
- [ ] Principe du moindre privilège appliqué
- [ ] Permissions testées pour chaque utilisateur
- [ ] `ALTER DEFAULT PRIVILEGES` configuré
- [ ] Permissions documentées

### Sécurité
- [ ] Mots de passe dans `.env` (pas dans docker-compose)
- [ ] `.env` dans `.gitignore`
- [ ] Logs de connexion activés
- [ ] Scripts d'init sécurisés

---

## 🚀 Prochaines Étapes

Maintenant que vos utilisateurs et bases sont configurés :

1. **Ajouter pgAdmin** → [Configuration avec pgAdmin](05-config-avec-pgadmin.md)
2. **Créer des sauvegardes** → [Annexe C - Gestion des volumes](/annexes/C-gestion-volumes.md)
3. **Sécuriser davantage** → [Annexe D - Sécurité](/annexes/D-securite-bonnes-pratiques.md)
4. **Déployer une stack complète** → [Cas Pratiques](/cas-pratiques/)

---

## 📚 Ressources Complémentaires

### Documentation Officielle
- **PostgreSQL User Management** : [https://www.postgresql.org/docs/current/user-manag.html](https://www.postgresql.org/docs/current/user-manag.html)
- **PostgreSQL Privileges** : [https://www.postgresql.org/docs/current/ddl-priv.html](https://www.postgresql.org/docs/current/ddl-priv.html)
- **GRANT Reference** : [https://www.postgresql.org/docs/current/sql-grant.html](https://www.postgresql.org/docs/current/sql-grant.html)

### Guides et Tutoriels
- **PostgreSQL Permission Concepts** : [https://www.postgresqltutorial.com/postgresql-administration/postgresql-roles/](https://www.postgresqltutorial.com/postgresql-administration/postgresql-roles/)
- **Security Best Practices** : [https://wiki.postgresql.org/wiki/Security](https://wiki.postgresql.org/wiki/Security)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
