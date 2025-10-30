# 3.3 Gestion des utilisateurs et logins

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans MS SQL Server, la gestion des accès repose sur un système à **deux niveaux** qui peut sembler complexe au premier abord, mais qui offre une grande flexibilité et sécurité :

1. **Login** : Identité au niveau du **serveur** (permet de se connecter à SQL Server)
2. **User** : Identité au niveau de la **base de données** (permissions dans une BDD spécifique)

Cette architecture permet de contrôler finement qui peut se connecter au serveur et ce que chaque personne peut faire dans chaque base de données.

### Pourquoi ne pas utiliser uniquement `sa` ?

L'utilisateur `sa` (System Administrator) a **tous les droits** sur **tout le serveur**. C'est l'équivalent de "root" sous Linux. Utiliser `sa` pour tout présente des risques :

- 🚫 Pas de traçabilité (impossible de savoir qui a fait quoi)
- 🚫 Pas de limitation (un développeur peut accidentellement supprimer des données critiques)
- 🚫 Risque de sécurité (si les identifiants sont compromis, tout est compromis)

**Bonne pratique** : Créer des comptes dédiés avec des permissions limitées selon les besoins.

---

## 🎯 Ce que vous allez apprendre

Dans ce tutoriel, vous allez :
- ✅ Comprendre la différence entre Login et User
- ✅ Créer des logins (authentification serveur)
- ✅ Créer des utilisateurs dans une base de données
- ✅ Attribuer des permissions (rôles et privilèges)
- ✅ Gérer les mots de passe
- ✅ Supprimer proprement des accès

---

## 🧠 Comprendre l'architecture Logins/Users

### Le système à deux niveaux

```
┌─────────────────────────────────────────────────────────┐
│  SERVEUR SQL SERVER                                     │
│                                                         │
│  ┌──────────────────────────────────────────┐           │
│  │  LOGINS (Authentification serveur)       │           │
│  │                                           │          │
│  │  • sa (admin)                             │          │
│  │  • login_dev (développeur)                │          │
│  │  • login_app (application)                │          │
│  │  • login_readonly (lecture seule)         │          │
│  └──────────────────────────────────────────┘           │
│           │                                             │
│           ├─────────────┬─────────────┬──────────────┐  │
│           │             │             │              │  │
│           ▼             ▼             ▼              ▼  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  ...   │
│  │  BDD: prod  │ │  BDD: test  │ │  BDD: dev   │        │
│  │             │ │             │ │             │        │
│  │  USERS:     │ │  USERS:     │ │  USERS:     │        │
│  │  • user_dev │ │  • user_dev │ │  • user_dev │        │
│  │  • user_app │ │  • user_app │ │  • user_app │        │
│  └─────────────┘ └─────────────┘ └─────────────┘        │
└─────────────────────────────────────────────────────────┘
```

### Analogie simple

Pensez à un immeuble :
- **Login** = Badge d'entrée de l'immeuble (vous donne accès au bâtiment)
- **User** = Clé d'un appartement spécifique (vous donne accès à un logement précis)

Vous devez avoir les deux pour entrer dans un appartement !

---

## 📋 Prérequis

- Un conteneur MS SQL Server opérationnel (voir [Configuration basique](01-config-basique-docker-compose.md))
- Accès avec l'utilisateur `sa`
- Un client SQL (Azure Data Studio, SSMS, sqlcmd...)

---

## 🔌 Étape 1 : Se connecter au serveur

### A. Avec sqlcmd (ligne de commande)

```bash
docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U sa \
  -P "VotreMotDePasse123!"
```

### B. Avec un client graphique (Azure Data Studio, SSMS)

**Paramètres de connexion :**
- Serveur : `localhost` ou `127.0.0.1`
- Port : `1433`
- Authentification : SQL Server Authentication
- Utilisateur : `sa`
- Mot de passe : Votre mot de passe SA

---

## 👤 Étape 2 : Créer des Logins (niveau serveur)

