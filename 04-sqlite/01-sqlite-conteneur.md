# 🐳 SQLite en Conteneur Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

SQLite est une base de données **relationnelle** et **embarquée** : elle ne nécessite pas de serveur séparé et stocke toutes ses données dans un simple fichier. C'est la base de données parfaite pour :

- 🧪 Les tests et prototypes rapides
- 📱 Les applications mobiles
- 🔧 Les petits projets personnels
- 📊 Le stockage de configurations ou de logs

Contrairement à MariaDB, PostgreSQL ou MongoDB qui fonctionnent comme des **services permanents**, SQLite est **juste un fichier** que vos applications lisent et écrivent.

### 🤔 Pourquoi utiliser SQLite avec Docker ?

Vous pourriez vous demander : "Si SQLite est juste un fichier, pourquoi utiliser Docker ?"

**Bonne question !** Voici les avantages :

1. **Outils graphiques** : Accéder à des interfaces comme `sqlite-web` sans installation
2. **Version précise** : Utiliser une version spécifique de SQLite sans la modifier sur votre système
3. **Isolation** : Tester des modifications sans risque
4. **Portabilité** : Partager facilement votre environnement avec votre équipe

---

## 🎯 Méthode 1 : SQLite avec l'image Alpine (Ligne de commande)

C'est la méthode la plus **légère et simple** pour travailler avec SQLite en ligne de commande.

### A. Lancement rapide

```bash
# Lancer un conteneur Alpine avec SQLite installé
docker run -it --rm alpine sh
```

**Explications :**
- `docker run` : Lance un nouveau conteneur
- `-it` : Mode interactif (vous pouvez taper des commandes)
- `--rm` : Supprime automatiquement le conteneur à la sortie
- `alpine` : Une image Linux ultra-légère (seulement 5 MB !)
- `sh` : Lance le shell (terminal)

### B. Installer SQLite dans le conteneur

Une fois à l'intérieur du conteneur :

```bash
# Mettre à jour la liste des paquets
apk update

# Installer SQLite
apk add sqlite

# Vérifier l'installation
sqlite3 --version
```

### C. Créer et utiliser une base de données

```bash
# Créer une nouvelle base de données (ou ouvrir une existante)
sqlite3 ma_base.db
```

Vous êtes maintenant dans le **shell SQLite**. Essayez ces commandes :

```sql
-- Créer une table simple
CREATE TABLE utilisateurs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nom TEXT NOT NULL,
    email TEXT UNIQUE
);

-- Insérer des données
INSERT INTO utilisateurs (nom, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO utilisateurs (nom, email) VALUES ('Bob', 'bob@example.com');

-- Afficher les données
SELECT * FROM utilisateurs;

-- Quitter SQLite
.quit
```

### D. Quitter le conteneur

```bash
# Quitter le shell Alpine
exit
```

⚠️ **Attention** : Avec cette méthode, vos données sont **perdues** à la sortie du conteneur (à cause de `--rm`). Pour les conserver, passez à la méthode 2.

---

## 🎯 Méthode 2 : SQLite avec Volume Persistant

Cette méthode conserve votre fichier de base de données **sur votre machine hôte**, même après la fermeture du conteneur.

### A. Créer un dossier pour vos données

```bash
# Sur Windows (PowerShell) :
mkdir C:\sqlite-data

# Sur macOS/Linux :
mkdir ~/sqlite-data
```

### B. Lancer le conteneur avec un volume

```bash
# Windows (PowerShell) :
docker run -it --rm -v C:\sqlite-data:/data alpine sh

# macOS/Linux :
docker run -it --rm -v ~/sqlite-data:/data alpine sh
```

**Explications du `-v` (volume) :**
- `-v C:\sqlite-data:/data` : Relie le dossier `C:\sqlite-data` de votre PC au dossier `/data` du conteneur
- Tout ce que vous créez dans `/data` sera **sauvegardé sur votre PC**

### C. Installer SQLite et créer une base

```bash
# Installer SQLite
apk update && apk add sqlite

# Se déplacer dans le dossier persistant
cd /data

# Créer votre base de données
sqlite3 ma_base_persistante.db
```

