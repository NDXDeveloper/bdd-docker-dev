# 🐳 SQLite avec Volume Persistant

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans la fiche précédente, nous avons vu comment utiliser SQLite dans un conteneur Docker de manière basique. Le problème ? **Les données disparaissent dès que le conteneur est supprimé** ! 😱

Dans cette fiche, nous allons apprendre à **persister vos données SQLite** grâce aux volumes Docker. Vos fichiers de base de données seront stockés sur votre machine et survivront à l'arrêt ou à la suppression des conteneurs.

### 🎯 Ce que vous allez apprendre

- ✅ Comprendre les volumes Docker
- ✅ Créer un volume persistant pour SQLite
- ✅ Gérer vos bases de données de manière durable
- ✅ Sauvegarder et restaurer vos données
- ✅ Partager des bases SQLite entre conteneurs

---

## 🧠 Comprendre les Volumes Docker

### Qu'est-ce qu'un volume ?

Un **volume Docker** est un espace de stockage qui existe **en dehors du conteneur**. Pensez-y comme à un disque dur externe que vous pouvez brancher et débrancher de différents ordinateurs (conteneurs).

### Les deux types de volumes

#### 1. **Bind Mount** (Montage direct)
Relie **directement** un dossier de votre PC à un dossier du conteneur.

```
Votre PC                    Conteneur Docker
┌─────────────────┐        ┌──────────────┐
│ C:\mes-donnees\ │ <----> │ /data        │
└─────────────────┘        └──────────────┘
```

**Avantages :**
- ✅ Vous voyez et modifiez les fichiers directement depuis votre PC
- ✅ Facile à sauvegarder (copier le dossier)
- ✅ Peut être partagé avec d'autres applications

**Inconvénients :**
- ⚠️ Problèmes de permissions possibles (Linux/macOS)
- ⚠️ Différences de chemin selon l'OS (Windows vs Linux)

#### 2. **Named Volume** (Volume nommé)
Un espace géré **automatiquement** par Docker.

```
Docker gère                 Conteneur Docker
┌─────────────────┐        ┌──────────────┐
│ Volume "sqlite" │ <----> │ /data        │
└─────────────────┘        └──────────────┘
```

**Avantages :**
- ✅ Géré par Docker (pas de soucis de permissions)
- ✅ Portable entre différents OS
- ✅ Peut être partagé entre conteneurs

**Inconvénients :**
- ⚠️ Moins intuitif (fichiers cachés par Docker)
- ⚠️ Nécessite des commandes Docker pour accéder aux fichiers

**🎓 Pour les débutants :** Utilisez les **Bind Mounts** (méthode 1) car c'est plus simple à comprendre et à gérer.

---

## 🎯 Méthode 1 : Bind Mount (Montage Direct)

C'est la méthode **la plus simple** pour débuter. Vos fichiers seront directement visibles sur votre PC.

### A. Créer le dossier de données

Choisissez un emplacement sur votre PC pour stocker vos bases de données.

```bash
# Windows (PowerShell)
New-Item -ItemType Directory -Path C:\sqlite-data

# Windows (CMD)
mkdir C:\sqlite-data

# macOS / Linux
mkdir ~/sqlite-data
cd ~/sqlite-data
```

### B. Lancer le conteneur avec le volume

```bash
# Windows (PowerShell ou CMD)
docker run -d --name sqlite_persistant -v C:\sqlite-data:/data alpine tail -f /dev/null

# macOS / Linux
docker run -d --name sqlite_persistant -v ~/sqlite-data:/data alpine tail -f /dev/null
```

**Décortiquons cette commande :**

| Élément | Signification |
|---------|--------------|
| `docker run` | Lance un nouveau conteneur |
| `-d` | Mode détaché (tourne en arrière-plan) |
| `--name sqlite_persistant` | Nom du conteneur pour l'identifier facilement |
| `-v C:\sqlite-data:/data` | **Volume** : lie `C:\sqlite-data` (PC) à `/data` (conteneur) |
| `alpine` | Image Linux légère |
| `tail -f /dev/null` | Commande qui garde le conteneur actif |

