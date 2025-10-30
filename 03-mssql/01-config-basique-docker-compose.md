# 3.1 Configuration basique avec docker-compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Microsoft SQL Server (souvent appelé **MS SQL** ou **MSSQL**) est un système de gestion de base de données relationnelle développé par Microsoft. Il est très utilisé dans les environnements Windows et les applications .NET, mais il fonctionne également très bien sur Linux grâce à Docker.

Dans ce tutoriel, nous allons apprendre à :
- Démarrer un conteneur MS SQL Server avec Docker Compose
- Comprendre les variables d'environnement essentielles
- Se connecter à la base de données
- Gérer le conteneur (arrêter, redémarrer, supprimer)

---

## 🎯 Ce que vous allez obtenir

À la fin de ce tutoriel, vous aurez :
- ✅ Un serveur MS SQL Server opérationnel
- ✅ Accessible sur `localhost:1433`
- ✅ Avec un utilisateur `sa` (administrateur système)
- ✅ Des données persistantes dans un volume Docker
- ✅ La possibilité de vous connecter avec un client SQL

---

## 📋 Prérequis

Avant de commencer, assurez-vous d'avoir :
- Docker installé sur votre machine ([guide d'installation](https://docs.docker.com/get-docker/))
- Docker Compose installé (généralement inclus avec Docker Desktop)
- Un éditeur de texte (VS Code, Notepad++, Sublime Text...)
- Environ 2 Go d'espace disque disponible

### Vérifier l'installation

Ouvrez un terminal et exécutez :

```bash
# Vérifier Docker
docker --version
# Résultat attendu : Docker version 20.10.x ou supérieur

# Vérifier Docker Compose
docker-compose --version
# Résultat attendu : Docker Compose version 2.x ou supérieur
```

---

## 📁 Étape 1 : Créer le dossier du projet

Créons d'abord un dossier pour notre projet MS SQL Server.

```bash
# Créer un dossier pour le projet
mkdir mssql-docker

# Se placer dans ce dossier
cd mssql-docker
```

---

## 📝 Étape 2 : Créer le fichier docker-compose.yml

Dans le dossier `mssql-docker`, créez un fichier nommé `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  mssql:
    # Image officielle de Microsoft SQL Server 2022
    image: mcr.microsoft.com/mssql/server:2022-latest

    # Nom du conteneur (plus facile à identifier)
    container_name: mssql_dev

    # Redémarrage automatique (sauf arrêt manuel)
    restart: unless-stopped

    # Variables d'environnement
    environment:
      # Accepter le contrat de licence (obligatoire)
      ACCEPT_EULA: "Y"

      # Édition de SQL Server (Developer = gratuite pour le développement)
      MSSQL_PID: "Developer"

      # Mot de passe de l'utilisateur 'sa' (administrateur)
      # ⚠️ CHANGEZ CE MOT DE PASSE !
      # Exigences : 8+ caractères, majuscules, minuscules, chiffres, caractères spéciaux
      SA_PASSWORD: "VotreMotDePasse123!"

    # Exposition du port
    ports:
      # Port hôte:Port conteneur
      - "1433:1433"

    # Volume pour la persistance des données
    volumes:
      - mssql_data:/var/opt/mssql

# Déclaration du volume
volumes:
  mssql_data:
```

---

## 🔍 Comprendre le fichier docker-compose.yml

Décortiquons les éléments importants du fichier :

### 📦 L'image Docker

```yaml
image: mcr.microsoft.com/mssql/server:2022-latest
```

- `mcr.microsoft.com` : Registre Microsoft Container Registry
- `mssql/server` : Image officielle de SQL Server
- `2022-latest` : Version de SQL Server (2022 est la plus récente)

**Autres versions disponibles :**
- `2019-latest` : SQL Server 2019
- `2017-latest` : SQL Server 2017

### 🔐 Les variables d'environnement

#### `ACCEPT_EULA: "Y"`
- **Obligatoire** : Accepte le contrat de licence Microsoft
- Sans cette ligne, le conteneur ne démarrera pas

#### `MSSQL_PID: "Developer"`
- Définit l'édition de SQL Server à utiliser
- **Options disponibles :**
  - `Developer` : Gratuite, pour développement/test uniquement
  - `Express` : Gratuite, limitée (10 Go de stockage max)
  - `Standard` : Payante (nécessite une licence)
  - `Enterprise` : Payante (nécessite une licence)

#### `SA_PASSWORD: "VotreMotDePasse123!"`
- Mot de passe de l'utilisateur `sa` (System Administrator)
- **Exigences de sécurité :**
  - Au moins 8 caractères
  - Au moins 3 des 4 catégories : majuscules, minuscules, chiffres, caractères spéciaux
  - Ne doit pas contenir le nom d'utilisateur (`sa`)

> ⚠️ **Sécurité** : Ne JAMAIS utiliser ce mot de passe exemple en production ! Changez-le immédiatement.

### 🌐 Les ports

```yaml
ports:
  - "1433:1433"
```

- `1433` est le port standard de MS SQL Server
- Le premier `1433` : port sur votre machine (hôte)
- Le second `1433` : port dans le conteneur
- Format : `"port_hôte:port_conteneur"`

### 💾 Les volumes

```yaml
volumes:
  - mssql_data:/var/opt/mssql
```

- `mssql_data` : Nom du volume Docker (stockage persistant)
- `/var/opt/mssql` : Chemin dans le conteneur où SQL Server stocke ses données
- **Important** : Sans volume, toutes vos données disparaîtront si vous supprimez le conteneur !

---

## ▶️ Étape 3 : Démarrer MS SQL Server

Dans votre terminal, depuis le dossier `mssql-docker`, exécutez :

```bash
docker-compose up -d
```

**Détails de la commande :**
- `up` : Démarre les services définis dans docker-compose.yml
- `-d` : Mode "detached" (arrière-plan)

**Sortie attendue :**
```
[+] Running 2/2
 ✔ Network mssql-docker_default  Created
 ✔ Container mssql_dev           Started
```

### 📊 Vérifier que le conteneur fonctionne

```bash
docker-compose ps
```

**Sortie attendue :**
```
NAME        IMAGE                                       STATUS        PORTS
mssql_dev   mcr.microsoft.com/mssql/server:2022-latest  Up 10 seconds 0.0.0.0:1433->1433/tcp
```

Le statut doit afficher `Up` (en fonctionnement).

### 📋 Voir les logs

Pour vérifier que SQL Server a bien démarré :

```bash
docker-compose logs -f
```

**Message important à repérer :**
```
SQL Server is now ready for client connections.
```

Appuyez sur `Ctrl+C` pour quitter l'affichage des logs.

---

## 🔌 Étape 4 : Se connecter à MS SQL Server

### A. Depuis la ligne de commande (sqlcmd)

**Se connecter au conteneur :**

```bash
docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
  -S localhost \
  -U sa \
  -P "VotreMotDePasse123!"
```

**Détails de la commande :**
- `docker exec -it` : Exécute une commande dans le conteneur
- `mssql_dev` : Nom du conteneur
- `/opt/mssql-tools/bin/sqlcmd` : Outil de ligne de commande SQL Server
- `-S localhost` : Serveur (Server)
- `-U sa` : Utilisateur (User)
- `-P "..."` : Mot de passe (Password)

**Tester une requête :**

```sql
-- Lister les bases de données
SELECT name FROM sys.databases;
GO

-- Quitter
EXIT
```

> 💡 **Astuce** : Dans SQL Server, les commandes doivent être suivies de `GO` pour être exécutées.

### B. Depuis un client SQL graphique

Vous pouvez utiliser différents clients :

#### 1. **Azure Data Studio** (recommandé, gratuit)
- Télécharger : https://aka.ms/azuredatastudio
- Multi-plateforme (Windows, macOS, Linux)

#### 2. **SQL Server Management Studio (SSMS)** (Windows uniquement)
- Télécharger : https://aka.ms/ssmsfullsetup
- Plus complet mais lourd

#### 3. **DBeaver** (gratuit, multi-BDD)
- Télécharger : https://dbeaver.io/

**Paramètres de connexion :**

| Paramètre | Valeur |
|-----------|--------|
| **Serveur / Hôte** | `localhost` ou `127.0.0.1` |
| **Port** | `1433` |
| **Authentification** | SQL Server Authentication |
| **Utilisateur** | `sa` |
| **Mot de passe** | `VotreMotDePasse123!` |

---

## 🎨 Étape 5 : Créer votre première base de données

Une fois connecté avec `sqlcmd` ou un client graphique :

```sql
-- Créer une nouvelle base de données
CREATE DATABASE ma_premiere_bdd;
GO

-- Vérifier qu'elle a été créée
SELECT name FROM sys.databases WHERE name = 'ma_premiere_bdd';
GO

-- Utiliser cette base de données
USE ma_premiere_bdd;
GO

-- Créer une table simple
CREATE TABLE utilisateurs (
    id INT PRIMARY KEY IDENTITY(1,1),
    nom NVARCHAR(100),
    email NVARCHAR(255),
    date_creation DATETIME DEFAULT GETDATE()
);
GO

-- Insérer des données
INSERT INTO utilisateurs (nom, email)
VALUES
    ('Alice Dupont', 'alice@example.com'),
    ('Bob Martin', 'bob@example.com');
GO

-- Lire les données
SELECT * FROM utilisateurs;
GO
```

---

## 🛠️ Commandes de gestion du conteneur

### Arrêter le conteneur (sans supprimer les données)

```bash
docker-compose stop
```

Le conteneur est arrêté, mais vos données restent sauvegardées dans le volume.

### Redémarrer le conteneur

```bash
docker-compose start
```

Vos données sont toujours là !

### Voir les logs en temps réel

```bash
docker-compose logs -f
```

### Voir l'état du conteneur

```bash
docker-compose ps
```

### Redémarrer complètement

```bash
docker-compose restart
```

---

## 🗑️ Suppression complète (ATTENTION : perte de données)

### Arrêter et supprimer le conteneur (données préservées)

```bash
docker-compose down
```

Le conteneur est supprimé, mais le volume `mssql_data` existe toujours.

### Supprimer également le volume (⚠️ PERTE DE DONNÉES)

```bash
docker-compose down -v
```

Ou manuellement :

```bash
# Lister les volumes
docker volume ls

# Supprimer le volume spécifique
docker volume rm mssql-docker_mssql_data
```

---

## 🔍 Dépannage

### Problème : Le conteneur ne démarre pas

**Vérifier les logs :**
```bash
docker-compose logs
```

**Causes fréquentes :**
1. **Mot de passe trop faible**
   - Solution : Utilisez un mot de passe respectant les exigences

2. **Port 1433 déjà utilisé**
   - Vérifier : `netstat -an | grep 1433` (Linux/macOS) ou `netstat -an | findstr 1433` (Windows)
   - Solution : Changer le port dans docker-compose.yml : `"1434:1433"`

3. **Manque de mémoire**
   - SQL Server nécessite au moins 2 Go de RAM
   - Solution : Vérifier les ressources Docker Desktop

### Problème : Impossible de se connecter

**Vérifier que le conteneur fonctionne :**
```bash
docker-compose ps
```

**Vérifier que le port est bien exposé :**
```bash
docker port mssql_dev
```

**Tester la connexion réseau :**
```bash
telnet localhost 1433
# Ou avec nc (netcat)
nc -zv localhost 1433
```

### Problème : Mot de passe refusé

- Vérifiez que vous utilisez bien le mot de passe défini dans `SA_PASSWORD`
- Le mot de passe est sensible à la casse
- Vérifiez qu'il n'y a pas d'espaces avant/après

---

## 📊 Utilisation des ressources

MS SQL Server est relativement gourmand en ressources :

**Minimum recommandé :**
- **RAM** : 2 Go minimum
- **CPU** : 2 cœurs
- **Disque** : 6 Go (image + données)

**Configuration Docker Desktop :**
- Ouvrir Docker Desktop → Settings → Resources
- Allouer au moins 4 Go de RAM pour un bon confort

---

## 💡 Bonnes pratiques

### 1. Utiliser un fichier .env pour les secrets

Au lieu de mettre le mot de passe directement dans `docker-compose.yml`, créez un fichier `.env` :

**Fichier `.env` :**
```env
SA_PASSWORD=VotreMotDePasse123!
```

**docker-compose.yml modifié :**
```yaml
environment:
  ACCEPT_EULA: "Y"
  MSSQL_PID: "Developer"
  SA_PASSWORD: ${SA_PASSWORD}
```

**N'oubliez pas d'ajouter `.env` à votre `.gitignore` !**

### 2. Nommer explicitement les volumes

```yaml
volumes:
  mssql_data:
    name: mssql_dev_data
```

Cela facilite l'identification du volume.

### 3. Limiter les ressources (optionnel)

```yaml
services:
  mssql:
    # ... autres configurations
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          memory: 2G
```

---

## 📚 Aller plus loin

Maintenant que vous avez une configuration basique fonctionnelle, vous pouvez explorer :

- **[Configuration avec IP fixe](02-config-ip-fixe.md)** : Assigner une adresse IP statique au conteneur
- **[Gestion des utilisateurs](03-gestion-utilisateurs-logins.md)** : Créer des utilisateurs avec des permissions limitées
- **[Restauration de backup](04-restauration-backup.md)** : Importer des bases de données existantes

---

## 📝 Récapitulatif

Vous avez appris à :
- ✅ Créer un fichier `docker-compose.yml` pour MS SQL Server
- ✅ Comprendre les variables d'environnement essentielles
- ✅ Démarrer et gérer un conteneur SQL Server
- ✅ Se connecter à la base de données (ligne de commande et GUI)
- ✅ Créer une base de données et des tables
- ✅ Gérer la persistance des données avec les volumes
- ✅ Dépanner les problèmes courants

---

## 🎯 Points clés à retenir

1. **ACCEPT_EULA** est obligatoire pour SQL Server
2. Le mot de passe `SA_PASSWORD` doit être fort (8+ caractères, complexe)
3. Le port standard de SQL Server est **1433**
4. Les données sont stockées dans `/var/opt/mssql` dans le conteneur
5. Toujours utiliser un volume pour persister les données
6. L'édition **Developer** est gratuite pour le développement

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