```sql
-- Exemple de création de table
CREATE TABLE produits (
    id INTEGER PRIMARY KEY,
    nom TEXT,
    prix REAL
);

INSERT INTO produits VALUES (1, 'Ordinateur', 899.99);
INSERT INTO produits VALUES (2, 'Souris', 19.99);

SELECT * FROM produits;

.quit
```

### D. Vérifier la persistance

1. **Quittez le conteneur** : `exit`
2. **Vérifiez votre dossier** :
   - Windows : Ouvrez `C:\sqlite-data`
   - macOS/Linux : `ls ~/sqlite-data`
3. Vous devriez voir `ma_base_persistante.db` ! 🎉

### E. Relancer pour accéder à vos données

```bash
# Relancer avec le même volume
docker run -it --rm -v ~/sqlite-data:/data alpine sh

# Installer SQLite à nouveau (car c'est un nouveau conteneur)
apk add sqlite

# Accéder à votre base existante
cd /data
sqlite3 ma_base_persistante.db

# Vos données sont toujours là !
SELECT * FROM produits;
```

---

## 🎯 Méthode 3 : SQLite avec Docker Compose (Recommandé)

Cette méthode est **la plus pratique** pour un usage régulier car elle permet de :
- ✅ Garder la configuration dans un fichier
- ✅ Lancer/arrêter facilement
- ✅ Partager la configuration avec votre équipe

### A. Créer les fichiers nécessaires

**1. Structure des dossiers :**

```
mon-projet-sqlite/
├── docker-compose.yml
└── data/                (sera créé automatiquement)
```

**2. Créer le fichier `docker-compose.yml` :**

```yaml
version: '3.8'

services:
  sqlite:
    image: alpine:latest
    container_name: sqlite_dev
    # Garde le conteneur actif
    command: tail -f /dev/null
    volumes:
      # Relie le dossier 'data' local au dossier '/data' du conteneur
      - ./data:/data
    working_dir: /data
```