### C. Installer SQLite dans le conteneur

```bash
# Entrer dans le conteneur
docker exec -it sqlite_persistant sh

# Une fois à l'intérieur, installer SQLite
apk update
apk add sqlite

# Vérifier l'installation
sqlite3 --version
```

### D. Créer votre première base persistante

```bash
# Se placer dans le dossier persistant
cd /data

# Créer une base de données
sqlite3 bibliotheque.db
```

Vous êtes maintenant dans le shell SQLite. Créons une vraie base de données :

```sql
-- Créer une table de livres
CREATE TABLE livres (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    titre TEXT NOT NULL,
    auteur TEXT NOT NULL,
    annee INTEGER,
    lu BOOLEAN DEFAULT 0
);

-- Insérer quelques livres
INSERT INTO livres (titre, auteur, annee, lu) VALUES
    ('1984', 'George Orwell', 1949, 1),
    ('Le Petit Prince', 'Antoine de Saint-Exupéry', 1943, 1),
    ('Dune', 'Frank Herbert', 1965, 0);

-- Vérifier les données
SELECT * FROM livres;

-- Afficher de manière plus lisible
.mode column
.headers on
SELECT * FROM livres;

-- Quitter SQLite
.quit
```

### E. Vérifier la persistance

**1. Depuis votre PC :**

Ouvrez votre explorateur de fichiers :
- Windows : Allez dans `C:\sqlite-data`
- macOS : Ouvrez le Finder et allez dans votre dossier personnel, puis `sqlite-data`
- Linux : `ls ~/sqlite-data`

Vous devriez voir le fichier `bibliotheque.db` ! 🎉

**2. Tester la persistance :**

```bash
# Quitter le conteneur
exit

# Supprimer le conteneur
docker rm -f sqlite_persistant

# Relancer un NOUVEAU conteneur avec le MÊME volume
docker run -d --name sqlite_nouveau -v C:\sqlite-data:/data alpine tail -f /dev/null

# Entrer dedans
docker exec -it sqlite_nouveau sh

# Installer SQLite
apk add sqlite

# Accéder à votre base
cd /data
sqlite3 bibliotheque.db

# Vos données sont toujours là !
SELECT * FROM livres;
```

**Magie !** 🪄 Même après avoir supprimé le conteneur, vos données sont sauvegardées.

---

## 🎯 Méthode 2 : Named Volume (Volume Nommé)

Cette méthode est **gérée automatiquement par Docker**. Plus robuste, mais moins intuitive pour les débutants.

### A. Créer un volume nommé

```bash
# Créer un volume appelé "sqlite_data"
docker volume create sqlite_data

# Lister tous les volumes
docker volume ls

# Inspecter le volume (voir où Docker le stocke)
docker volume inspect sqlite_data
```

### B. Lancer le conteneur avec le volume nommé

```bash
# Utiliser le volume nommé
docker run -d --name sqlite_volume -v sqlite_data:/data alpine tail -f /dev/null
```

**Différence avec la méthode 1 :**
- Au lieu de `-v C:\sqlite-data:/data` (chemin complet)
- On utilise `-v sqlite_data:/data` (nom du volume)

### C. Utiliser le conteneur normalement

```bash
# Entrer dans le conteneur
docker exec -it sqlite_volume sh

# Installer SQLite
apk add sqlite

# Créer une base
cd /data
sqlite3 notes.db
```

```sql
-- Créer une table de notes
CREATE TABLE notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    titre TEXT NOT NULL,
    contenu TEXT,
    date_creation DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Ajouter des notes
INSERT INTO notes (titre, contenu) VALUES
    ('Courses', 'Acheter du pain, des œufs et du lait'),
    ('Idée projet', 'Créer une application de gestion de tâches'),
    ('Rendez-vous', 'Dentiste le 15 à 14h');

-- Afficher
SELECT * FROM notes;

-- Quitter
.quit
exit
```

### D. Accéder aux fichiers du volume nommé

C'est moins direct qu'avec un Bind Mount, mais possible :

