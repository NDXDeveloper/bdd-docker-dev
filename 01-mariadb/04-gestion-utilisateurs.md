# Gestion des utilisateurs et permissions MariaDB

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous apprend à créer et gérer des utilisateurs dans MariaDB, ainsi qu'à leur attribuer les permissions appropriées. La gestion des utilisateurs est essentielle pour la sécurité et l'organisation de vos bases de données.

**Ce que vous allez apprendre :**
- Comprendre le système d'utilisateurs MariaDB
- Créer différents types d'utilisateurs (admin, développeur, lecture seule)
- Gérer les permissions (GRANT/REVOKE)
- Comprendre les hôtes et restrictions d'accès
- Suivre les bonnes pratiques de sécurité

**Durée estimée :** 25 minutes

---

## 🎯 Prérequis

Avant de commencer, vous devez :

- ✅ Avoir suivi la [fiche 1.1 - Configuration basique](01-config-basique-docker-compose.md)
- ✅ Savoir se connecter à MariaDB via le terminal
- ✅ Comprendre les bases du SQL (optionnel mais utile)

---

## 🤔 Comprendre les utilisateurs MariaDB

### Qu'est-ce qu'un utilisateur ?

Un **utilisateur** dans MariaDB est un compte qui permet de se connecter au serveur de base de données. Chaque utilisateur a :