### A. Login avec authentification SQL Server

C'est le type le plus courant pour Docker/Linux.

#### Créer un login développeur

```sql
-- Créer un login pour un développeur
CREATE LOGIN dev_user
WITH PASSWORD = 'DevPassword123!';
GO
```

**Détails :**
- `dev_user` : Nom du login (identifiant de connexion)
- `PASSWORD` : Mot de passe (doit respecter la politique de complexité)

#### Créer un login pour une application

```sql
-- Créer un login pour une application
CREATE LOGIN app_user
WITH PASSWORD = 'AppPassword123!';
GO
```

#### Créer un login en lecture seule

```sql
-- Créer un login pour consultation uniquement
CREATE LOGIN readonly_user
WITH PASSWORD = 'ReadPassword123!';
GO
```

### B. Options avancées lors de la création

```sql
-- Login avec options complètes
CREATE LOGIN user_complet
WITH PASSWORD = 'Password123!',
     DEFAULT_DATABASE = ma_base,           -- BDD par défaut
     CHECK_EXPIRATION = OFF,                -- Pas d'expiration du mot de passe
     CHECK_POLICY = ON;                     -- Vérifier la politique de mot de passe
GO
```

**Options disponibles :**
- `DEFAULT_DATABASE` : Base de données ouverte automatiquement après connexion
- `CHECK_EXPIRATION` : Active/désactive l'expiration du mot de passe
- `CHECK_POLICY` : Active/désactive la vérification de la politique Windows
- `DEFAULT_LANGUAGE` : Langue par défaut (french, english...)

### C. Vérifier les logins créés

```sql
-- Lister tous les logins
SELECT name, type_desc, create_date, modify_date
FROM sys.server_principals
WHERE type IN ('S', 'U')  -- S = SQL Login, U = Windows Login
ORDER BY name;
GO
```

**Sortie exemple :**
```
name              type_desc             create_date              modify_date
----------------- --------------------- ------------------------ ------------------------
app_user          SQL_LOGIN             2025-10-30 10:30:00.000  2025-10-30 10:30:00.000
dev_user          SQL_LOGIN             2025-10-30 10:25:00.000  2025-10-30 10:25:00.000
readonly_user     SQL_LOGIN             2025-10-30 10:35:00.000  2025-10-30 10:35:00.000
sa                SQL_LOGIN             2025-01-01 00:00:00.000  2025-01-01 00:00:00.000
```

---

## 🗄️ Étape 3 : Créer une base de données de test

Créons une base de données pour tester nos permissions.

```sql
-- Créer une base de données de test
CREATE DATABASE ma_base_test;
GO

-- Vérifier la création
SELECT name, database_id, create_date
FROM sys.databases
WHERE name = 'ma_base_test';
GO
```

---

## 👥 Étape 4 : Créer des Users (niveau base de données)

Un **Login** permet de se connecter au serveur, mais il ne donne **aucun droit** dans les bases de données. Il faut créer des **Users** dans chaque BDD.

### A. Passer en mode base de données

```sql
-- Utiliser la base de données
USE ma_base_test;
GO
```

### B. Créer des users liés aux logins

```sql
-- Créer un user pour le login développeur
CREATE USER dev_user FOR LOGIN dev_user;
GO

-- Créer un user pour le login application
CREATE USER app_user FOR LOGIN app_user;
GO

-- Créer un user pour le login lecture seule
CREATE USER readonly_user FOR LOGIN readonly_user;
GO
```

**Syntaxe :**
```sql
CREATE USER [nom_user] FOR LOGIN [nom_login];
```

> 💡 **Convention** : On utilise souvent le même nom pour le login et le user, mais ce n'est pas obligatoire.

### C. Vérifier les users créés

```sql
-- Lister les users de la base actuelle
SELECT name, type_desc, create_date
FROM sys.database_principals
WHERE type = 'S'  -- S = SQL User
ORDER BY name;
GO
```

---

## 🔐 Étape 5 : Attribuer des permissions (Rôles)