```bash
# Copier la base depuis le volume vers votre PC
docker cp sqlite_volume:/data/notes.db ./notes_backup.db

# Modifier localement puis remettre dans le volume
docker cp ./notes_backup.db sqlite_volume:/data/notes.db
```

---

## 🎯 Méthode 3 : Docker Compose avec Volume Persistant

C'est la méthode **professionnelle** et la plus pratique pour un usage quotidien.

### A. Créer la structure

```
mon-projet-sqlite/
├── docker-compose.yml
└── data/                # Sera créé automatiquement
```

### B. Fichier docker-compose.yml (Avec Bind Mount)

```yaml
version: '3.8'

services:
  sqlite:
    image: alpine:latest
    container_name: sqlite_compose
    command: tail -f /dev/null
    volumes:
      # Bind mount : dossier local -> dossier conteneur
      - ./data:/data
    working_dir: /data
    restart: unless-stopped
```

**Explications :**
- `./data:/data` : Crée un dossier `data` à côté du `docker-compose.yml`
- `working_dir: /data` : Ouvre directement dans `/data` à l'entrée
- `restart: unless-stopped` : Redémarre automatiquement (sauf si arrêt manuel)

### C. Fichier docker-compose.yml (Avec Named Volume)

```yaml
version: '3.8'

services:
  sqlite:
    image: alpine:latest
    container_name: sqlite_compose
    command: tail -f /dev/null
    volumes:
      # Volume nommé géré par Docker
      - sqlite_data:/data
    working_dir: /data
    restart: unless-stopped

# Déclaration du volume
volumes:
  sqlite_data:
    # Docker gère automatiquement son emplacement
```

### D. Utilisation

```bash
# Démarrer
docker-compose up -d

# Entrer dans le conteneur
docker exec -it sqlite_compose sh

# Installer SQLite (une seule fois)
apk add sqlite

# Créer/utiliser vos bases
sqlite3 ma_base.db
```

### E. Gestion quotidienne

```bash
# Voir l'état
docker-compose ps

# Voir les logs
docker-compose logs -f

# Arrêter (sans supprimer les données)
docker-compose stop

# Redémarrer
docker-compose start

# Supprimer le conteneur (les données restent)
docker-compose down

# Supprimer TOUT (conteneur + données)
docker-compose down -v
```

---

## 📊 Gérer Plusieurs Bases de Données

Vous pouvez organiser plusieurs bases dans le même volume.

### Exemple de structure

```
data/
├── bibliotheque.db      # Base de livres
├── notes.db             # Base de notes
├── projets.db           # Base de projets
└── backups/
    ├── bibliotheque_backup_20251030.db
    └── notes_backup_20251030.db
```

### Script pour créer plusieurs bases

```bash
# Entrer dans le conteneur
docker exec -it sqlite_compose sh

# Créer plusieurs bases
cd /data

# Base 1 : Gestion de contacts
sqlite3 contacts.db <<EOF
CREATE TABLE contacts (
    id INTEGER PRIMARY KEY,
    nom TEXT,
    telephone TEXT,
    email TEXT
);
INSERT INTO contacts VALUES (1, 'Alice Dupont', '0612345678', 'alice@example.com');
.quit
EOF

# Base 2 : Gestion de tâches
sqlite3 taches.db <<EOF
CREATE TABLE taches (
    id INTEGER PRIMARY KEY,
    description TEXT,
    terminee BOOLEAN DEFAULT 0
);
INSERT INTO taches VALUES (1, 'Apprendre Docker', 1);
INSERT INTO taches VALUES (2, 'Maîtriser SQLite', 0);
.quit
EOF

# Lister toutes vos bases
ls -lh *.db
```

---

## 💾 Sauvegarde et Restauration

### Sauvegarder une base SQLite

**Méthode 1 : Copie simple du fichier**

```bash
# Depuis votre PC (si Bind Mount)
# Windows
copy C:\sqlite-data\bibliotheque.db C:\sqlite-data\backups\bibliotheque_backup.db

# Linux/macOS
cp ~/sqlite-data/bibliotheque.db ~/sqlite-data/backups/bibliotheque_backup.db
```

**Méthode 2 : Utiliser la commande SQLite `.backup`**

