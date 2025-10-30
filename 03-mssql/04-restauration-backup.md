# 3.4 Restauration de backup

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

La restauration de backup (sauvegarde) est une opération courante lorsque vous devez :

- 🔄 Migrer une base de données d'un serveur vers un autre
- 🧪 Créer un environnement de test avec des données réelles
- 📊 Importer une base de données fournie par un client ou un partenaire
- 🎓 Travailler sur une base de données d'exemple pour apprendre
- 💾 Restaurer des données après un problème

Dans ce tutoriel, nous allons apprendre à restaurer des backups MS SQL Server (fichiers `.bak`) dans votre conteneur Docker.

---

## 🎯 Ce que vous allez apprendre

À la fin de ce tutoriel, vous saurez :
- ✅ Préparer votre environnement Docker pour recevoir des backups
- ✅ Copier un fichier `.bak` dans le conteneur
- ✅ Lister le contenu d'un backup
- ✅ Restaurer une base de données complète
- ✅ Gérer les conflits de noms de fichiers
- ✅ Restaurer avec des options avancées
- ✅ Vérifier que la restauration a réussi

---

## 📋 Prérequis

- Un conteneur MS SQL Server opérationnel (voir [Configuration basique](01-config-basique-docker-compose.md))
- Accès avec l'utilisateur `sa`
- Un fichier de backup `.bak` à restaurer
- Notions de base sur Docker et SQL Server

---

## 🧠 Comprendre les backups SQL Server

### Types de backups

| Type | Extension | Description | Utilisation |
|------|-----------|-------------|-------------|
| **Full Backup** | `.bak` | Sauvegarde complète de la base | Restauration complète |
| **Differential** | `.bak` | Changements depuis le dernier full | Restauration incrémentale |
| **Transaction Log** | `.trn` | Journal des transactions | Point-in-time recovery |

Dans ce tutoriel, nous nous concentrons sur les **Full Backups** (`.bak`), qui sont les plus courants.

### Anatomie d'un fichier .bak

Un fichier `.bak` contient :
- 📊 Les données (fichier de données `.mdf`)
- 📝 Le journal des transactions (fichier log `.ldf`)
- 🔐 Les métadonnées (noms, chemins, tailles...)
- 👥 Les utilisateurs et permissions de la base

---

## 📁 Étape 1 : Préparer l'environnement Docker

### A. Modifier docker-compose.yml pour le volume de backup

Ajoutons un volume pour faciliter le transfert des fichiers de backup.

**Fichier `docker-compose.yml` modifié :**

```yaml
version: '3.8'

services:
  mssql:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: mssql_dev
    restart: unless-stopped
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Developer"
      SA_PASSWORD: "VotreMotDePasse123!"
    ports:
      - "1433:1433"
    volumes:
      # Volume pour les données
      - mssql_data:/var/opt/mssql
      # Volume pour les backups (NOUVEAU)
      - ./backups:/var/opt/mssql/backups

volumes:
  mssql_data:
```

**Explications :**
- `./backups` : Dossier sur votre machine (hôte)
- `/var/opt/mssql/backups` : Dossier dans le conteneur
- Les fichiers placés dans `./backups` seront accessibles dans le conteneur

### B. Créer le dossier backups

```bash
# Créer le dossier sur votre machine
mkdir backups
```

### C. Redémarrer le conteneur avec la nouvelle configuration

```bash
# Arrêter le conteneur actuel
docker-compose down

# Redémarrer avec la nouvelle configuration
docker-compose up -d
```

---

## 📥 Étape 2 : Obtenir un fichier de backup

### A. Utiliser un backup existant

Si vous avez déjà un fichier `.bak`, placez-le dans le dossier `backups` :

```bash
# Exemple : copier un backup depuis un autre emplacement
cp /chemin/vers/votre/backup.bak ./backups/
```

### B. Créer un backup de test (optionnel)

Si vous n'avez pas de backup, créons-en un pour l'exemple :

```sql
-- Se connecter au serveur
USE master;
GO

-- Créer une base de données de test
CREATE DATABASE test_backup_db;
GO

USE test_backup_db;
GO

-- Créer une table et insérer des données
CREATE TABLE produits (
    id INT PRIMARY KEY IDENTITY(1,1),
    nom NVARCHAR(100),
    prix DECIMAL(10,2)
);
GO

INSERT INTO produits (nom, prix) VALUES
    ('Ordinateur', 999.99),
    ('Souris', 19.99),
    ('Clavier', 49.99);
GO

-- Créer un backup de cette base
BACKUP DATABASE test_backup_db
TO DISK = '/var/opt/mssql/backups/test_backup_db.bak'
WITH FORMAT, INIT, NAME = 'Full Backup de test_backup_db';
GO
```

