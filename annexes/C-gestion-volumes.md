# Annexe C - Gestion des Volumes Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Les **volumes Docker** sont le mécanisme de persistance des données pour vos conteneurs. Sans volume, toutes les données d'un conteneur sont perdues quand il est supprimé. Comprendre les volumes est essentiel pour ne jamais perdre vos données !

**Ce que vous allez apprendre :**
- 💾 Comprendre la persistance des données dans Docker
- 📦 Différences entre volumes nommés et bind mounts
- 💿 Sauvegarder et restaurer vos données
- 🧹 Nettoyer les volumes inutilisés
- 🔧 Gérer les volumes avec Docker Compose

**Niveau :** 🟡 Intermédiaire (mais expliqué pour débutants)

**Durée de lecture :** 35 minutes

---

## 📑 Table des Matières

1. [Comprendre le Problème de Persistance](#-1-comprendre-le-problème-de-persistance)
2. [Types de Stockage Docker](#-2-types-de-stockage-docker)
3. [Volumes Nommés](#-3-volumes-nommés)
4. [Bind Mounts](#-4-bind-mounts)
5. [Comparaison et Choix](#-5-comparaison-et-choix)
6. [Sauvegarde et Restauration](#-6-sauvegarde-et-restauration)
7. [Nettoyage des Volumes](#-7-nettoyage-des-volumes)
8. [Cas d'Usage Pratiques](#-8-cas-dusage-pratiques)

---

## 💥 1. Comprendre le Problème de Persistance

### 1.1 Le Comportement par Défaut

**Sans volume, les données sont éphémères :**

```
┌──────────────────────────────────────────┐
│  Conteneur MariaDB (sans volume)         │
│                                          │
│  📝 Base de données avec 1000 users      │
│  📊 Tables, données...                   │
└──────────────────────────────────────────┘
            ↓
    docker rm conteneur
            ↓
        💥 POOF !
            ↓
    Toutes les données perdues !
```

**Scénario catastrophe :**

```bash
# Jour 1 : Vous créez une base de données
docker run -d --name ma_base mariadb

# Vous ajoutez 1000 utilisateurs...
# Vous travaillez pendant des jours...

# Jour 5 : Vous supprimez le conteneur
docker rm -f ma_base

# 😱 Toutes vos données ont disparu !
```

---

### 1.2 La Solution : Les Volumes

**Avec un volume, les données persistent :**

```
┌──────────────────────────────────────────┐
│  Conteneur MariaDB                       │
│      ↓                                   │
│  📂 Volume Docker (mariadb_data)         │
│  📝 Base de données avec 1000 users      │
│  📊 Tables, données...                   │
└──────────────────────────────────────────┘
            ↓
    docker rm conteneur
            ↓
        ✅ Conteneur supprimé
            ↓
    📂 Volume toujours là !
            ↓
    🎉 Données préservées !
```

**Avec volume :**

```bash
# Créer un conteneur avec volume
docker run -d --name ma_base \
  -v mariadb_data:/var/lib/mysql \
  mariadb

# Supprimer le conteneur
docker rm -f ma_base

# ✅ Les données sont toujours dans le volume !

# Recréer le conteneur
docker run -d --name ma_base_nouveau \
  -v mariadb_data:/var/lib/mysql \
  mariadb

# 🎉 Toutes les données sont revenues !
```

---

### 1.3 Où Vivent les Données ?

**Système de fichiers du conteneur (éphémère) :**
```
/var/lib/docker/containers/<id>/  ← Supprimé avec le conteneur
```

**Volume Docker (persistant) :**
```
/var/lib/docker/volumes/<nom>/    ← Reste même après suppression
```

---

## 📦 2. Types de Stockage Docker

Docker propose **trois méthodes** pour persister des données.

### 2.1 Vue d'Ensemble

```
┌────────────────────────────────────────────────────────┐
│  Machine Hôte                                          │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐   │
│  │   Volume     │  │ Bind Mount   │  │  tmpfs      │   │
│  │   Nommé      │  │              │  │  (RAM)      │   │
│  └──────────────┘  └──────────────┘  └─────────────┘   │
│         ↓                 ↓                  ↓         │
│  Géré par Docker   Dossier hôte      Mémoire RAM       │
│  (recommandé)      (dev/config)      (temporaire)      │
└────────────────────────────────────────────────────────┘
```

---

### 2.2 Tableau Comparatif

| Critère | Volume Nommé | Bind Mount | tmpfs |
|---------|--------------|------------|-------|
| **Géré par** | Docker | Vous | Système |
| **Emplacement** | `/var/lib/docker/volumes/` | N'importe où | RAM |
| **Portabilité** | ✅ Excellent | ⚠️ Dépend des chemins | N/A |
| **Performances** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Backup** | ✅ Facile | ⚠️ Manuel | ❌ Impossible |
| **Permissions** | ✅ Gérées | ⚠️ Complexes (Linux) | ✅ |
| **Use Case** | **Données de BDD** | Config, développement | Cache, secrets |

---

### 2.3 Recommandations

| Type de Données | Solution Recommandée | Pourquoi |
|-----------------|---------------------|----------|
| 🗄️ **Données de BDD** | Volume nommé | Performances, portabilité |
| ⚙️ **Fichiers de config** | Bind mount | Édition facile |
| 💻 **Code source (dev)** | Bind mount | Hot reload |
| 📄 **Logs** | Volume nommé ou bind | Selon besoin |
| 🔐 **Secrets temporaires** | tmpfs | Sécurité |

---

## 📁 3. Volumes Nommés

### 3.1 Qu'est-ce qu'un Volume Nommé ?

Un **volume nommé** est un espace de stockage géré entièrement par Docker. Vous lui donnez un nom, Docker s'occupe du reste.

**Analogie : Le coffre-fort de la banque**

```
Vous : "Je veux un coffre nommé 'mes_documents'"
Banque (Docker) : "OK, je le crée et je gère tout"
Vous : "Où est-il ?"
Banque : "Ne vous inquiétez pas, je sais où il est"
```

---

### 3.2 Créer un Volume Nommé

**Méthode 1 : Création explicite**

```bash
# Créer un volume
docker volume create mariadb_data

# Vérifier
docker volume ls

# Résultat :
# DRIVER    VOLUME NAME
# local     mariadb_data
```

---

**Méthode 2 : Création automatique (lors du `docker run`)**

```bash
# Docker créera automatiquement le volume s'il n'existe pas
docker run -d \
  --name ma_base \
  -v mariadb_data:/var/lib/mysql \
  mariadb:10.11

# Le volume est créé automatiquement
```

---

**Méthode 3 : Avec Docker Compose (RECOMMANDÉ)**

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    volumes:
      # Syntaxe : nom_volume:chemin_dans_conteneur
      - mariadb_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret

# Déclaration du volume
volumes:
  mariadb_data:
    # Docker le créera automatiquement
```

---

### 3.3 Inspecter un Volume

```bash
# Voir les détails d'un volume
docker volume inspect mariadb_data
```

**Sortie exemple :**
```json
[
    {
        "CreatedAt": "2024-10-29T10:30:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/mariadb_data/_data",
        "Name": "mariadb_data",
        "Options": {},
        "Scope": "local"
    }
]
```

**Points importants :**
- `Mountpoint` : Emplacement réel sur le disque (géré par Docker)
- `Driver` : Type de stockage (local par défaut)

---

### 3.4 Utiliser un Volume Nommé

**Syntaxe dans docker-compose.yml :**

```yaml
services:
  mariadb:
    volumes:
      - nom_volume:chemin_conteneur[:options]

volumes:
  nom_volume:
```

**Exemples :**

```yaml
# Lecture/Écriture (par défaut)
volumes:
  - mariadb_data:/var/lib/mysql

# Lecture seule
volumes:
  - mariadb_data:/var/lib/mysql:ro

# Avec options de montage
volumes:
  - mariadb_data:/var/lib/mysql:rw,z
```

---

### 3.5 Avantages des Volumes Nommés

✅ **Gestion simplifiée**
```bash
# Pas besoin de connaître le chemin physique
docker volume ls
docker volume rm mariadb_data
```

✅ **Portabilité**
```bash
# Fonctionne pareil sur Windows, Linux, macOS
# Pas de problème de chemins absolus
```

✅ **Performances optimales**
```bash
# Docker choisit le meilleur driver selon le système
```

✅ **Backup facilité**
```bash
# Docker sait exactement où sont les données
```

✅ **Pas de problèmes de permissions**
```bash
# Docker gère les permissions automatiquement
```

---

### 3.6 Volumes Nommés Avancés

**Volume avec driver personnalisé :**

```bash
# Créer un volume avec driver NFS (stockage réseau)
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs_volume
```

**Volume avec labels (métadonnées) :**

```bash
docker volume create \
  --label project=blog \
  --label env=production \
  blog_data
```

**Rechercher par label :**

```bash
# Trouver tous les volumes d'un projet
docker volume ls --filter label=project=blog
```

---

## 📂 4. Bind Mounts

### 4.1 Qu'est-ce qu'un Bind Mount ?

Un **bind mount** est un lien direct entre un dossier de votre machine et un dossier dans le conteneur. C'est comme une fenêtre qui permet au conteneur de voir vos fichiers.

**Analogie : La chambre d'étudiant**

```
Votre chambre (dossier hôte) : /home/vous/projet
        ↕️ (Bind mount : lien direct)
Chambre du conteneur : /app

Tout changement dans un sens est visible de l'autre
```

---

### 4.2 Créer un Bind Mount

**Syntaxe avec docker run :**

```bash
docker run -d \
  --name mon_app \
  -v /chemin/sur/hote:/chemin/dans/conteneur \
  mon_image
```

**Exemple concret :**

```bash
# Partager le dossier actuel avec le conteneur
docker run -d \
  --name dev_app \
  -v $(pwd)/src:/app/src \
  node:18

# $(pwd) = chemin absolu du dossier actuel
```

---

**Syntaxe avec Docker Compose :**

```yaml
version: '3.8'

services:
  app:
    image: node:18
    volumes:
      # Syntaxe : ./chemin_relatif:chemin_conteneur
      - ./src:/app/src
      - ./config:/app/config:ro  # Lecture seule
```

**Points importants :**
- ✅ Chemins relatifs possibles (`./dossier`)
- ⚠️ Chemins absolus dépendent de l'OS
- 🔧 Parfait pour le développement

---

### 4.3 Cas d'Usage des Bind Mounts

#### Use Case 1 : Développement avec Hot Reload

```yaml
version: '3.8'

services:
  frontend:
    image: node:18
    working_dir: /app
    command: npm run dev
    volumes:
      # Code source en bind mount
      - ./frontend:/app
    ports:
      - "3000:3000"
```

**Workflow :**
```
1. Vous modifiez ./frontend/src/App.js sur votre PC
2. Le changement est INSTANTANÉMENT visible dans le conteneur
3. Le serveur de dev redémarre automatiquement
4. Le navigateur se rafraîchit
```

---

#### Use Case 2 : Fichiers de Configuration

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    volumes:
      # Données en volume nommé
      - mariadb_data:/var/lib/mysql
      # Config en bind mount (éditable facilement)
      - ./config/my.cnf:/etc/mysql/conf.d/custom.cnf:ro

volumes:
  mariadb_data:
```

**Avantages :**
- ✅ Vous éditez `./config/my.cnf` avec votre éditeur préféré
- ✅ Changements visibles immédiatement
- ✅ Versionnable dans Git

---

#### Use Case 3 : Logs Accessibles

```yaml
version: '3.8'

services:
  nginx:
    image: nginx
    volumes:
      - ./logs/nginx:/var/log/nginx
```

**Résultat :**
```
Vous pouvez lire les logs directement :
./logs/nginx/access.log
./logs/nginx/error.log
```

---

### 4.4 Options des Bind Mounts

```yaml
volumes:
  # Lecture/Écriture (défaut)
  - ./data:/app/data

  # Lecture seule
  - ./config:/app/config:ro

  # Avec consistance (macOS/Windows)
  - ./src:/app/src:cached       # Performances optimisées
  - ./src:/app/src:delegated    # Encore plus rapide
  - ./src:/app/src:consistent   # Cohérence stricte
```

**Explications :**
- `ro` : read-only (le conteneur ne peut pas modifier)
- `cached` : Optimise les lectures (macOS/Windows)
- `delegated` : Optimise les écritures (macOS/Windows)

---

### 4.5 Problèmes Courants des Bind Mounts

#### Problème 1 : Permissions (Linux)

**Symptôme :**
```
Permission denied: /app/data/file.txt
```

**Cause :** L'utilisateur dans le conteneur n'a pas les droits sur le dossier hôte.

**Solution 1 : Changer les permissions du dossier**
```bash
# Donner tous les droits (développement uniquement)
chmod -R 777 ./data

# Ou donner au même UID que le conteneur
chown -R 1000:1000 ./data
```

**Solution 2 : Lancer le conteneur avec votre UID**
```yaml
services:
  app:
    user: "${UID}:${GID}"
    volumes:
      - ./data:/app/data
```

```bash
# Définir les variables
export UID=$(id -u)
export GID=$(id -g)
docker-compose up -d
```

---

#### Problème 2 : Chemins Windows

**❌ Mauvais (ne fonctionne pas sur Linux) :**
```yaml
volumes:
  - C:\Users\Nom\projet:/app
```

**✅ Bon (portable) :**
```yaml
volumes:
  - ./projet:/app  # Chemin relatif
```

---

#### Problème 3 : Dossier Vide

**Symptôme :** Le dossier dans le conteneur est vide après le bind mount.

**Cause :** Le bind mount "écrase" le contenu du conteneur.

**Solution : Copier d'abord le contenu**
```dockerfile
# Dans le Dockerfile
COPY ./default_files /app/files

# Puis bind mount par-dessus si besoin
```

---

## ⚖️ 5. Comparaison et Choix

### 5.1 Tableau Décisionnel

| Vous voulez... | Solution | Exemple |
|----------------|----------|---------|
| 🗄️ Sauvegarder des données de BDD | Volume nommé | `mariadb_data:/var/lib/mysql` |
| 💻 Développer avec hot reload | Bind mount | `./src:/app/src` |
| ⚙️ Éditer facilement un fichier de config | Bind mount | `./my.cnf:/etc/mysql/conf.d/custom.cnf` |
| 📊 Garder des logs accessibles | Bind mount | `./logs:/var/log/nginx` |
| 🚀 Déployer en production | Volume nommé | Toujours |
| 🔐 Stocker des secrets temporaires | tmpfs | (en mémoire) |

---

### 5.2 Comparaison Détaillée

#### Volumes Nommés

**Quand utiliser :**
- ✅ Données de bases de données
- ✅ Données d'applications (uploads, etc.)
- ✅ Tout ce qui doit être sauvegardé
- ✅ Production

**Avantages :**
- Gérés par Docker
- Portables (Windows/Linux/macOS)
- Performances optimales
- Backups facilités
- Pas de problèmes de permissions

**Inconvénients :**
- Emplacement caché (pas facile d'accéder directement)
- Édition moins intuitive

---

#### Bind Mounts

**Quand utiliser :**
- ✅ Développement local
- ✅ Fichiers de configuration
- ✅ Code source
- ✅ Logs
- ✅ Tout ce que vous voulez éditer facilement

**Avantages :**
- Accès direct aux fichiers
- Édition avec vos outils habituels
- Hot reload en développement
- Facilement versionnable (Git)

**Inconvénients :**
- Problèmes de permissions (Linux)
- Chemins dépendent de l'OS
- Moins performant (surtout Windows/macOS)

---

### 5.3 Configuration Type pour un Projet

**docker-compose.yml idéal :**

```yaml
version: '3.8'

services:
  # Base de données
  mariadb:
    image: mariadb:10.11
    volumes:
      # Données : VOLUME NOMMÉ
      - mariadb_data:/var/lib/mysql
      # Config : BIND MOUNT
      - ./config/mariadb/my.cnf:/etc/mysql/conf.d/custom.cnf:ro

  # Application
  app:
    image: node:18
    volumes:
      # Code source (dev) : BIND MOUNT
      - ./app:/usr/src/app
      # node_modules : VOLUME ANONYME (optimisation)
      - /usr/src/app/node_modules
      # Uploads : VOLUME NOMMÉ
      - app_uploads:/usr/src/app/uploads

  # Nginx
  nginx:
    image: nginx
    volumes:
      # Config : BIND MOUNT
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      # Logs : BIND MOUNT (accessibles)
      - ./logs/nginx:/var/log/nginx

volumes:
  mariadb_data:
  app_uploads:
```

**Résumé :**
- 📦 Données critiques → Volume nommé
- 📝 Fichiers éditables → Bind mount
- 🚀 Meilleur des deux mondes

---

## 💿 6. Sauvegarde et Restauration

### 6.1 Pourquoi Sauvegarder ?

```
Scénarios catastrophes :
❌ Disque dur qui lâche
❌ Suppression accidentelle (docker volume rm)
❌ Corruption de données
❌ Besoin de migrer vers un autre serveur
```

**Solution : Backups réguliers** 📦

---

### 6.2 Sauvegarder un Volume Nommé

#### Méthode 1 : Via un Conteneur Temporaire (Simple)

```bash
# Sauvegarder le volume "mariadb_data" dans un fichier tar
docker run --rm \
  -v mariadb_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mariadb_backup_$(date +%Y%m%d).tar.gz -C /data .
```

**Explication :**
1. `--rm` : Supprime le conteneur après utilisation
2. `-v mariadb_data:/data` : Monte le volume à sauvegarder
3. `-v $(pwd):/backup` : Monte le dossier actuel pour sauvegarder le fichier
4. `tar czf` : Compresse le contenu dans un fichier .tar.gz
5. `$(date +%Y%m%d)` : Ajoute la date au nom du fichier

**Résultat :**
```
Fichier créé : mariadb_backup_20241029.tar.gz (dans le dossier actuel)
```

---

#### Méthode 2 : Script de Backup Automatisé

**Créer un fichier `backup-volume.sh` :**

```bash
#!/bin/bash

# Configuration
VOLUME_NAME=$1
BACKUP_DIR="./backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${VOLUME_NAME}_${TIMESTAMP}.tar.gz"

# Créer le dossier de backup s'il n'existe pas
mkdir -p "$BACKUP_DIR"

# Sauvegarder le volume
echo "🔄 Sauvegarde du volume $VOLUME_NAME..."
docker run --rm \
  -v "$VOLUME_NAME":/data \
  -v "$BACKUP_DIR":/backup \
  alpine tar czf "/backup/${VOLUME_NAME}_${TIMESTAMP}.tar.gz" -C /data .

if [ $? -eq 0 ]; then
    echo "✅ Backup créé : $BACKUP_FILE"
    ls -lh "$BACKUP_FILE"
else
    echo "❌ Erreur lors du backup"
    exit 1
fi
```

**Utilisation :**

```bash
# Rendre exécutable
chmod +x backup-volume.sh

# Sauvegarder un volume
./backup-volume.sh mariadb_data

# Résultat :
# ✅ Backup créé : ./backups/mariadb_data_20241029_143022.tar.gz
```

---

#### Méthode 3 : Backup Direct de la Base de Données

Pour les bases de données, utilisez leurs outils natifs :

**MariaDB/MySQL :**
```bash
# Backup SQL
docker exec mariadb_container mysqldump \
  -u root -p'root_password' \
  --all-databases \
  > backup_$(date +%Y%m%d).sql

# Backup compressé
docker exec mariadb_container mysqldump \
  -u root -p'root_password' \
  --all-databases \
  | gzip > backup_$(date +%Y%m%d).sql.gz
```

**PostgreSQL :**
```bash
# Backup
docker exec postgres_container pg_dumpall \
  -U postgres \
  > backup_$(date +%Y%m%d).sql
```

**MongoDB :**
```bash
# Backup
docker exec mongo_container mongodump \
  --out /backup

# Copier le backup
docker cp mongo_container:/backup ./mongo_backup_$(date +%Y%m%d)
```

---

### 6.3 Restaurer un Volume

#### Méthode 1 : Restaurer depuis un fichier tar

```bash
# 1. Créer un nouveau volume (ou utiliser un existant)
docker volume create mariadb_data_restored

# 2. Restaurer le contenu
docker run --rm \
  -v mariadb_data_restored:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/mariadb_backup_20241029.tar.gz"

# 3. Utiliser ce volume avec un conteneur
docker run -d \
  --name mariadb_restored \
  -v mariadb_data_restored:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mariadb:10.11
```

---

#### Méthode 2 : Script de Restauration

**Créer un fichier `restore-volume.sh` :**

```bash
#!/bin/bash

# Configuration
VOLUME_NAME=$1
BACKUP_FILE=$2

# Vérifications
if [ -z "$VOLUME_NAME" ] || [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <volume_name> <backup_file>"
    exit 1
fi

if [ ! -f "$BACKUP_FILE" ]; then
    echo "❌ Fichier de backup introuvable : $BACKUP_FILE"
    exit 1
fi

# Créer le volume s'il n'existe pas
docker volume create "$VOLUME_NAME"

# Restaurer
echo "🔄 Restauration du volume $VOLUME_NAME depuis $BACKUP_FILE..."
docker run --rm \
  -v "$VOLUME_NAME":/data \
  -v "$(pwd)":/backup \
  alpine sh -c "cd /data && tar xzf /backup/$(basename $BACKUP_FILE)"

if [ $? -eq 0 ]; then
    echo "✅ Volume restauré avec succès"
    docker volume inspect "$VOLUME_NAME"
else
    echo "❌ Erreur lors de la restauration"
    exit 1
fi
```

**Utilisation :**

```bash
chmod +x restore-volume.sh

# Restaurer
./restore-volume.sh mariadb_data_new ./backups/mariadb_data_20241029.tar.gz
```

---

#### Méthode 3 : Restaurer une Base de Données

**MariaDB/MySQL :**
```bash
# Depuis un fichier SQL
docker exec -i mariadb_container mysql \
  -u root -p'root_password' \
  < backup_20241029.sql

# Depuis un fichier compressé
gunzip < backup_20241029.sql.gz | \
docker exec -i mariadb_container mysql \
  -u root -p'root_password'
```

**PostgreSQL :**
```bash
# Restauration
docker exec -i postgres_container psql \
  -U postgres \
  < backup_20241029.sql
```

---

### 6.4 Cloner un Volume

Créer une copie exacte d'un volume :

```bash
# 1. Créer le nouveau volume
docker volume create mariadb_data_copy

# 2. Copier le contenu
docker run --rm \
  -v mariadb_data:/source:ro \
  -v mariadb_data_copy:/destination \
  alpine sh -c "cp -av /source/. /destination/"

echo "✅ Volume cloné : mariadb_data → mariadb_data_copy"
```

---

### 6.5 Migrer vers un Autre Serveur

**Sur le serveur source :**

```bash
# 1. Sauvegarder
./backup-volume.sh mariadb_data

# 2. Transférer le fichier
scp ./backups/mariadb_data_*.tar.gz user@nouveau-serveur:/tmp/
```

**Sur le nouveau serveur :**

```bash
# 3. Restaurer
./restore-volume.sh mariadb_data /tmp/mariadb_data_*.tar.gz

# 4. Lancer le conteneur
docker-compose up -d
```

---

### 6.6 Stratégie de Backup Recommandée

#### Développement

```bash
# Backup manuel avant actions risquées
./backup-volume.sh mariadb_data
```

#### Production

```bash
# Backups automatiques quotidiens (cron)
# Ajouter dans crontab :
0 2 * * * /chemin/backup-volume.sh mariadb_data
0 2 * * * /chemin/backup-volume.sh postgres_data

# Garder 7 derniers backups
find /backups -name "*.tar.gz" -mtime +7 -delete
```

**Checklist Production :**
- [ ] Backups quotidiens automatisés
- [ ] Stockage hors serveur (S3, NAS, autre serveur)
- [ ] Tests de restauration réguliers
- [ ] Rotation des backups (garder 7j, 4 semaines, 12 mois)
- [ ] Alertes en cas d'échec

---

## 🧹 7. Nettoyage des Volumes

### 7.1 Pourquoi Nettoyer ?

**Les volumes s'accumulent :**

```bash
# Lister les volumes
docker volume ls

# Résultat après quelques mois :
# DRIVER    VOLUME NAME
# local     old_project_db_1
# local     old_project_db_2
# local     test_mariadb_data
# local     abandoned_postgres_volume
# local     mariadb_data
# ...
# (des dizaines de volumes inutilisés)
```

**Conséquences :**
- 💾 Espace disque gaspillé
- 🐌 Ralentissement des commandes Docker
- 🤷 Confusion (quel volume appartient à quel projet ?)

---

### 7.2 Identifier les Volumes Inutilisés

```bash
# Lister tous les volumes
docker volume ls

# Voir l'utilisation disque
docker system df -v

# Résultat exemple :
# VOLUME NAME               SIZE
# mariadb_data              450MB   (en cours d'utilisation)
# old_project_db            2.1GB   (non utilisé)
# test_volume               0B      (vide, non utilisé)
```

---

### 7.3 Supprimer un Volume Spécifique

```bash
# Supprimer un volume
docker volume rm nom_du_volume

# Supprimer plusieurs volumes
docker volume rm volume1 volume2 volume3
```

**⚠️ ATTENTION :**
```bash
# ❌ NE PAS FAIRE si le volume est utilisé
docker volume rm mariadb_data
# Error: volume is in use

# ✅ D'abord arrêter le conteneur
docker-compose down
# Puis supprimer
docker volume rm mariadb_data
```

---

### 7.4 Supprimer Tous les Volumes Inutilisés (Prune)

```bash
# Supprimer tous les volumes non utilisés
docker volume prune

# Résultat :
# WARNING! This will remove all local volumes not used by at least one container.
# Are you sure you want to continue? [y/N] y
#
# Deleted Volumes:
# old_project_db
# test_volume
# abandoned_postgres_volume
#
# Total reclaimed space: 3.5GB
```

**Sans confirmation :**
```bash
docker volume prune -f
```

---

### 7.5 Nettoyer avec Docker Compose

**Supprimer volumes d'un projet spécifique :**

```bash
# Depuis le dossier du projet
# Arrêter et supprimer conteneurs + volumes
docker-compose down -v

# Le flag -v supprime les volumes déclarés dans le compose
```

**⚠️ ATTENTION :** Cela supprime TOUTES les données du projet !

---

### 7.6 Script de Nettoyage Intelligent

**Créer un fichier `clean-volumes.sh` :**

```bash
#!/bin/bash

echo "🔍 Analyse des volumes Docker..."

# Afficher l'utilisation actuelle
echo ""
echo "📊 Espace utilisé :"
docker system df -v | grep "Local Volumes"

echo ""
echo "📦 Volumes existants :"
docker volume ls

echo ""
read -p "Voulez-vous supprimer les volumes non utilisés ? [y/N] " -n 1 -r
echo ""

if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo "🧹 Nettoyage en cours..."
    docker volume prune -f

    echo ""
    echo "✅ Nettoyage terminé !"
    echo "📊 Espace libéré :"
    docker system df -v | grep "Local Volumes"
else
    echo "❌ Nettoyage annulé"
fi
```

**Utilisation :**

```bash
chmod +x clean-volumes.sh
./clean-volumes.sh
```

---

### 7.7 Nettoyage Complet du Système

```bash
# Nettoyer TOUT (conteneurs, images, volumes, réseaux)
docker system prune -a --volumes

# ⚠️ TRÈS DANGEREUX : Supprime TOUTES les données Docker
```

**Ce qui est supprimé :**
- ❌ Tous les conteneurs arrêtés
- ❌ Toutes les images non utilisées
- ❌ Tous les volumes non utilisés
- ❌ Tous les réseaux non utilisés

**Utilisez uniquement si :**
- ✅ Vous voulez repartir de zéro
- ✅ Vous avez sauvegardé vos données importantes
- ✅ Vous êtes sûr de ce que vous faites

---

### 7.8 Surveillance de l'Espace Disque

**Commande rapide :**

```bash
# Voir l'utilisation globale
docker system df

# Résultat :
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          10        5         2.5GB     1.2GB (48%)
# Containers      8         3         450MB     200MB (44%)
# Local Volumes   15        5         8.5GB     4.2GB (49%)
# Build Cache     0         0         0B        0B
```

**Vue détaillée :**

```bash
# Voir TOUS les détails
docker system df -v
```

---

### 7.9 Bonnes Pratiques de Nettoyage

**Routine recommandée :**

```bash
# Toutes les semaines
docker volume prune -f
docker image prune -f

# Tous les mois
docker system prune -f

# Avant actions importantes
./backup-volume.sh important_data
docker volume prune -f
```

**Automatisation (cron) :**

```bash
# Ajouter dans crontab (crontab -e)
# Nettoyage tous les dimanches à 3h du matin
0 3 * * 0 docker volume prune -f
0 3 * * 0 docker image prune -f
```

---

## 🎯 8. Cas d'Usage Pratiques

### 8.1 Projet de Développement Full-Stack

```yaml
version: '3.8'

services:
  # Base de données
  postgres:
    image: postgres:15
    volumes:
      # Données : VOLUME NOMMÉ
      - postgres_data:/var/lib/postgresql/data
      # Scripts d'init : BIND MOUNT
      - ./database/init:/docker-entrypoint-initdb.d:ro

  # Backend API
  backend:
    image: node:18
    working_dir: /app
    command: npm run dev
    volumes:
      # Code source : BIND MOUNT (hot reload)
      - ./backend:/app
      # node_modules : VOLUME ANONYME (performances)
      - /app/node_modules
      # Uploads : VOLUME NOMMÉ
      - backend_uploads:/app/uploads

  # Frontend
  frontend:
    image: node:18
    working_dir: /app
    command: npm run dev
    volumes:
      # Code source : BIND MOUNT
      - ./frontend:/app
      - /app/node_modules

volumes:
  postgres_data:
  backend_uploads:
```

**Avantages :**
- 🔄 Hot reload pour dev
- 💾 Données persistantes
- 🚀 node_modules optimisés

---

### 8.2 Environnement de Production

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    volumes:
      # Données : VOLUME NOMMÉ (backup facile)
      - mariadb_data:/var/lib/mysql
      # Config : BIND MOUNT (fichier éditable)
      - ./config/prod.cnf:/etc/mysql/conf.d/custom.cnf:ro
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}

  app:
    image: mon_app:latest
    volumes:
      # Uploads : VOLUME NOMMÉ
      - app_uploads:/var/www/uploads
      # Logs : BIND MOUNT (surveillance)
      - ./logs/app:/var/log/app
    environment:
      DB_HOST: mariadb

volumes:
  mariadb_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/data/mariadb  # SSD dédié

  app_uploads:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/storage/uploads  # Disque de stockage
```

---

### 8.3 Tests et CI/CD

```yaml
version: '3.8'

services:
  test_db:
    image: mariadb:10.11
    # tmpfs : Données en mémoire (rapide, éphémère)
    tmpfs:
      - /var/lib/mysql:size=500M
    environment:
      MYSQL_ROOT_PASSWORD: test
      MYSQL_DATABASE: test_db

  test_app:
    image: mon_app:test
    depends_on:
      - test_db
    # Pas de volumes : Tests isolés
```

**Avantages :**
- ⚡ Très rapide (RAM)
- 🧹 Nettoyage automatique
- 🔒 Isolation complète

---

### 8.4 Migration de Données

**Scénario :** Migrer de MariaDB vers PostgreSQL

```bash
# 1. Backup MariaDB
docker exec mariadb mysqldump \
  -u root -p'password' \
  --all-databases > mariadb_backup.sql

# 2. Convertir (pgloader ou manuellement)
# ...

# 3. Créer volume PostgreSQL
docker volume create postgres_data

# 4. Import dans PostgreSQL
docker run -d --name postgres_temp \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15

docker exec -i postgres_temp psql -U postgres < postgres_import.sql

# 5. Utiliser le nouveau volume
docker-compose up -d
```

---

## 📊 Tableaux de Référence

### Commandes Volumes Essentielles

| Commande | Description | Exemple |
|----------|-------------|---------|
| `docker volume create` | Créer un volume | `docker volume create mon_volume` |
| `docker volume ls` | Lister les volumes | `docker volume ls` |
| `docker volume inspect` | Détails d'un volume | `docker volume inspect mon_volume` |
| `docker volume rm` | Supprimer un volume | `docker volume rm mon_volume` |
| `docker volume prune` | Supprimer volumes inutilisés | `docker volume prune -f` |

---

### Syntaxe des Volumes dans docker-compose.yml

| Format | Type | Exemple | Use Case |
|--------|------|---------|----------|
| `nom:/chemin` | Volume nommé | `db_data:/var/lib/mysql` | Données de BDD |
| `./dossier:/chemin` | Bind mount relatif | `./config:/etc/app` | Config |
| `/abs/path:/chemin` | Bind mount absolu | `/data:/mnt/data` | Données fixes |
| `/chemin` | Volume anonyme | `/app/node_modules` | Optimisation |

---

### Options de Montage

| Option | Description | Exemple |
|--------|-------------|---------|
| `ro` | Read-only | `./config:/etc:ro` |
| `rw` | Read-write (défaut) | `./data:/app:rw` |
| `z` | SELinux label | `./data:/app:z` |
| `Z` | SELinux private | `./data:/app:Z` |
| `cached` | Cache optimisé | `./src:/app:cached` |
| `delegated` | Écriture optimisée | `./src:/app:delegated` |

---

## 🎓 Résumé des Concepts Clés

### Ce qu'il faut retenir

| Concept | Points Clés |
|---------|-------------|
| **Volumes Nommés** | Gérés par Docker, portables, performances optimales |
| **Bind Mounts** | Accès direct, édition facile, parfait pour dev |
| **tmpfs** | En mémoire, rapide, éphémère |
| **Backup** | Scripts automatisés, rotation, tests réguliers |
| **Nettoyage** | `docker volume prune` régulièrement |
| **Production** | Toujours volumes nommés + backups |

---

### Workflow Recommandé

```
Développement:
└─> Bind mounts pour code + volumes nommés pour données

Production:
└─> Volumes nommés pour tout + backups automatisés

Tests:
└─> tmpfs ou volumes anonymes (éphémères)
```

---

## 💡 Conseils Finaux

### Bonnes Pratiques

✅ **Toujours utiliser des volumes** pour les données importantes
✅ **Sauvegarder régulièrement** (automatisez !)
✅ **Nommer clairement** vos volumes (`projet_service_data`)
✅ **Documenter** quelle donnée va où
✅ **Tester vos restaurations** régulièrement
✅ **Nettoyer** les volumes inutilisés

❌ **Ne jamais supprimer** un volume sans backup
❌ **Ne pas mélanger** bind mounts et volumes nommés pour les mêmes données
❌ **Ne pas stocker** de données en dur dans les conteneurs
❌ **Ne pas oublier** le flag `-v` avec `docker-compose down`

---

### Checklist Avant Suppression

Avant de faire `docker volume rm` ou `docker-compose down -v` :

- [ ] Ai-je un backup récent ?
- [ ] Ce volume contient-il des données importantes ?
- [ ] Puis-je recréer ces données facilement ?
- [ ] Ai-je vérifié le bon volume (`docker volume inspect`) ?
- [ ] Suis-je sûr à 100% ?

**En cas de doute → BACKUP !**

---

## 🚀 Pour Aller Plus Loin

### Documentation Officielle

- 📖 [Docker Volumes](https://docs.docker.com/storage/volumes/)
- 📖 [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- 📖 [Manage Data](https://docs.docker.com/storage/)

### Annexes Connexes

- **[Annexe A - Commandes](A-reference-commandes.md)** - Toutes les commandes volumes
- **[Annexe E - Dépannage](E-depannage.md)** - Problèmes de volumes
- **[Cas Pratique 05](../cas-pratiques/05-migration-donnees.md)** - Migration de données

### Sujets Avancés

- Volumes avec drivers personnalisés (NFS, S3)
- Plugins de backup automatiques (Velero, Duplicati)
- Réplication de volumes entre serveurs
- Chiffrement de volumes

---

## 📝 Template de Configuration

```yaml
version: '3.8'

services:
  # Service avec volumes optimisés
  app:
    image: mon_app
    volumes:
      # Données persistantes : VOLUME NOMMÉ
      - app_data:/var/app/data

      # Configuration : BIND MOUNT (lecture seule)
      - ./config:/etc/app:ro

      # Code source (dev uniquement) : BIND MOUNT
      - ./src:/app/src

      # Optimisation (évite les conflits) : VOLUME ANONYME
      - /app/node_modules

      # Logs accessibles : BIND MOUNT
      - ./logs:/var/log/app

volumes:
  app_data:
    driver: local
    # Options de driver (optionnel)
    driver_opts:
      type: none
      o: bind
      device: /mnt/data/app
```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Sauvegardez souvent, dormez tranquille ! 😴💾*