```bash
# Depuis le conteneur
docker exec -it sqlite_compose sh

cd /data
sqlite3 bibliotheque.db ".backup backups/bibliotheque_$(date +%Y%m%d).db"
```

**Méthode 3 : Export SQL**

```bash
# Exporter en SQL (lisible par les humains)
docker exec -it sqlite_compose sh

sqlite3 bibliotheque.db ".output bibliotheque.sql"
sqlite3 bibliotheque.db ".dump"
```

### Restaurer une base SQLite

**Depuis une copie :**

```bash
# Remplacer la base actuelle par le backup
cp backups/bibliotheque_backup.db bibliotheque.db
```

**Depuis un fichier SQL :**

```bash
# Créer une nouvelle base et importer
sqlite3 nouvelle_base.db < bibliotheque.sql
```

### Script de backup automatique

Créez un fichier `backup.sh` :

```bash
#!/bin/bash
# backup.sh - Sauvegarde automatique de toutes les bases SQLite

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="backups"

# Créer le dossier de backup s'il n'existe pas
docker exec sqlite_compose mkdir -p /data/$BACKUP_DIR

# Sauvegarder chaque base .db
for db in $(docker exec sqlite_compose ls /data/*.db); do
    filename=$(basename $db)
    docker exec sqlite_compose sqlite3 /data/$filename ".backup /data/$BACKUP_DIR/${filename%.db}_$DATE.db"
    echo "✅ Sauvegarde de $filename terminée"
done

echo "🎉 Toutes les bases ont été sauvegardées !"
```

Utilisez-le :

```bash
# Rendre exécutable (Linux/macOS)
chmod +x backup.sh

# Lancer
./backup.sh
```

---

## 🔄 Partager des Données Entre Conteneurs

Vous pouvez utiliser le **même volume** dans plusieurs conteneurs.

### Exemple : SQLite + Application Web

```yaml
version: '3.8'

services:
  # Conteneur SQLite
  sqlite:
    image: alpine:latest
    container_name: sqlite_db
    command: tail -f /dev/null
    volumes:
      - shared_data:/data

  # Conteneur d'une application (exemple : Python)
  app:
    image: python:3.11-alpine
    container_name: python_app
    command: tail -f /dev/null
    volumes:
      # Partage le MÊME volume
      - shared_data:/app/data
    depends_on:
      - sqlite

volumes:
  # Volume partagé entre les deux conteneurs
  shared_data:
```

**Utilisation :**

```bash
# Démarrer les deux conteneurs
docker-compose up -d

# Créer une base dans le conteneur SQLite
docker exec -it sqlite_db sh
apk add sqlite
cd /data
sqlite3 partage.db
# Créer des données...
exit

# Accéder à la MÊME base depuis l'app Python
docker exec -it python_app sh
cd /app/data
ls -l partage.db  # Le fichier est là !
```

---

## 🧹 Gestion et Nettoyage des Volumes

### Lister les volumes

```bash
# Tous les volumes
docker volume ls

# Trouver un volume spécifique
docker volume ls | grep sqlite
```

### Inspecter un volume

```bash
# Voir les détails (où il est stocké, qui l'utilise...)
docker volume inspect sqlite_data
```

### Supprimer un volume

```bash
# Supprimer un volume (ATTENTION : irréversible)
docker volume rm sqlite_data

# Supprimer tous les volumes non utilisés
docker volume prune

# Avec confirmation automatique
docker volume prune -f
```

### Libérer de l'espace

```bash
# Voir l'espace utilisé
docker system df

# Nettoyer tout ce qui n'est pas utilisé
docker system prune -a --volumes

# Attention : ceci supprime TOUT ce qui n'est pas actif !
```

---

## 📏 Bonnes Pratiques

### ✅ À FAIRE

1. **Nommer explicitement vos volumes**
   ```yaml
   volumes:
     - mes_donnees_sqlite:/data  # Bon
   ```

2. **Sauvegarder régulièrement**
   ```bash
   # Automatiser avec un cron job (Linux) ou Planificateur de tâches (Windows)
   0 2 * * * /home/user/backup-sqlite.sh
   ```