**Vérifier que le fichier a été créé :**

```bash
# Sur votre machine
ls -lh ./backups/

# Vous devriez voir : test_backup_db.bak
```

---

## 🔍 Étape 3 : Inspecter le contenu d'un backup

Avant de restaurer, il est utile de voir ce que contient le backup.

### A. Se connecter au serveur

```bash
docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U sa \
  -P "VotreMotDePasse123!"
```

### B. Lister le contenu du backup

```sql
-- Voir les informations générales du backup
RESTORE HEADERONLY
FROM DISK = '/var/opt/mssql/backups/test_backup_db.bak';
GO
```

**Informations importantes affichées :**
- `DatabaseName` : Nom de la base de données
- `BackupName` : Description du backup
- `BackupSize` : Taille du backup
- `BackupStartDate` : Date de création
- `BackupFinishDate` : Date de fin

### C. Lister les fichiers logiques

```sql
-- Voir les fichiers contenus dans le backup
RESTORE FILELISTONLY
FROM DISK = '/var/opt/mssql/backups/test_backup_db.bak';
GO
```

**Colonnes importantes :**
- `LogicalName` : Nom logique du fichier (ex: `test_backup_db`, `test_backup_db_log`)
- `PhysicalName` : Chemin d'origine du fichier
- `Type` : `D` = Data (données), `L` = Log (journal)
- `Size` : Taille en octets

> 💡 **Note** : Les noms logiques seront nécessaires pour la restauration !

**Exemple de sortie :**

```
LogicalName          PhysicalName                                    Type  Size
-------------------- ----------------------------------------------- ----- ------------
test_backup_db       C:\Program Files\...\test_backup_db.mdf         D     8388608
test_backup_db_log   C:\Program Files\...\test_backup_db_log.ldf     L     8388608
```

---

## ♻️ Étape 4 : Restaurer le backup (cas simple)

### A. Restauration basique

Si vous restaurez sous le **même nom** et qu'aucune base du même nom n'existe :

```sql
-- Restauration simple
RESTORE DATABASE test_backup_db
FROM DISK = '/var/opt/mssql/backups/test_backup_db.bak'
WITH REPLACE;
GO
```

**Options :**
- `REPLACE` : Remplace la base si elle existe déjà
- Sans `REPLACE`, vous aurez une erreur si la base existe

### B. Vérifier la restauration

```sql
-- Vérifier que la base existe
SELECT name, state_desc, create_date
FROM sys.databases
WHERE name = 'test_backup_db';
GO

-- Vérifier le contenu
USE test_backup_db;
GO

SELECT * FROM produits;
GO
```

**Sortie attendue :**
```
id  nom          prix
--- ------------ -------
1   Ordinateur   999.99
2   Souris       19.99
3   Clavier      49.99
```

✅ La restauration a réussi !

---

## 🎨 Étape 5 : Restaurer avec un nouveau nom

Souvent, vous voulez restaurer un backup sans écraser la base existante (par exemple, pour créer une copie de test).

### A. Restaurer sous un nom différent

```sql
-- Restaurer en renommant la base
RESTORE DATABASE test_backup_db_copie
FROM DISK = '/var/opt/mssql/backups/test_backup_db.bak'
WITH MOVE 'test_backup_db' TO '/var/opt/mssql/data/test_backup_db_copie.mdf',
     MOVE 'test_backup_db_log' TO '/var/opt/mssql/data/test_backup_db_copie_log.ldf',
     REPLACE;
GO
```

**Détails importants :**

1. `RESTORE DATABASE test_backup_db_copie` : Nouveau nom de la base
2. `MOVE 'test_backup_db'` : Nom logique du fichier de données (obtenu avec `FILELISTONLY`)
3. `TO '/var/opt/mssql/data/...'` : Nouveau chemin physique
4. `MOVE 'test_backup_db_log'` : Nom logique du fichier log
5. `REPLACE` : Permet d'écraser si la base existe