**Explications :**
- `command: tail -f /dev/null` : Astuce pour garder le conteneur actif (sinon il s'arrête immédiatement)
- `working_dir: /data` : Définit `/data` comme dossier par défaut
- `./data:/data` : Les fichiers seront dans le dossier `data` à côté du `docker-compose.yml`

### B. Lancer l'environnement

```bash
# Démarrer le conteneur en arrière-plan
docker-compose up -d

# Vérifier qu'il tourne
docker-compose ps
```

### C. Accéder au conteneur

```bash
# Entrer dans le conteneur
docker exec -it sqlite_dev sh

# Installer SQLite (à faire une seule fois après le premier lancement)
apk add sqlite

# Créer/accéder à votre base
sqlite3 ma_base.db
```

### D. Gestion quotidienne

```bash
# Arrêter le conteneur (sans supprimer les données)
docker-compose stop

# Redémarrer
docker-compose start

# Voir les logs
docker-compose logs -f

# Supprimer complètement (ATTENTION : supprime aussi le dossier data)
docker-compose down
# Pour garder les données, ne supprimez PAS le dossier data/
```

---

## 📊 Commandes SQLite Utiles

Une fois dans le shell SQLite (`sqlite3 ma_base.db`), voici les commandes les plus pratiques :

### Commandes de gestion

```sql
-- Lister toutes les tables
.tables

-- Afficher la structure d'une table
.schema nom_table

-- Afficher tous les schémas (structure complète)
.schema

-- Changer le format d'affichage (plus lisible)
.mode column
.headers on

-- Sauvegarder la base dans un fichier SQL
.output backup.sql
.dump
.output stdout

-- Charger un fichier SQL
.read backup.sql

-- Quitter
.quit
```

### Commandes SQL courantes

```sql
-- Créer une table
CREATE TABLE notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    titre TEXT NOT NULL,
    contenu TEXT,
    date_creation DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Insérer des données
INSERT INTO notes (titre, contenu) VALUES ('Ma première note', 'Ceci est un test');

-- Lire les données
SELECT * FROM notes;

-- Mettre à jour
UPDATE notes SET contenu = 'Contenu modifié' WHERE id = 1;

-- Supprimer
DELETE FROM notes WHERE id = 1;

-- Supprimer une table
DROP TABLE notes;
```

---

## 🔍 Accéder à votre base depuis votre PC

Votre fichier `ma_base.db` est maintenant **sur votre PC** (dans le dossier `data/`). Vous pouvez l'ouvrir avec des outils graphiques comme :

- **DB Browser for SQLite** (gratuit, multi-plateforme)
- **DBeaver** (supporte aussi SQLite)
- **DataGrip** (JetBrains, payant)
- **Visual Studio Code** (avec extension SQLite)

Exemple avec DB Browser :
1. Téléchargez [DB Browser for SQLite](https://sqlitebrowser.org/)
2. Ouvrez le logiciel
3. Cliquez sur "Ouvrir une base de données"
4. Naviguez vers `mon-projet-sqlite/data/ma_base.db`
5. Explorez vos tables ! 🎉

---

## 🧹 Nettoyage Complet

### Supprimer le conteneur

```bash
# Avec docker-compose
docker-compose down

# Avec docker run
docker stop sqlite_dev
docker rm sqlite_dev
```

### Supprimer les données (ATTENTION : irréversible)

```bash
# Supprimer le dossier data
# Windows :
Remove-Item -Recurse -Force C:\sqlite-data

# macOS/Linux :
rm -rf ~/sqlite-data

# Ou si vous utilisez docker-compose :
rm -rf ./data
```

---

## ✅ Résumé des Méthodes

| Méthode | Avantages | Inconvénients | Idéal pour |
|---------|-----------|---------------|------------|
| **Alpine simple** | ⚡ Ultra rapide | ❌ Données perdues | Tests rapides |
| **Volume persistant** | ✅ Données sauvées | 🔧 Commande longue | Usage ponctuel |
| **Docker Compose** | ✅ Configuration réutilisable<br>✅ Facile à partager | 📝 Fichier à créer | Usage régulier |

---

## 💡 Conseils Pratiques

### 1. Backup automatique

Créez un script pour sauvegarder régulièrement :

```bash
#!/bin/bash
# backup.sh
docker exec sqlite_dev sqlite3 /data/ma_base.db ".backup /data/backup_$(date +%Y%m%d).db"
```

### 2. Alias pour accès rapide

Ajoutez à votre `~/.bashrc` ou `~/.zshrc` :

```bash
alias sqlite-dev="docker exec -it sqlite_dev sqlite3 /data/ma_base.db"
```

Puis utilisez simplement :

```bash
sqlite-dev
```

### 3. Vérifier la taille de la base

```bash
# Depuis votre PC
ls -lh data/ma_base.db

# Depuis le conteneur
du -h /data/ma_base.db
```

---

## 🎓 Pour Aller Plus Loin

- 📖 [Documentation officielle SQLite](https://sqlite.org/docs.html)
- 🎥 [Tutoriel SQLite en français](https://www.youtube.com/results?search_query=tutoriel+sqlite+français)
- 🔧 [Fiche suivante : SQLite avec volume persistant](02-sqlite-volume-persistant.md)
- 🖥️ [Fiche : Outils graphiques pour SQLite](03-outils-graphiques.md)

---

## ❓ Questions Fréquentes

**Q : Puis-je utiliser SQLite pour une application web en production ?**
R : SQLite convient pour des sites à **faible trafic** (< 100 000 requêtes/jour). Au-delà, préférez PostgreSQL ou MariaDB.

**Q : Comment transférer ma base SQLite vers PostgreSQL ?**
R : Utilisez des outils comme `pgloader` ou des scripts de migration. Consultez [ce guide](https://wiki.postgresql.org/wiki/Converting_from_other_Databases_to_PostgreSQL).

**Q : SQLite supporte-t-il les utilisateurs et permissions ?**
R : Non, SQLite n'a **pas de système d'utilisateurs**. Les permissions sont gérées au niveau du fichier (permissions du système d'exploitation).

**Q : Quelle est la taille maximale d'une base SQLite ?**
R : Théoriquement **281 TB**, mais en pratique, restez sous **1 GB** pour de bonnes performances.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