3. **Utiliser .gitignore pour les données**
   ```gitignore
   # .gitignore
   data/*.db
   data/backups/
   ```

4. **Documenter l'emplacement des volumes**
   ```yaml
   # docker-compose.yml
   volumes:
     # Volume pour les bases SQLite de développement
     # Emplacement : ./data (bind mount)
     - ./data:/data
   ```

### ❌ À ÉVITER

1. **Ne pas spécifier de volume** (données perdues !)
2. **Utiliser des chemins relatifs complexes** (`../../data`)
3. **Mélanger données et code source** dans le même dossier
4. **Oublier de sauvegarder** avant `docker-compose down -v`

---

## 🔍 Dépannage

### Problème : "Permission denied" (Linux/macOS)

```bash
# Vérifier les permissions du dossier
ls -la ~/sqlite-data

# Donner les bonnes permissions
chmod -R 755 ~/sqlite-data

# Ou changer le propriétaire
sudo chown -R $USER:$USER ~/sqlite-data
```

### Problème : Le fichier .db n'apparaît pas

```bash
# Vérifier que le volume est bien monté
docker inspect sqlite_compose | grep Mounts -A 10

# Vérifier depuis le conteneur
docker exec -it sqlite_compose ls -la /data
```

### Problème : "Database is locked"

```bash
# Un autre processus utilise la base
# Fermer toutes les connexions SQLite actives
# Puis redémarrer le conteneur
docker restart sqlite_compose
```

### Problème : Le volume est plein

```bash
# Vérifier l'espace disque
df -h

# Vérifier la taille des bases
docker exec sqlite_compose du -sh /data/*

# Nettoyer les anciens backups
docker exec sqlite_compose rm /data/backups/*_old.db
```

---

## 📊 Comparaison des Méthodes

| Critère | Bind Mount | Named Volume | Docker Compose |
|---------|------------|--------------|----------------|
| **Simplicité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Accès fichiers** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ (bind) |
| **Portabilité** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Permissions** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Partage équipe** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Débutants** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

**🎯 Recommandation :**
- **Débutants** : Bind Mount avec Docker Compose
- **Avancés** : Named Volume avec Docker Compose
- **Production** : Named Volume avec backups automatisés

---

## 💡 Cas d'Usage Réels

### 1. Application Mobile (développement)

```yaml
# Structure pour une app mobile
volumes:
  - ./databases:/app/databases  # Bases SQLite de l'app
  - ./backups:/app/backups      # Sauvegardes
```

### 2. Tests Automatisés

```yaml
# Base de test qui se reset à chaque lancement
services:
  sqlite_test:
    image: alpine
    volumes:
      - ./test-data:/data
    # Script qui reset la base au démarrage
    command: sh -c "rm -f /data/test.db && tail -f /dev/null"
```

### 3. Développement d'API

```yaml
# API Node.js + SQLite
services:
  api:
    image: node:18-alpine
    volumes:
      - ./api:/app
      - shared_db:/app/db  # Base partagée

  db_admin:
    image: alpine
    volumes:
      - shared_db:/data
```

---

## 🎓 Pour Aller Plus Loin

- 📖 [Fiche suivante : Outils graphiques pour SQLite](03-outils-graphiques.md)
- 🔧 [Annexe C : Gestion des volumes Docker](../annexes/C-gestion-volumes.md)
- 💾 [Annexe : Sauvegardes et restauration](../annexes/C-gestion-volumes.md#sauvegardes)
- 📚 [Documentation Docker Volumes](https://docs.docker.com/storage/volumes/)

---

## ✅ Points Clés à Retenir

1. 📁 **Bind Mount** = Dossier de votre PC visible dans le conteneur
2. 🗄️ **Named Volume** = Stockage géré automatiquement par Docker
3. 🔄 Les données **persistent** après suppression du conteneur
4. 💾 **Sauvegardez régulièrement** vos bases importantes
5. 🧹 **Nettoyez** les vieux volumes pour libérer de l'espace
6. 🐳 **Docker Compose** simplifie la gestion des volumes

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