SQL Server utilise des **rôles de base de données** pour simplifier la gestion des permissions.

### A. Rôles prédéfinis principaux

| Rôle | Description | Permissions |
|------|-------------|-------------|
| **db_owner** | Propriétaire | TOUS les droits dans la BDD |
| **db_datareader** | Lecteur | SELECT sur toutes les tables |
| **db_datawriter** | Écrivain | INSERT, UPDATE, DELETE |
| **db_ddladmin** | Admin structure | CREATE, ALTER, DROP des objets |
| **db_backupoperator** | Sauvegarde | BACKUP de la BDD |
| **db_securityadmin** | Sécurité | Gérer les rôles et permissions |
| **db_denydatareader** | Blocage lecture | Interdit SELECT |
| **db_denydatawriter** | Blocage écriture | Interdit INSERT, UPDATE, DELETE |

### B. Attribuer des rôles aux users

```sql
-- Développeur : Tous les droits dans cette BDD
ALTER ROLE db_owner ADD MEMBER dev_user;
GO

-- Application : Lecture et écriture
ALTER ROLE db_datareader ADD MEMBER app_user;
ALTER ROLE db_datawriter ADD MEMBER app_user;
GO

-- Lecture seule : Uniquement SELECT
ALTER ROLE db_datareader ADD MEMBER readonly_user;
GO
```

### C. Vérifier les rôles attribués

```sql
-- Voir les membres de chaque rôle
SELECT
    r.name AS role_name,
    m.name AS member_name
FROM sys.database_role_members rm
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
JOIN sys.database_principals m ON rm.member_principal_id = m.principal_id
ORDER BY r.name, m.name;
GO
```

**Sortie exemple :**
```
role_name          member_name
------------------ ----------------
db_datareader      app_user
db_datareader      readonly_user
db_datawriter      app_user
db_owner           dev_user
```

---

## 🧪 Étape 6 : Tester les permissions

### A. Créer des objets de test

Connectez-vous avec `sa` ou `dev_user` :

```sql
USE ma_base_test;
GO

-- Créer une table de test
CREATE TABLE employes (
    id INT PRIMARY KEY IDENTITY(1,1),
    nom NVARCHAR(100),
    poste NVARCHAR(100),
    salaire DECIMAL(10,2)
);
GO

-- Insérer des données
INSERT INTO employes (nom, poste, salaire)
VALUES
    ('Alice Martin', 'Développeuse', 45000),
    ('Bob Durand', 'Designer', 42000),
    ('Charlie Lefebvre', 'Manager', 55000);
GO
```

### B. Tester avec readonly_user (lecture seule)

**Déconnectez-vous et reconnectez-vous avec readonly_user :**

```bash
docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U readonly_user \
  -P "ReadPassword123!" \
  -d ma_base_test
```

**Test de lecture (devrait fonctionner) :**
```sql
SELECT * FROM employes;
GO
```

**Résultat :** Affiche les 3 employés ✅

**Test d'écriture (devrait échouer) :**
```sql
INSERT INTO employes (nom, poste, salaire)
VALUES ('David Nouveau', 'Stagiaire', 30000);
GO
```

**Résultat attendu :**
```
Msg 229, Level 14, State 5
The INSERT permission was denied on the object 'employes'
```
❌ Refusé (normal, readonly_user n'a pas le droit d'écrire)

### C. Tester avec app_user (lecture + écriture)

**Reconnectez-vous avec app_user :**

```bash
docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U app_user \
  -P "AppPassword123!" \
  -d ma_base_test
```

**Test de lecture :**
```sql
SELECT * FROM employes;
GO
```
✅ Fonctionne

**Test d'écriture :**
```sql
INSERT INTO employes (nom, poste, salaire)
VALUES ('Emma Nouvelle', 'Analyste', 40000);
GO

SELECT * FROM employes WHERE nom = 'Emma Nouvelle';
GO
```
✅ Fonctionne