- 🔑 Un **nom** (login)
- 🔐 Un **mot de passe**
- 🌐 Un **hôte autorisé** (d'où il peut se connecter)
- 📝 Des **permissions** (ce qu'il peut faire)

### Format d'un utilisateur

En MariaDB, un utilisateur est identifié par : **`'nom'@'hôte'`**

**Exemples :**

| Utilisateur | Signification |
|-------------|---------------|
| `'root'@'localhost'` | Utilisateur `root` qui peut se connecter UNIQUEMENT depuis l'intérieur du conteneur |
| `'dev'@'%'` | Utilisateur `dev` qui peut se connecter depuis N'IMPORTE OÙ |
| `'app'@'192.168.1.%'` | Utilisateur `app` qui peut se connecter depuis le réseau `192.168.1.x` |
| `'admin'@'172.18.0.50'` | Utilisateur `admin` qui peut se connecter UNIQUEMENT depuis l'IP `172.18.0.50` |

### Le caractère spécial `%`

Le symbole `%` est un **joker** (wildcard) qui signifie "n'importe quoi" :

- `'%'` seul = n'importe quelle adresse IP
- `'192.168.%'` = toutes les IP commençant par 192.168
- `'%.exemple.com'` = tous les hôtes du domaine exemple.com

---

## 🔐 L'utilisateur root

### Particularités de root

L'utilisateur **root** est le **super-administrateur** créé automatiquement :

- ✅ Possède TOUS les privilèges
- ✅ Peut créer/supprimer des bases de données
- ✅ Peut créer/supprimer des utilisateurs
- ✅ Peut modifier les permissions
- ⚠️ À NE PAS utiliser pour les applications !

### Configuration dans docker-compose.yml

```yaml
environment:
  # Définition du mot de passe root (OBLIGATOIRE)
  MYSQL_ROOT_PASSWORD: mon_mot_de_passe_root_tres_securise
```

**💡 Bonne pratique :** Utilisez root uniquement pour l'administration, créez des utilisateurs spécifiques pour les applications.

---

## 👥 Types d'utilisateurs courants

### Vue d'ensemble

| Type | Usage | Permissions typiques |
|------|-------|---------------------|
| **Super Admin** | Administration complète | ALL PRIVILEGES |
| **Admin BDD** | Gestion d'une base spécifique | ALL sur une BDD |
| **Développeur** | Développement et tests | SELECT, INSERT, UPDATE, DELETE, CREATE, DROP |
| **Application** | Utilisation par une app | SELECT, INSERT, UPDATE, DELETE |
| **Lecture seule** | Reporting, analytics | SELECT uniquement |
| **Backup** | Sauvegardes | SELECT, LOCK TABLES, SHOW VIEW |

---

## 📝 Étape 1 : Se connecter en tant que root

### 1.1 Préparer l'environnement

Assurez-vous que votre conteneur MariaDB est démarré :

```bash
# Lancer le conteneur (si pas déjà fait)
docker-compose up -d

# Vérifier qu'il fonctionne
docker-compose ps
```

### 1.2 Se connecter au shell MariaDB

```bash
# Remplacez 'mariadb_dev' par le nom de votre conteneur
docker exec -it mariadb_dev mariadb -u root -p
```

Entrez le mot de passe root défini dans votre `docker-compose.yml`.

**Vous devriez voir :**
```
MariaDB [(none)]>
```

C'est le **prompt MariaDB** où vous pouvez taper des commandes SQL.

---

## 👤 Étape 2 : Créer des utilisateurs

### 2.1 Syntaxe de base

```sql
CREATE USER 'nom_utilisateur'@'hôte' IDENTIFIED BY 'mot_de_passe';
```

### 2.2 Créer un utilisateur administrateur (accès depuis partout)

```sql
-- Créer un admin accessible depuis n'importe où
CREATE USER 'admin_distant'@'%' IDENTIFIED BY 'password_admin_tres_securise';

-- Vérifier la création
SELECT User, Host FROM mysql.user WHERE User = 'admin_distant';
```

**Résultat attendu :**
```
+----------------+------+
| User           | Host |
+----------------+------+
| admin_distant  | %    |
+----------------+------+
```

### 2.3 Créer un utilisateur pour une application (accès local uniquement)

```sql
-- Utilisateur pour une app tournant dans le même conteneur
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password_app';

-- Vérifier
SELECT User, Host FROM mysql.user WHERE User = 'app_user';
```

### 2.4 Créer un utilisateur lecture seule

```sql
-- Utilisateur qui ne peut que lire les données
CREATE USER 'lecteur'@'%' IDENTIFIED BY 'password_lecteur';
```

### 2.5 Créer un utilisateur pour un réseau spécifique

```sql
-- Accès uniquement depuis le réseau 192.168.1.x
CREATE USER 'dev_local'@'192.168.1.%' IDENTIFIED BY 'password_dev';

-- Accès depuis une IP Docker spécifique
CREATE USER 'app_container'@'172.18.0.50' IDENTIFIED BY 'password_container';
```

---

## 🔑 Étape 3 : Attribuer des permissions (GRANT)

### 3.1 Syntaxe de base

```sql
GRANT permissions ON base.table TO 'utilisateur'@'hôte';
```

**Composants :**
- `permissions` : Ce que l'utilisateur peut faire (SELECT, INSERT, etc.)
- `base.table` : Sur quoi (quelle base/table)
- `TO 'utilisateur'@'hôte'` : À qui

### 3.2 Donner tous les privilèges (Super Admin)

```sql
-- Tous les privilèges sur TOUTES les bases
GRANT ALL PRIVILEGES ON *.* TO 'admin_distant'@'%';

-- Avec possibilité de donner des privilèges à d'autres (GRANT OPTION)
GRANT ALL PRIVILEGES ON *.* TO 'admin_distant'@'%' WITH GRANT OPTION;

-- Appliquer les changements immédiatement
FLUSH PRIVILEGES;
```

**Explication :**
- `*.*` = toutes les bases (`*`) et toutes les tables (`*`)
- `WITH GRANT OPTION` = peut créer des utilisateurs et donner des permissions

### 3.3 Permissions sur une base spécifique

```sql
-- Créer d'abord une base de données
CREATE DATABASE app_production;

-- Donner tous les droits sur cette base uniquement
GRANT ALL PRIVILEGES ON app_production.* TO 'app_user'@'localhost';

-- Appliquer
FLUSH PRIVILEGES;
```

### 3.4 Permissions spécifiques pour une application

```sql
-- Application qui lit et écrit des données (pas de modifications de structure)
GRANT SELECT, INSERT, UPDATE, DELETE ON app_production.* TO 'app_user'@'localhost';

FLUSH PRIVILEGES;
```

**Permissions expliquées :**
- `SELECT` : Lire les données (requêtes SELECT)
- `INSERT` : Ajouter des données (INSERT INTO)
- `UPDATE` : Modifier des données existantes (UPDATE)
- `DELETE` : Supprimer des données (DELETE FROM)

### 3.5 Utilisateur lecture seule (reporting)

```sql
-- Uniquement la lecture des données
GRANT SELECT ON app_production.* TO 'lecteur'@'%';

FLUSH PRIVILEGES;
```

### 3.6 Développeur complet (toutes permissions sauf administration)

```sql
-- Base de développement
CREATE DATABASE app_development;

-- Permissions étendues pour développement
GRANT SELECT, INSERT, UPDATE, DELETE,
      CREATE, DROP, ALTER, INDEX,
      CREATE TEMPORARY TABLES, LOCK TABLES
ON app_development.* TO 'dev_local'@'192.168.1.%';

FLUSH PRIVILEGES;
```

**Permissions supplémentaires :**
- `CREATE` : Créer des tables
- `DROP` : Supprimer des tables
- `ALTER` : Modifier la structure des tables
- `INDEX` : Créer/supprimer des index
- `CREATE TEMPORARY TABLES` : Tables temporaires
- `LOCK TABLES` : Verrouiller les tables

---

## 📋 Étape 4 : Voir les permissions

### 4.1 Afficher les utilisateurs existants

```sql
-- Liste de tous les utilisateurs
SELECT User, Host FROM mysql.user;
```

### 4.2 Voir les permissions d'un utilisateur

```sql
-- Permissions d'un utilisateur spécifique
SHOW GRANTS FOR 'app_user'@'localhost';
```

**Résultat exemple :**
```sql
+------------------------------------------------------------------------+
| Grants for app_user@localhost                                         |
+------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `app_user`@`localhost`                          |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `app_production`.* TO `app_user`@`localhost` |
+------------------------------------------------------------------------+
```

### 4.3 Voir vos propres permissions

```sql
-- Voir avec quel utilisateur vous êtes connecté
SELECT USER(), CURRENT_USER();

-- Voir vos permissions actuelles
SHOW GRANTS;
```

---

## 🔄 Étape 5 : Modifier des permissions

### 5.1 Ajouter des permissions

```sql
-- Ajouter la permission CREATE à un utilisateur existant
GRANT CREATE ON app_production.* TO 'app_user'@'localhost';

FLUSH PRIVILEGES;
```

### 5.2 Retirer des permissions (REVOKE)

```sql
-- Retirer la permission DELETE
REVOKE DELETE ON app_production.* FROM 'app_user'@'localhost';

FLUSH PRIVILEGES;
```

### 5.3 Retirer toutes les permissions

```sql
-- Retirer tous les privilèges (mais l'utilisateur existe encore)
REVOKE ALL PRIVILEGES ON *.* FROM 'dev_local'@'192.168.1.%';

FLUSH PRIVILEGES;
```

---

## 🗑️ Étape 6 : Supprimer un utilisateur

### 6.1 Supprimer un utilisateur

```sql
-- Supprimer complètement un utilisateur
DROP USER 'lecteur'@'%';

-- Vérifier la suppression
SELECT User, Host FROM mysql.user WHERE User = 'lecteur';
```

### 6.2 Supprimer plusieurs utilisateurs

```sql
-- Supprimer plusieurs utilisateurs en une fois
DROP USER
  'user1'@'localhost',
  'user2'@'%',
  'user3'@'192.168.1.%';
```

---

## 🔐 Étape 7 : Modifier un mot de passe

### 7.1 Changer le mot de passe d'un utilisateur

```sql
-- Modifier le mot de passe d'un utilisateur
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'nouveau_mot_de_passe';

FLUSH PRIVILEGES;
```

### 7.2 Changer votre propre mot de passe

```sql
-- Si vous êtes connecté en tant que cet utilisateur
SET PASSWORD = PASSWORD('mon_nouveau_mot_de_passe');

-- Ou avec la syntaxe moderne
ALTER USER USER() IDENTIFIED BY 'mon_nouveau_mot_de_passe';
```

### 7.3 Changer le mot de passe root

**⚠️ Depuis l'extérieur du conteneur :**

```bash
# Méthode 1 : Via docker exec
docker exec -it mariadb_dev mariadb -u root -p -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'nouveau_password_root';"

# Méthode 2 : Via variable d'environnement (redémarrage nécessaire)
# Modifier docker-compose.yml puis :
docker-compose down
docker-compose up -d
```

---

## 📊 Tableau récapitulatif des permissions

### Permissions de données

| Permission | Description | Exemple SQL |
|------------|-------------|-------------|
| `SELECT` | Lire les données | `SELECT * FROM users;` |
| `INSERT` | Ajouter des données | `INSERT INTO users VALUES (...);` |
| `UPDATE` | Modifier des données | `UPDATE users SET name = 'John';` |
| `DELETE` | Supprimer des données | `DELETE FROM users WHERE id = 5;` |

### Permissions de structure

| Permission | Description | Exemple SQL |
|------------|-------------|-------------|
| `CREATE` | Créer tables/BDD | `CREATE TABLE products (...);` |
| `DROP` | Supprimer tables/BDD | `DROP TABLE old_data;` |
| `ALTER` | Modifier structure | `ALTER TABLE users ADD COLUMN age INT;` |
| `INDEX` | Gérer les index | `CREATE INDEX idx_name ON users(name);` |

### Permissions d'administration

| Permission | Description |
|------------|-------------|
| `ALL PRIVILEGES` | Tous les privilèges |
| `GRANT OPTION` | Peut donner ses privilèges à d'autres |
| `CREATE USER` | Créer des utilisateurs |
| `RELOAD` | Recharger les privilèges |
| `SHUTDOWN` | Arrêter le serveur |
| `PROCESS` | Voir les processus en cours |
| `SUPER` | Opérations super-utilisateur |

---

## 🎯 Scénarios pratiques

### Scénario 1 : Configuration pour une application web

```sql
-- Créer la base de données
CREATE DATABASE mon_site_web;

-- Créer l'utilisateur pour l'application
CREATE USER 'webapp'@'%' IDENTIFIED BY 'password_webapp_securise';

-- Donner les permissions nécessaires (pas de DROP/ALTER pour la sécurité)
GRANT SELECT, INSERT, UPDATE, DELETE ON mon_site_web.* TO 'webapp'@'%';

FLUSH PRIVILEGES;
```

### Scénario 2 : Équipe de développement

```sql
-- Base de dev
CREATE DATABASE projet_dev;

-- Développeur 1 (accès total sur la base dev)
CREATE USER 'dev1'@'192.168.1.%' IDENTIFIED BY 'password_dev1';
GRANT ALL PRIVILEGES ON projet_dev.* TO 'dev1'@'192.168.1.%';

-- Développeur 2 (même chose)
CREATE USER 'dev2'@'192.168.1.%' IDENTIFIED BY 'password_dev2';
GRANT ALL PRIVILEGES ON projet_dev.* TO 'dev2'@'192.168.1.%';

-- Stagiaire (lecture seule)
CREATE USER 'stagiaire'@'192.168.1.%' IDENTIFIED BY 'password_stagiaire';
GRANT SELECT ON projet_dev.* TO 'stagiaire'@'192.168.1.%';

FLUSH PRIVILEGES;
```

### Scénario 3 : Utilisateur de backup

```sql
-- Utilisateur spécialisé pour les sauvegardes
CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'password_backup';

-- Permissions minimales pour faire un dump
GRANT SELECT, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER ON *.* TO 'backup_user'@'localhost';

FLUSH PRIVILEGES;
```

### Scénario 4 : Application multi-conteneurs Docker

```sql
-- Frontend (lecture seule sur les données publiques)
CREATE USER 'frontend'@'172.18.0.50' IDENTIFIED BY 'password_frontend';
GRANT SELECT ON mon_site_web.public_data TO 'frontend'@'172.18.0.50';

-- Backend API (CRUD complet)
CREATE USER 'backend'@'172.18.0.60' IDENTIFIED BY 'password_backend';
GRANT SELECT, INSERT, UPDATE, DELETE ON mon_site_web.* TO 'backend'@'172.18.0.60';

-- Service analytics (lecture seule, toutes les tables)
CREATE USER 'analytics'@'172.18.0.70' IDENTIFIED BY 'password_analytics';
GRANT SELECT ON mon_site_web.* TO 'analytics'@'172.18.0.70';

FLUSH PRIVILEGES;
```

---

## 🔒 Bonnes pratiques de sécurité

### ✅ À FAIRE

1. **Principe du moindre privilège**
   ```sql
   -- ✅ BON : Donner uniquement ce qui est nécessaire
   GRANT SELECT, INSERT, UPDATE ON app.* TO 'user'@'%';

   -- ❌ MAUVAIS : Donner tous les droits par facilité
   GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';
   ```

2. **Restreindre les hôtes**
   ```sql
   -- ✅ BON : Limiter l'accès à une IP/réseau
   CREATE USER 'app'@'192.168.1.50' IDENTIFIED BY 'password';

   -- ❌ À ÉVITER : Accès depuis partout
   CREATE USER 'app'@'%' IDENTIFIED BY 'password';
   ```

3. **Mots de passe forts**
   ```sql
   -- ✅ BON
   IDENTIFIED BY 'Xk9$mP2#qL8@vN5&wR7!';

   -- ❌ MAUVAIS
   IDENTIFIED BY 'password123';
   ```

4. **Ne pas utiliser root pour les applications**
   ```sql
   -- ❌ MAUVAIS : Connexion d'une app en root
   mysql_connect('localhost', 'root', 'rootpass');

   -- ✅ BON : Utilisateur dédié
   mysql_connect('localhost', 'app_user', 'app_pass');
   ```

5. **Supprimer les utilisateurs inutilisés**
   ```sql
   -- Audit régulier
   SELECT User, Host FROM mysql.user;

   -- Suppression
   DROP USER 'ancien_user'@'%';
   ```

### ❌ À NE PAS FAIRE

- ❌ Partager les identifiants entre plusieurs personnes/apps
- ❌ Stocker les mots de passe en clair dans le code source
- ❌ Utiliser des mots de passe faibles
- ❌ Donner `ALL PRIVILEGES` par défaut
- ❌ Oublier de faire `FLUSH PRIVILEGES` après modifications

---

## 🧪 Tester les permissions

### Méthode de test

1. **Créer un utilisateur de test**
   ```sql
   CREATE USER 'test_user'@'%' IDENTIFIED BY 'test_pass';
   GRANT SELECT ON ma_base.ma_table TO 'test_user'@'%';
   FLUSH PRIVILEGES;
   ```

2. **Se connecter avec cet utilisateur**
   ```bash
   docker exec -it mariadb_dev mariadb -u test_user -ptest_pass
   ```

3. **Tester les permissions**
   ```sql
   -- Devrait fonctionner
   USE ma_base;
   SELECT * FROM ma_table LIMIT 1;

   -- Devrait échouer (pas de permission INSERT)
   INSERT INTO ma_table VALUES ('test');
   -- Erreur: INSERT command denied
   ```

4. **Nettoyer**
   ```bash
   # Se reconnecter en root
   docker exec -it mariadb_dev mariadb -u root -p
   ```

   ```sql
   DROP USER 'test_user'@'%';
   ```

---

## 🛠️ Commandes utiles

### Diagnostic et monitoring

```sql
-- Voir qui est connecté actuellement
SHOW PROCESSLIST;

-- Voir toutes les connexions d'un utilisateur
SELECT * FROM information_schema.processlist WHERE USER = 'app_user';

-- Voir les bases de données accessibles
SHOW DATABASES;

-- Tester une connexion
SELECT 1;

-- Voir la version de MariaDB
SELECT VERSION();

-- Voir l'utilisateur actuel
SELECT USER(), CURRENT_USER();
```

---

## ✅ Récapitulatif

Vous avez appris à :

- ✅ Comprendre le format `'utilisateur'@'hôte'` et le wildcard `%`
- ✅ Créer différents types d'utilisateurs (admin, app, lecture seule)
- ✅ Attribuer des permissions avec `GRANT`
- ✅ Retirer des permissions avec `REVOKE`
- ✅ Voir et vérifier les permissions (`SHOW GRANTS`)
- ✅ Modifier et supprimer des utilisateurs
- ✅ Appliquer les bonnes pratiques de sécurité
- ✅ Gérer des scénarios réels (app web, équipe dev, backups)

**Commandes essentielles :**
- `CREATE USER 'nom'@'hôte' IDENTIFIED BY 'password';`
- `GRANT permissions ON base.table TO 'nom'@'hôte';`
- `REVOKE permissions ON base.table FROM 'nom'@'hôte';`
- `SHOW GRANTS FOR 'nom'@'hôte';`
- `DROP USER 'nom'@'hôte';`
- `FLUSH PRIVILEGES;`

---

## 🚀 Prochaines étapes

Maintenant que vous maîtrisez la gestion des utilisateurs, explorez :

- **[1.5 Accès réseau local](05-acces-reseau-local.md)** - Permettre l'accès depuis d'autres machines
- **[Annexe D - Sécurité](../annexes/D-securite-bonnes-pratiques.md)** - Approfondir la sécurité

---

## 📚 Ressources complémentaires

- [Documentation MariaDB - GRANT](https://mariadb.com/kb/en/grant/)
- [Documentation MariaDB - Account Management](https://mariadb.com/kb/en/account-management-sql-commands/)
- [Documentation MariaDB - Privileges](https://mariadb.com/kb/en/grant/#privilege-levels)

---

## ❓ FAQ - Questions fréquentes

**Q : Quelle est la différence entre 'user'@'localhost' et 'user'@'127.0.0.1' ?**
R : `localhost` utilise une socket Unix (plus rapide), tandis que `127.0.0.1` utilise TCP/IP. Dans un conteneur Docker, privilégiez `localhost` pour les connexions internes.

**Q : Pourquoi dois-je faire FLUSH PRIVILEGES ?**
R : `FLUSH PRIVILEGES` recharge les tables de privilèges en mémoire. Nécessaire après une modification manuelle de la table `mysql.user`, mais pas toujours après `GRANT`/`REVOKE` (MariaDB moderne).

**Q : Comment voir les utilisateurs qui ont accès à une base spécifique ?**
R :
```sql
SELECT DISTINCT User, Host FROM mysql.db WHERE Db = 'ma_base';
```

**Q : Puis-je créer un utilisateur sans mot de passe ?**
R : Techniquement oui, mais **c'est très dangereux** :
```sql
CREATE USER 'test'@'localhost';  -- PAS DE MOT DE PASSE !
```

**Q : Comment limiter le nombre de connexions d'un utilisateur ?**
R :
```sql
CREATE USER 'limited'@'%' IDENTIFIED BY 'password'
  WITH MAX_CONNECTIONS_PER_HOUR 100
       MAX_QUERIES_PER_HOUR 10000;
```

**Q : Que signifie "USAGE" dans SHOW GRANTS ?**
R : `USAGE` signifie "aucun privilège" – c'est le privilège minimum qui permet juste de se connecter.

**Q : Comment créer un utilisateur qui peut tout faire SAUF supprimer des tables ?**
R :
```sql
GRANT ALL PRIVILEGES ON ma_base.* TO 'user'@'%';
REVOKE DROP ON ma_base.* FROM 'user'@'%';
```

**Q : Les utilisateurs créés dans docker-compose.yml persistent-ils après un redémarrage ?**
R : Oui, si vous utilisez un volume pour `/var/lib/mysql`. Les utilisateurs SQL sont stockés dans la base système `mysql`.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