> ⚠️ **Attention** : Les noms logiques (`'test_backup_db'` et `'test_backup_db_log'`) doivent correspondre exactement à ceux affichés par `RESTORE FILELISTONLY` !

### B. Vérifier la copie

```sql
-- Lister les bases
SELECT name FROM sys.databases;
GO

-- Tester la nouvelle base
USE test_backup_db_copie;
GO

SELECT * FROM produits;
GO
```

Vous avez maintenant **deux bases** : l'originale et la copie ! 🎉

---

## 🔧 Étape 6 : Restauration avec gestion des erreurs

### A. Script complet avec vérifications

```sql
USE master;
GO

-- 1. Vérifier si la base existe déjà
IF EXISTS (SELECT name FROM sys.databases WHERE name = 'ma_base_restauree')
BEGIN
    -- Mettre la base en mode SINGLE_USER pour fermer les connexions
    ALTER DATABASE ma_base_restauree SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

    -- Supprimer la base
    DROP DATABASE ma_base_restauree;
    PRINT 'Base existante supprimée';
END
GO

-- 2. Restaurer le backup
RESTORE DATABASE ma_base_restauree
FROM DISK = '/var/opt/mssql/backups/test_backup_db.bak'
WITH
    MOVE 'test_backup_db' TO '/var/opt/mssql/data/ma_base_restauree.mdf',
    MOVE 'test_backup_db_log' TO '/var/opt/mssql/data/ma_base_restauree_log.ldf',
    REPLACE,
    STATS = 10;  -- Affiche la progression tous les 10%
GO

-- 3. Vérifier la restauration
IF EXISTS (SELECT name FROM sys.databases WHERE name = 'ma_base_restauree')
BEGIN
    PRINT 'Restauration réussie !';

    -- Remettre en mode multi-utilisateur
    ALTER DATABASE ma_base_restauree SET MULTI_USER;

    -- Afficher des infos
    SELECT
        name,
        state_desc,
        recovery_model_desc,
        compatibility_level
    FROM sys.databases
    WHERE name = 'ma_base_restauree';
END
ELSE
BEGIN
    PRINT 'Erreur : La restauration a échoué';
END
GO
```

**Options utiles :**
- `STATS = 10` : Affiche la progression (10% = tous les 10%)
- `SINGLE_USER` : Force la fermeture des connexions actives
- `ROLLBACK IMMEDIATE` : Annule les transactions en cours immédiatement

---

## 🌐 Étape 7 : Restaurer un backup externe (via copie)

Si votre backup n'est pas dans le dossier `./backups`, vous devez le copier dans le conteneur.

### A. Copier un fichier depuis votre machine vers le conteneur

```bash
# Syntaxe : docker cp <source_hôte> <conteneur>:<destination_conteneur>
docker cp /chemin/vers/backup.bak mssql_dev:/var/opt/mssql/backups/backup.bak
```

**Exemple concret :**

```bash
# Copier depuis votre bureau vers le conteneur
docker cp ~/Desktop/production_backup.bak mssql_dev:/var/opt/mssql/backups/production_backup.bak
```

### B. Vérifier que le fichier est présent

```bash
# Lister les fichiers dans le conteneur
docker exec mssql_dev ls -lh /var/opt/mssql/backups/
```

### C. Restaurer le backup copié

```sql
RESTORE DATABASE production_db
FROM DISK = '/var/opt/mssql/backups/production_backup.bak'
WITH
    MOVE 'production_db' TO '/var/opt/mssql/data/production_db.mdf',
    MOVE 'production_db_log' TO '/var/opt/mssql/data/production_db_log.ldf',
    REPLACE,
    STATS = 10;
GO
```

---

## 📊 Étape 8 : Restaurer plusieurs backups (script automatisé)

Si vous avez plusieurs backups à restaurer, voici un script pratique :

### A. Script PowerShell (Windows)

```powershell
# Liste des backups à restaurer
$backups = @(
    @{File="backup1.bak"; DbName="database1"},
    @{File="backup2.bak"; DbName="database2"},
    @{File="backup3.bak"; DbName="database3"}
)

foreach ($backup in $backups) {
    Write-Host "Copie de $($backup.File)..."
    docker cp "./backups/$($backup.File)" mssql_dev:/var/opt/mssql/backups/

    Write-Host "Restauration de $($backup.DbName)..."

    $sql = @"
RESTORE DATABASE [$($backup.DbName)]
FROM DISK = '/var/opt/mssql/backups/$($backup.File)'
WITH REPLACE, STATS = 10;
GO
"@

    $sql | docker exec -i mssql_dev /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "VotreMotDePasse123!"
}

Write-Host "Toutes les restaurations sont terminées !"
```