**Test de modification de structure (devrait échouer) :**
```sql
ALTER TABLE employes ADD email NVARCHAR(255);
GO
```

**Résultat attendu :**
```
Msg 297, Level 16, State 1
The user does not have permission to perform this action
```
❌ Refusé (app_user n'a pas db_ddladmin)

---

## 🎛️ Permissions granulaires (avancé)

Au lieu d'utiliser des rôles, vous pouvez attribuer des permissions très précises.

### A. Permissions sur des tables spécifiques

```sql
-- Connecté en tant que sa ou dev_user
USE ma_base_test;
GO

-- Donner SELECT uniquement sur la table employes
GRANT SELECT ON employes TO readonly_user;
GO

-- Donner INSERT et UPDATE, mais pas DELETE
GRANT INSERT, UPDATE ON employes TO app_user;
GO

-- Interdire explicitement la suppression
DENY DELETE ON employes TO app_user;
GO
```

### B. Permissions sur des colonnes

```sql
-- Autoriser SELECT seulement sur certaines colonnes
-- (masquer la colonne salaire)
GRANT SELECT ON employes (id, nom, poste) TO readonly_user;
GO
```

### C. Vérifier les permissions d'un user

```sql
-- Voir toutes les permissions d'un user
USE ma_base_test;
GO

SELECT
    pr.name AS user_name,
    pr.type_desc,
    pe.permission_name,
    pe.state_desc,
    OBJECT_NAME(pe.major_id) AS object_name
FROM sys.database_principals pr
LEFT JOIN sys.database_permissions pe ON pr.principal_id = pe.grantee_principal_id
WHERE pr.name = 'app_user'
ORDER BY pe.permission_name;
GO
```

---

## 🔄 Gestion des mots de passe

### Changer le mot de passe d'un login

```sql
-- Changer le mot de passe
ALTER LOGIN dev_user
WITH PASSWORD = 'NouveauMotDePasse123!';
GO
```

### Changer son propre mot de passe

Si un utilisateur veut changer son propre mot de passe :

```sql
-- L'utilisateur doit connaître son ancien mot de passe
ALTER LOGIN dev_user
WITH PASSWORD = 'NouveauMotDePasse123!'
OLD_PASSWORD = 'DevPassword123!';
GO
```

### Réinitialiser un mot de passe oublié (admin uniquement)

```sql
-- Seul sa ou un admin peut faire ceci
ALTER LOGIN dev_user
WITH PASSWORD = 'MotDePasseReset123!';
GO
```

---

## 🔓 Activer/Désactiver un login

### Désactiver temporairement un login

```sql
-- Désactiver le login (ne peut plus se connecter)
ALTER LOGIN dev_user DISABLE;
GO
```

### Réactiver un login

```sql
-- Réactiver le login
ALTER LOGIN dev_user ENABLE;
GO
```

### Vérifier l'état d'un login

```sql
SELECT
    name,
    is_disabled,
    create_date,
    modify_date
FROM sys.server_principals
WHERE name = 'dev_user';
GO
```

---

## 🗑️ Supprimer des accès

### A. Retirer un user d'un rôle

```sql
USE ma_base_test;
GO

-- Retirer app_user du rôle db_datawriter
ALTER ROLE db_datawriter DROP MEMBER app_user;
GO
```

### B. Supprimer un user d'une base de données

```sql
USE ma_base_test;
GO

-- Supprimer le user
DROP USER readonly_user;
GO
```

> ⚠️ **Important** : Le login existe toujours au niveau serveur !

### C. Supprimer un login (niveau serveur)

```sql
-- Se connecter à la base master
USE master;
GO

-- Supprimer le login
DROP LOGIN readonly_user;
GO
```

> ⚠️ **Attention** : Vous devez d'abord supprimer tous les users associés dans toutes les bases avant de supprimer le login, sinon vous aurez une erreur.

### D. Supprimer proprement (login + users)

**Procédure complète :**

```sql
-- 1. Lister toutes les bases où le user existe
USE master;
GO

SELECT
    db.name AS database_name,
    dp.name AS user_name
FROM sys.databases db
CROSS APPLY (
    SELECT name
    FROM sys.database_principals
    WHERE name = 'dev_user'
) dp
WHERE db.database_id > 4;  -- Exclure les bases système
GO

-- 2. Supprimer le user de chaque base
USE ma_base_test;
GO
DROP USER dev_user;
GO

-- Répéter pour chaque base...

-- 3. Supprimer le login
USE master;
GO
DROP LOGIN dev_user;
GO
```

---

## 📊 Script de diagnostic complet

Voici un script pour voir tous vos logins, users et permissions :

```sql
-- ================================================
-- DIAGNOSTIC COMPLET DE SÉCURITÉ
-- ================================================

-- 1. Tous les logins du serveur
PRINT '=== LOGINS (Niveau Serveur) ==='
SELECT
    name,
    type_desc,
    is_disabled,
    create_date
FROM sys.server_principals
WHERE type IN ('S', 'U')
ORDER BY name;
GO

-- 2. Users de la base actuelle
PRINT '=== USERS (Base actuelle) ==='
SELECT
    name,
    type_desc,
    create_date
FROM sys.database_principals
WHERE type = 'S'
ORDER BY name;
GO

-- 3. Rôles et membres
PRINT '=== RÔLES ET MEMBRES ==='
SELECT
    r.name AS role_name,
    m.name AS member_name
FROM sys.database_role_members rm
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
JOIN sys.database_principals m ON rm.member_principal_id = m.principal_id
ORDER BY r.name, m.name;
GO

-- 4. Permissions détaillées
PRINT '=== PERMISSIONS DÉTAILLÉES ==='
SELECT
    pr.name AS user_name,
    pe.permission_name,
    pe.state_desc,
    OBJECT_NAME(pe.major_id) AS object_name
FROM sys.database_principals pr
LEFT JOIN sys.database_permissions pe ON pr.principal_id = pe.grantee_principal_id
WHERE pr.type = 'S'
ORDER BY pr.name, pe.permission_name;
GO
```

---

## 🎨 Cas d'usage typiques

### 1. Développeur Full Access

```sql
-- Login
CREATE LOGIN dev_full WITH PASSWORD = 'DevFull123!';
GO

-- User dans chaque BDD de développement
USE dev_database;
GO
CREATE USER dev_full FOR LOGIN dev_full;
ALTER ROLE db_owner ADD MEMBER dev_full;
GO
```

### 2. Application Web (CRUD)

```sql
-- Login
CREATE LOGIN web_app WITH PASSWORD = 'WebApp123!';
GO

-- User avec lecture/écriture uniquement
USE production_db;
GO
CREATE USER web_app FOR LOGIN web_app;
ALTER ROLE db_datareader ADD MEMBER web_app;
ALTER ROLE db_datawriter ADD MEMBER web_app;
GO
```

### 3. Service de Reporting (Lecture Seule)

```sql
-- Login
CREATE LOGIN reporting WITH PASSWORD = 'Report123!';
GO

-- User lecture seule sur toutes les tables
USE production_db;
GO
CREATE USER reporting FOR LOGIN reporting;
ALTER ROLE db_datareader ADD MEMBER reporting;
GO
```

### 4. Admin de Backup

```sql
-- Login
CREATE LOGIN backup_admin WITH PASSWORD = 'Backup123!';
GO

-- User avec droits de sauvegarde
USE master;
GO
CREATE USER backup_admin FOR LOGIN backup_admin;
ALTER ROLE db_backupoperator ADD MEMBER backup_admin;
GO
```

---

## 💡 Bonnes pratiques

### 1. Principe du moindre privilège

Donnez uniquement les permissions nécessaires :
- ❌ **Éviter** : Donner `db_owner` à tout le monde
- ✅ **Préférer** : Utiliser `db_datareader` + `db_datawriter` si suffisant

### 2. Ne jamais partager le compte sa

- Créez des comptes individuels
- Désactivez `sa` en production si possible :
  ```sql
  ALTER LOGIN sa DISABLE;
  GO
  ```

### 3. Utiliser des mots de passe forts

**Exigences minimales :**
- 8+ caractères
- Majuscules + minuscules
- Chiffres
- Caractères spéciaux (!@#$%^&*)

### 4. Documenter les accès

Créez un fichier `USERS.md` dans votre projet :

```markdown
# Utilisateurs SQL Server

## Logins créés
- `dev_user` : Développeur (db_owner sur dev_database)
- `app_user` : Application web (lecture/écriture)
- `readonly_user` : Reporting (lecture seule)

## Mots de passe
Stockés dans le gestionnaire de mots de passe d'équipe
```

### 5. Auditer régulièrement

```sql
-- Qui s'est connecté récemment ?
SELECT
    login_name,
    MAX(login_time) AS last_login
FROM sys.dm_exec_sessions
GROUP BY login_name
ORDER BY last_login DESC;
GO
```

---

## 🔍 Dépannage

### Problème : "Login failed for user"

**Causes possibles :**
1. Mauvais mot de passe
2. Login désactivé
3. Login n'existe pas
4. Problème de réseau/connexion

**Vérification :**
```sql
-- Vérifier si le login existe et est actif
SELECT name, is_disabled
FROM sys.server_principals
WHERE name = 'dev_user';
GO
```

### Problème : "The user does not have permission"

**Cause** : Le user n'a pas les permissions nécessaires.

**Vérification :**
```sql
-- Voir les rôles du user
SELECT
    r.name AS role_name
FROM sys.database_role_members rm
JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
JOIN sys.database_principals m ON rm.member_principal_id = m.principal_id
WHERE m.name = 'app_user';
GO
```

### Problème : "Cannot drop the login, as the user is currently logged in"

**Solution :**
```sql
-- 1. Tuer les sessions actives
USE master;
GO

DECLARE @kill VARCHAR(8000) = '';
SELECT @kill = @kill + 'KILL ' + CONVERT(VARCHAR(5), session_id) + ';'
FROM sys.dm_exec_sessions
WHERE login_name = 'dev_user';

EXEC(@kill);
GO

-- 2. Puis supprimer le login
DROP LOGIN dev_user;
GO
```

---

## 📚 Aller plus loin

- **[Restauration de backup](04-restauration-backup.md)** : Importer des bases avec leurs users
- **[Annexe D - Sécurité](../annexes/D-securite-bonnes-pratiques.md)** : Bonnes pratiques de sécurité avancées
- **Documentation Microsoft** : [Principals (Database Engine)](https://learn.microsoft.com/sql/relational-databases/security/authentication-access/principals-database-engine)

---

## 📝 Récapitulatif

Vous avez appris à :
- ✅ Comprendre la différence entre Login (serveur) et User (base de données)
- ✅ Créer des logins avec différents niveaux d'accès
- ✅ Créer des users dans des bases de données
- ✅ Utiliser les rôles prédéfinis (db_owner, db_datareader, db_datawriter...)
- ✅ Attribuer des permissions granulaires sur des objets spécifiques
- ✅ Gérer les mots de passe (création, modification, réinitialisation)
- ✅ Activer/désactiver des logins
- ✅ Supprimer proprement des logins et users
- ✅ Auditer et diagnostiquer les permissions

---

## 🎯 Points clés à retenir

1. **Login** = Porte d'entrée du serveur | **User** = Accès à une base spécifique
2. Un login peut avoir des users dans plusieurs bases de données
3. Les **rôles** simplifient la gestion des permissions
4. **db_owner** = Tous les droits | **db_datareader** = Lecture | **db_datawriter** = Écriture
5. Toujours appliquer le **principe du moindre privilège**
6. Ne jamais partager le compte `sa` en production
7. Documenter tous les comptes créés
8. Tester les permissions après création

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