### B. Script Bash (Linux/macOS)

```bash
#!/bin/bash

# Liste des backups
declare -A backups=(
    ["backup1.bak"]="database1"
    ["backup2.bak"]="database2"
    ["backup3.bak"]="database3"
)

for file in "${!backups[@]}"; do
    db_name="${backups[$file]}"

    echo "Copie de $file..."
    docker cp "./backups/$file" mssql_dev:/var/opt/mssql/backups/

    echo "Restauration de $db_name..."

    docker exec mssql_dev /opt/mssql-tools/bin/sqlcmd \
        -S localhost -U sa -P "VotreMotDePasse123!" \
        -Q "RESTORE DATABASE [$db_name] FROM DISK = '/var/opt/mssql/backups/$file' WITH REPLACE, STATS = 10;"
done

echo "Toutes les restaurations sont terminées !"
```

---

## 🔐 Étape 9 : Restaurer avec les utilisateurs (Orphan Users)

### A. Problème des "Orphan Users"

Après une restauration, les utilisateurs de la base de données peuvent être "orphelins" (leurs logins n'existent pas sur le nouveau serveur).

**Diagnostic :**

```sql
USE ma_base_restauree;
GO

-- Lister les orphan users
SELECT
    dp.name AS user_name,
    dp.type_desc,
    dp.create_date
FROM sys.database_principals dp
LEFT JOIN sys.server_principals sp ON dp.sid = sp.sid
WHERE dp.type = 'S'  -- SQL User
  AND sp.sid IS NULL  -- Pas de login correspondant
  AND dp.name NOT IN ('guest', 'INFORMATION_SCHEMA', 'sys', 'dbo');
GO
```

### B. Solution 1 : Créer les logins manquants

```sql
-- Créer le login sur le serveur
USE master;
GO

CREATE LOGIN app_user WITH PASSWORD = 'AppPassword123!';
GO

-- Lier le user existant au nouveau login
USE ma_base_restauree;
GO

ALTER USER app_user WITH LOGIN = app_user;
GO
```

### C. Solution 2 : Utiliser sp_change_users_login (ancienne méthode)

```sql
USE ma_base_restauree;
GO

-- Réparer automatiquement le lien
EXEC sp_change_users_login 'Auto_Fix', 'app_user';
GO
```

### D. Solution 3 : Script complet de réparation

```sql
USE ma_base_restauree;
GO

-- Créer une procédure pour réparer tous les orphan users
DECLARE @UserName NVARCHAR(128);
DECLARE @SQL NVARCHAR(MAX);

DECLARE orphan_cursor CURSOR FOR
SELECT dp.name
FROM sys.database_principals dp
LEFT JOIN sys.server_principals sp ON dp.sid = sp.sid
WHERE dp.type = 'S'
  AND sp.sid IS NULL
  AND dp.name NOT IN ('guest', 'INFORMATION_SCHEMA', 'sys', 'dbo');

OPEN orphan_cursor;
FETCH NEXT FROM orphan_cursor INTO @UserName;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Réparation de : ' + @UserName;

    SET @SQL = 'ALTER USER [' + @UserName + '] WITH LOGIN = [' + @UserName + ']';

    BEGIN TRY
        EXEC sp_executesql @SQL;
        PRINT 'Succès !';
    END TRY
    BEGIN CATCH
        PRINT 'Erreur : Le login ' + @UserName + ' n''existe pas. Créez-le d''abord.';
    END CATCH

    FETCH NEXT FROM orphan_cursor INTO @UserName;
END

CLOSE orphan_cursor;
DEALLOCATE orphan_cursor;
GO
```

---

## 🛠️ Options avancées de restauration

### A. Restaurer en mode NO RECOVERY (pour restaurations multiples)

Utilisé pour restaurer un Full Backup suivi de backups différentiels ou logs :

```sql
-- 1. Restaurer le Full Backup sans finaliser
RESTORE DATABASE ma_base
FROM DISK = '/var/opt/mssql/backups/full_backup.bak'
WITH NORECOVERY, REPLACE;
GO

-- 2. Restaurer un backup différentiel (si vous en avez)
RESTORE DATABASE ma_base
FROM DISK = '/var/opt/mssql/backups/diff_backup.bak'
WITH NORECOVERY;
GO

-- 3. Finaliser (rendre la base utilisable)
RESTORE DATABASE ma_base WITH RECOVERY;
GO
```

### B. Restaurer à un point dans le temps (Point-in-Time Recovery)

```sql
-- Nécessite des backups de logs de transactions
RESTORE DATABASE ma_base
FROM DISK = '/var/opt/mssql/backups/full_backup.bak'
WITH NORECOVERY, REPLACE;
GO

RESTORE LOG ma_base
FROM DISK = '/var/opt/mssql/backups/log_backup.trn'
WITH STOPAT = '2025-10-30 14:30:00', RECOVERY;
GO
```

### C. Restaurer en lecture seule

```sql
RESTORE DATABASE ma_base_readonly
FROM DISK = '/var/opt/mssql/backups/backup.bak'
WITH REPLACE;
GO

-- Mettre en lecture seule
ALTER DATABASE ma_base_readonly SET READ_ONLY;
GO
```

---

## 🔍 Dépannage

### Problème : "The backup set holds a backup of a database other than the existing database"

**Cause** : Vous essayez de restaurer sur une base qui existe déjà avec un nom différent.

**Solution** : Utilisez `REPLACE` ou supprimez la base existante :

```sql
DROP DATABASE ma_base;
GO

RESTORE DATABASE ma_base
FROM DISK = '/var/opt/mssql/backups/backup.bak';
GO
```

### Problème : "Cannot open backup device... Operating system error 3"

**Cause** : Le fichier `.bak` n'existe pas au chemin spécifié.

**Solutions** :

```bash
# Vérifier que le fichier existe dans le conteneur
docker exec mssql_dev ls -l /var/opt/mssql/backups/

# Si absent, le copier
docker cp ./backups/backup.bak mssql_dev:/var/opt/mssql/backups/
```

### Problème : "Exclusive access could not be obtained"

**Cause** : Des connexions sont ouvertes sur la base de données.

**Solution** : Forcer la fermeture des connexions :

```sql
USE master;
GO

ALTER DATABASE ma_base SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
GO

RESTORE DATABASE ma_base
FROM DISK = '/var/opt/mssql/backups/backup.bak'
WITH REPLACE;
GO

ALTER DATABASE ma_base SET MULTI_USER;
GO
```

### Problème : "Incorrect or damaged backup"

**Cause** : Le fichier `.bak` est corrompu ou incomplet.

**Diagnostic** :

```sql
-- Vérifier l'intégrité du backup
RESTORE VERIFYONLY
FROM DISK = '/var/opt/mssql/backups/backup.bak';
GO
```

**Solutions** :
- Retélécharger le fichier de backup
- Vérifier la copie (taille, checksum)
- Utiliser un autre backup

### Problème : "Logical file 'X' is not part of database 'Y'"

**Cause** : Les noms logiques dans la commande `MOVE` ne correspondent pas au backup.

**Solution** : Vérifier les vrais noms avec `FILELISTONLY` :

```sql
RESTORE FILELISTONLY
FROM DISK = '/var/opt/mssql/backups/backup.bak';
GO

-- Utiliser les VRAIS noms logiques dans MOVE
```

---

## 📊 Monitoring de la restauration

### Voir la progression en temps réel

```sql
-- Dans une autre fenêtre SQL, pendant la restauration
SELECT
    r.session_id,
    r.command,
    r.percent_complete,
    r.total_elapsed_time / 60000 AS elapsed_minutes,
    r.estimated_completion_time / 60000 AS remaining_minutes,
    CONVERT(VARCHAR(20), DATEADD(ms, r.estimated_completion_time, GETDATE()), 120) AS estimated_end_time
FROM sys.dm_exec_requests r
WHERE r.command = 'RESTORE DATABASE';
GO
```

---

## 💡 Bonnes pratiques

### 1. Toujours vérifier le contenu avant de restaurer

```sql
-- Étape 1 : Informations générales
RESTORE HEADERONLY FROM DISK = '/path/to/backup.bak';

-- Étape 2 : Fichiers logiques
RESTORE FILELISTONLY FROM DISK = '/path/to/backup.bak';

-- Étape 3 : Vérifier l'intégrité
RESTORE VERIFYONLY FROM DISK = '/path/to/backup.bak';
```

### 2. Documenter les restaurations

Créez un fichier `RESTORES.md` :

```markdown
# Restaurations effectuées

## 2025-10-30 : Base client_db
- Backup : client_db_20251029.bak
- Taille : 2.5 GB
- Durée : 5 minutes
- Source : Production serveur PROD-01
- Users orphelins réparés : app_user, reporting_user
```

### 3. Tester après restauration

```sql
-- Vérifier la cohérence de la base
DBCC CHECKDB ('ma_base_restauree') WITH NO_INFOMSGS;
GO

-- Vérifier les statistiques
USE ma_base_restauree;
GO

SELECT
    OBJECT_NAME(object_id) AS table_name,
    SUM(row_count) AS total_rows
FROM sys.dm_db_partition_stats
WHERE index_id IN (0,1)
GROUP BY object_id
ORDER BY total_rows DESC;
GO
```

### 4. Automatiser les restaurations fréquentes

Créez un script shell/PowerShell pour les environnements de développement :

```bash
#!/bin/bash
# restore_dev_db.sh

echo "Téléchargement du dernier backup..."
wget https://backups.monsite.com/latest.bak -O ./backups/latest.bak

echo "Restauration..."
docker exec mssql_dev /opt/mssql-tools/bin/sqlcmd \
    -S localhost -U sa -P "$SA_PASSWORD" \
    -Q "RESTORE DATABASE dev_db FROM DISK = '/var/opt/mssql/backups/latest.bak' WITH REPLACE, STATS = 10;"

echo "Terminé !"
```

---

## 📚 Aller plus loin

- **[Gestion des utilisateurs](03-gestion-utilisateurs-logins.md)** : Gérer les orphan users après restauration
- **[Configuration avec IP fixe](02-config-ip-fixe.md)** : Environnement réseau stable
- **[Annexe C - Gestion des volumes](../annexes/C-gestion-volumes.md)** : Backups et stockage Docker

---

## 📝 Récapitulatif

Vous avez appris à :
- ✅ Configurer Docker pour recevoir des fichiers de backup
- ✅ Inspecter le contenu d'un backup (HEADERONLY, FILELISTONLY)
- ✅ Restaurer une base de données complète
- ✅ Restaurer avec un nouveau nom (option MOVE)
- ✅ Copier des backups dans le conteneur (docker cp)
- ✅ Gérer les orphan users après restauration
- ✅ Utiliser des options avancées (NO RECOVERY, Point-in-Time)
- ✅ Dépanner les problèmes courants
- ✅ Monitorer la progression de la restauration
- ✅ Automatiser les restaurations répétitives

---

## 🎯 Points clés à retenir

1. **Toujours vérifier** le contenu du backup avec `FILELISTONLY` avant de restaurer
2. Utiliser **`MOVE`** pour changer les noms de fichiers physiques
3. Utiliser **`REPLACE`** pour écraser une base existante
4. Les **noms logiques** dans `MOVE` doivent correspondre exactement au backup
5. Ajouter un **volume partagé** (`./backups`) simplifie grandement les restaurations
6. **Réparer les orphan users** après restauration avec `ALTER USER ... WITH LOGIN`
7. **`STATS = 10`** affiche la progression (utile pour les gros backups)
8. Utiliser **`SINGLE_USER`** pour forcer la fermeture des connexions
9. **Tester** la base après restauration (`DBCC CHECKDB`)
10. **Documenter** toutes les restaurations effectuées

---

## 🔗 Commandes essentielles (mémo)

```sql
-- Inspecter un backup
RESTORE HEADERONLY FROM DISK = '/path/backup.bak';
RESTORE FILELISTONLY FROM DISK = '/path/backup.bak';
RESTORE VERIFYONLY FROM DISK = '/path/backup.bak';

-- Restauration simple
RESTORE DATABASE ma_base FROM DISK = '/path/backup.bak' WITH REPLACE;

-- Restauration avec nouveau nom
RESTORE DATABASE nouvelle_base
FROM DISK = '/path/backup.bak'
WITH MOVE 'nom_logique_data' TO '/path/nouveau.mdf',
     MOVE 'nom_logique_log' TO '/path/nouveau_log.ldf',
     REPLACE;

-- Copier un backup dans le conteneur
docker cp backup.bak mssql_dev:/var/opt/mssql/backups/

-- Réparer orphan user
ALTER USER nom_user WITH LOGIN = nom_login;
```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
