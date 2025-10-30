# 🐳 Outils Graphiques pour SQLite

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Travailler avec SQLite en ligne de commande, c'est bien... mais utiliser une **interface graphique**, c'est encore mieux ! 🎨

Les outils graphiques (aussi appelés **GUI** - Graphical User Interface) vous permettent de :

- 👀 **Visualiser** vos tables et données de manière claire
- ✏️ **Éditer** les données en quelques clics
- 🔍 **Rechercher** rapidement dans vos bases
- 📊 **Analyser** la structure de vos tables
- 🚀 **Exécuter** des requêtes SQL avec autocomplétion
- 📈 **Exporter** vos données (CSV, JSON, Excel...)

Dans cette fiche, nous allons voir plusieurs outils graphiques que vous pouvez utiliser avec vos bases SQLite Docker, **sans rien installer** sur votre machine principale !

---

## 🎯 Les 3 Approches Possibles

### 1. Outils Installés sur Votre PC (Recommandé pour Débutants)

Vous accédez directement aux fichiers `.db` de votre dossier Docker.

**Avantages :**
- ✅ Simple et rapide
- ✅ Pas besoin de Docker pour consulter
- ✅ Interface native et performante

**Inconvénients :**
- ⚠️ Installation sur votre machine
- ⚠️ Un outil par système d'exploitation parfois

### 2. Outils Web dans Docker

Une interface web qui tourne dans un conteneur Docker.

**Avantages :**
- ✅ Rien à installer sur votre PC
- ✅ Accessible depuis un navigateur
- ✅ Peut être partagé en réseau

**Inconvénients :**
- ⚠️ Moins de fonctionnalités parfois
- ⚠️ Nécessite Docker en permanence

### 3. Outils Multi-Plateformes (DataGrip, DBeaver)

Des outils professionnels supportant plusieurs types de bases de données.

**Avantages :**
- ✅ Un seul outil pour toutes vos BDD
- ✅ Fonctionnalités avancées
- ✅ Professionnel

**Inconvénients :**
- ⚠️ Peut être complexe pour débuter
- ⚠️ Certains sont payants

---

## 🖥️ Méthode 1 : DB Browser for SQLite (Recommandé)

**DB Browser** est l'outil **gratuit et open-source** le plus populaire pour SQLite. Parfait pour les débutants !

### A. Installation

#### Windows

1. Téléchargez depuis [sqlitebrowser.org](https://sqlitebrowser.org/dl/)
2. Choisissez la version **Installer** (`.msi`)
3. Lancez l'installeur et suivez les étapes
4. Ouvrez **DB Browser for SQLite** depuis le menu Démarrer

#### macOS

```bash
# Avec Homebrew
brew install --cask db-browser-for-sqlite

# Ou téléchargez le .dmg depuis sqlitebrowser.org
```

#### Linux (Ubuntu/Debian)

```bash
# Via les dépôts officiels
sudo apt update
sudo apt install sqlitebrowser

# Ou via Snap
sudo snap install sqlitebrowser
```

### B. Connexion à votre Base Docker

Si vous utilisez un **bind mount** (méthode recommandée dans les fiches précédentes), c'est très simple :

1. **Ouvrez DB Browser for SQLite**
2. Cliquez sur **"Open Database"** (ou `Ctrl+O`)
3. Naviguez vers votre dossier Docker :
   - Windows : `C:\sqlite-data\`
   - macOS/Linux : `~/sqlite-data/`
4. Sélectionnez votre fichier `.db`
5. **C'est prêt !** 🎉

### C. Interface et Fonctionnalités

#### 📊 Onglet "Database Structure"

Visualisez la structure de votre base :

```
📁 ma_base.db
  📊 Tables
    ├── 📋 livres (3 colonnes, 15 lignes)
    ├── 📋 auteurs (2 colonnes, 8 lignes)
    └── 📋 emprunts (4 colonnes, 42 lignes)
  🔍 Indexes
  ⚡ Triggers
  👁️ Views
```

**Actions possibles :**
- 👆 Clic droit sur une table → **Modifier**, **Supprimer**, **Dupliquer**
- ➕ Bouton **"Create Table"** pour créer une nouvelle table visuellement
- 🔧 Modifier la structure d'une table en quelques clics

#### 📄 Onglet "Browse Data"

Afficher et éditer vos données comme dans Excel :

| id | titre | auteur | annee | lu |
|----|-------|--------|-------|-----|
| 1 | 1984 | George Orwell | 1949 | ✅ |
| 2 | Le Petit Prince | Antoine de Saint-Exupéry | 1943 | ✅ |
| 3 | Dune | Frank Herbert | 1965 | ❌ |

**Actions possibles :**
- ✏️ **Double-clic** sur une cellule pour l'éditer
- ➕ **Insérer** une nouvelle ligne
- 🗑️ **Supprimer** des lignes
- 🔍 **Filtrer** les données (barre de recherche en haut)
- 📊 **Trier** par colonne (clic sur l'en-tête)

#### 💻 Onglet "Execute SQL"

Écrivez et exécutez vos requêtes SQL :

```sql
-- Exemple de requête avec autocomplétion
SELECT titre, auteur, annee
FROM livres
WHERE lu = 1
ORDER BY annee DESC;
```

**Fonctionnalités :**
- 🔤 **Autocomplétion** des noms de tables et colonnes
- ⚡ **Exécution rapide** avec `Ctrl+Enter` ou `F5`
- 📜 **Historique** des requêtes exécutées
- 💾 **Sauvegarde** de vos requêtes favorites
- 📊 Résultats affichés dans un tableau interactif

#### 🛠️ Autres Fonctionnalités Utiles

**Export de données :**
- Fichier → Export → **CSV**, **JSON**, **SQL**
- Parfait pour partager ou analyser dans Excel/Pandas

**Import de données :**
- Fichier → Import → Charger un CSV/JSON dans une table

**Conception visuelle de tables :**
- Créer des tables en cliquant, sans écrire de SQL !

**Gestion des index :**
- Améliorer les performances de vos requêtes

---

## 🌐 Méthode 2 : sqlite-web (Interface Web dans Docker)

**sqlite-web** est une interface web légère qui tourne dans un conteneur Docker. Idéal si vous ne voulez rien installer !

### A. Configuration avec Docker Compose

Créez un fichier `docker-compose.yml` :

```yaml
version: '3.8'

services:
  sqlite:
    image: alpine:latest
    container_name: sqlite_db
    command: tail -f /dev/null
    volumes:
      - ./data:/data
    networks:
      - sqlite_network

  sqlite-web:
    image: coleifer/sqlite-web
    container_name: sqlite_web_ui
    ports:
      # Interface accessible sur http://localhost:8080
      - "8080:8080"
    volumes:
      # Partage le même dossier que le conteneur SQLite
      - ./data:/data
    # Spécifie quelle base ouvrir par défaut
    command: ["-H", "0.0.0.0", "-x", "/data/bibliotheque.db"]
    depends_on:
      - sqlite
    networks:
      - sqlite_network

networks:
  sqlite_network:
    driver: bridge
```

**Explications :**
- `coleifer/sqlite-web` : Image Docker avec l'interface web
- `-H 0.0.0.0` : Écoute sur toutes les interfaces (nécessaire pour Docker)
- `-x /data/bibliotheque.db` : Ouvre cette base par défaut
- Port `8080` : Interface accessible sur votre navigateur

### B. Lancement

```bash
# Démarrer les services
docker-compose up -d

# Vérifier que tout tourne
docker-compose ps
```

### C. Accès à l'Interface

Ouvrez votre navigateur et allez sur :

```
http://localhost:8080
```

Vous verrez une interface web avec :

- 📊 **Liste des tables** dans le menu de gauche
- 📄 **Données de la table** au centre
- 💻 **Éditeur SQL** en haut
- 🔍 **Recherche** et filtres

### D. Fonctionnalités de sqlite-web

#### Visualisation des Données

```
┌─────────────────────────────────────┐
│  sqlite-web - bibliotheque.db       │
├─────────────────────────────────────┤
│ Tables:                             │
│  📋 livres (15 rows)                │
│  📋 auteurs (8 rows)                │
│  📋 emprunts (42 rows)              │
├─────────────────────────────────────┤
│ Query:                              │
│  SELECT * FROM livres;              │
│  [Execute Query]                    │
├─────────────────────────────────────┤
│ Results:                            │
│  id | titre        | auteur         │
│  1  | 1984         | G. Orwell      │
│  2  | Petit Prince | Saint-Exupéry  │
└─────────────────────────────────────┘
```

#### Exécution de Requêtes SQL

```sql
-- Requêtes interactives
SELECT titre, COUNT(*) as nb_emprunts
FROM livres
JOIN emprunts ON livres.id = emprunts.livre_id
GROUP BY titre
ORDER BY nb_emprunts DESC;
```

#### Export de Résultats

- **Format JSON** : Cliquez sur "JSON" en haut des résultats
- **Format CSV** : Cliquez sur "CSV"
- **Format SQL** : Exportez la structure ou les données

### E. Configuration Avancée

#### Ouvrir Plusieurs Bases

Modifiez la commande pour ne pas spécifier de base par défaut :

```yaml
command: ["-H", "0.0.0.0", "-d", "/data"]
```

Vous pourrez alors **choisir** quelle base ouvrir depuis l'interface.

#### Mode Lecture Seule

Pour éviter les modifications accidentelles :

```yaml
command: ["-H", "0.0.0.0", "-x", "/data/bibliotheque.db", "-r"]
```

L'option `-r` active le **mode lecture seule** (read-only).

#### Authentification par Mot de Passe

```yaml
command: [
  "-H", "0.0.0.0",
  "-x", "/data/bibliotheque.db",
  "-p", "votre_mot_de_passe_ici"
]
```

**⚠️ Attention :** Ne mettez pas de vrai mot de passe en production dans un fichier docker-compose !

---

## 🚀 Méthode 3 : DBeaver (Multi-BDD Professionnel)

**DBeaver** est un outil **professionnel et gratuit** qui supporte **des dizaines de bases de données** (SQLite, PostgreSQL, MySQL, MongoDB...).

### A. Installation

1. Téléchargez depuis [dbeaver.io](https://dbeaver.io/download/)
2. Choisissez **DBeaver Community Edition** (gratuit)
3. Installez normalement
4. Lancez DBeaver

### B. Connexion à SQLite

**Étape 1 : Créer une nouvelle connexion**

1. Cliquez sur l'icône **"New Database Connection"** (prise électrique) ou `Ctrl+Shift+N`
2. Dans la liste, sélectionnez **SQLite**
3. Cliquez sur **Next**

**Étape 2 : Configurer la connexion**

```
┌────────────────────────────────────────┐
│ SQLite Connection Settings             │
├────────────────────────────────────────┤
│ Path:                                  │
│  [Browse...] C:\sqlite-data\biblio.db  │
│                                        │
│ ☐ Open database in read only mode     │
│ ☐ Enable shared cache                 │
│                                        │
│ [Test Connection]  [Finish]  [Cancel]  │
└────────────────────────────────────────┘
```

1. Cliquez sur **"Browse..."**
2. Naviguez vers votre dossier Docker (ex: `C:\sqlite-data\`)
3. Sélectionnez votre fichier `.db`
4. Cliquez sur **"Test Connection"** (DBeaver téléchargera les drivers si nécessaire)
5. Si ça fonctionne, cliquez sur **"Finish"**

**Étape 3 : Explorer votre base**

Vous verrez maintenant dans le panneau de gauche :

```
📁 SQLite - bibliotheque.db
  ├── 📊 Tables
  │   ├── 📋 livres
  │   ├── 📋 auteurs
  │   └── 📋 emprunts
  ├── 🔍 Indexes
  └── 👁️ Views
```

### C. Fonctionnalités de DBeaver

#### Visualisation des Données

- **Double-cliquez** sur une table pour voir son contenu
- **Vue grille** (comme Excel) pour éditer les données
- **Vue formulaire** pour éditer ligne par ligne
- **Filtres avancés** sur chaque colonne

#### Diagramme ER (Entity-Relationship)

Visualisez les relations entre vos tables :

1. Clic droit sur votre base → **View Diagram**
2. Vous verrez un schéma visuel de votre base

```
┌──────────────┐       ┌──────────────┐
│   livres     │───────│   emprunts   │
├──────────────┤  1:N  ├──────────────┤
│ id (PK)      │       │ id (PK)      │
│ titre        │       │ livre_id (FK)│
│ auteur       │       │ date         │
└──────────────┘       └──────────────┘
```

#### Éditeur SQL Avancé

- **Autocomplétion intelligente** des tables, colonnes, fonctions
- **Coloration syntaxique**
- **Exécution de scripts** multiples
- **Historique** de toutes vos requêtes
- **Favoris** pour sauvegarder vos requêtes

**Exemple d'utilisation :**

```sql
-- DBeaver propose l'autocomplétion en tapant
SELECT l.titre, a.nom AS auteur, COUNT(e.id) AS nb_emprunts
FROM livres l
  JOIN auteurs a ON l.auteur_id = a.id
  LEFT JOIN emprunts e ON l.id = e.livre_id
GROUP BY l.id
HAVING nb_emprunts > 5
ORDER BY nb_emprunts DESC;
```

#### Import/Export Massif

- **Import CSV** : Clic droit sur table → Import Data
- **Export multi-formats** : CSV, JSON, XML, SQL, HTML, Markdown...
- **Transfert entre BDD** : Copier des données d'une base à une autre

#### Gestion de Plusieurs Connexions

Connectez-vous à plusieurs bases simultanément :

```
Connexions
├── 📁 SQLite - bibliotheque.db
├── 📁 SQLite - notes.db
├── 📁 PostgreSQL - production
└── 📁 MariaDB - dev
```

Parfait quand vous travaillez avec plusieurs types de BDD !

---

## 🔧 Méthode 4 : Adminer (Ultra-Léger dans Docker)

**Adminer** est une interface PHP minimaliste qui tient dans **un seul fichier**. Très léger !

### A. Configuration Docker Compose

```yaml
version: '3.8'

services:
  sqlite:
    image: alpine:latest
    container_name: sqlite_db
    command: tail -f /dev/null
    volumes:
      - ./data:/data
    networks:
      - sqlite_network

  adminer:
    image: adminer:latest
    container_name: adminer_ui
    ports:
      - "8081:8080"
    environment:
      # Support de SQLite
      ADMINER_DEFAULT_SERVER: sqlite
    volumes:
      - ./data:/var/www/html/data
    networks:
      - sqlite_network

networks:
  sqlite_network:
```

### B. Utilisation

1. Lancez : `docker-compose up -d`
2. Ouvrez : `http://localhost:8081`
3. Dans le formulaire de connexion :
   - **System** : SQLite 3
   - **Server** : Laissez vide
   - **Username** : Laissez vide
   - **Password** : Laissez vide
   - **Database** : `data/bibliotheque.db`
4. Cliquez sur **Login**

**Fonctionnalités :**
- 📊 Visualisation de tables
- ✏️ Édition de données
- 💻 Exécution de SQL
- 📤 Export SQL/CSV

**Avantages :**
- ⚡ Très léger (seulement ~500 KB)
- 🚀 Démarrage instantané
- 🌐 Interface web familière

**Inconvénients :**
- 🔧 Moins de fonctionnalités qu'un outil dédié
- 🎨 Interface basique

---

## 📊 Comparaison des Outils

| Outil | Type | Gratuit | Difficulté | Performance | Recommandé pour |
|-------|------|---------|------------|-------------|-----------------|
| **DB Browser** | Desktop | ✅ | ⭐ | ⚡⚡⚡ | Débutants |
| **sqlite-web** | Web (Docker) | ✅ | ⭐⭐ | ⚡⚡ | Pas d'installation |
| **DBeaver** | Desktop | ✅ | ⭐⭐⭐ | ⚡⚡⚡ | Multi-BDD |
| **Adminer** | Web (Docker) | ✅ | ⭐⭐ | ⚡⚡ | Très léger |
| **DataGrip** | Desktop | ❌ (payant) | ⭐⭐⭐⭐ | ⚡⚡⚡⚡ | Professionnels |

### 🎯 Choix Recommandé par Profil

**👶 Débutant complet :**
→ **DB Browser for SQLite**
- Simple à installer
- Interface intuitive
- Documentation en français

**🐳 Fan de Docker :**
→ **sqlite-web**
- Rien à installer
- Interface moderne
- Léger

**👨‍💻 Développeur Multi-BDD :**
→ **DBeaver**
- Un outil pour toutes vos bases
- Fonctionnalités avancées
- Gratuit et complet

**⚡ Besoin de rapidité :**
→ **Adminer**
- Ultra-léger
- Démarre en 5 secondes
- Basique mais efficace

---

## 🔐 Accéder aux Bases depuis le Réseau Local

Si vous voulez que d'autres personnes de votre réseau (collègues, équipe) accèdent à l'interface web...

### Avec sqlite-web

```yaml
services:
  sqlite-web:
    image: coleifer/sqlite-web
    ports:
      # Exposer sur toutes les interfaces
      - "0.0.0.0:8080:8080"
    command: ["-H", "0.0.0.0", "-x", "/data/bibliotheque.db"]
```

**Accès depuis un autre PC :**
```
http://IP_DE_VOTRE_PC:8080
```

**Trouver votre IP :**
```bash
# Windows
ipconfig

# Linux/macOS
ip addr
# ou
ifconfig
```

### ⚠️ Sécurité Important

**NE JAMAIS** exposer ces interfaces sur Internet sans :
- 🔐 Authentification forte
- 🔒 HTTPS (SSL/TLS)
- 🛡️ Pare-feu configuré
- 🔑 Mots de passe robustes

**Pour le développement local uniquement !**

---

## 🎨 Astuce : Thèmes et Personnalisation

### DBeaver

**Changer le thème :**
1. Window → Preferences → Appearance
2. Choisir **Dark Theme** ou **Light Theme**

**Polices de code :**
1. Window → Preferences → Editors → Text Editors
2. Modifier la police et la taille

### DB Browser

**Personnaliser l'apparence :**
1. Edit → Preferences → General
2. Changer la langue, les couleurs, etc.

---

## 📱 Bonus : Outils Mobiles

Vous pouvez aussi gérer vos bases SQLite depuis votre smartphone/tablette !

### Android

- **SQLite Editor** : Gratuit, complet
- **Database Manager** : Simple, open-source

### iOS

- **SQLite Mobile** : Interface claire
- **DB Browser** : Version iOS de l'outil desktop

**Utilisation :**
1. Exportez votre `.db` depuis Docker
2. Transférez-le sur votre mobile (via cloud, USB, email...)
3. Ouvrez-le avec l'application

---

## 🛠️ Workflow Complet Recommandé

### Configuration Idéale pour Débutants

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Base SQLite principale
  sqlite:
    image: alpine:latest
    container_name: sqlite_db
    command: tail -f /dev/null
    volumes:
      - ./data:/data
    restart: unless-stopped

  # Interface web pour accès rapide
  sqlite-web:
    image: coleifer/sqlite-web
    container_name: sqlite_web
    ports:
      - "8080:8080"
    volumes:
      - ./data:/data
    command: ["-H", "0.0.0.0", "-d", "/data"]
    depends_on:
      - sqlite
    restart: unless-stopped
```

**Utilisation au quotidien :**

1. **Développement** : DB Browser for SQLite
   - Édition de structure
   - Requêtes complexes
   - Export de données

2. **Consultation rapide** : sqlite-web (http://localhost:8080)
   - Vérifier des données
   - Tester des requêtes simples
   - Partager avec l'équipe

3. **Ligne de commande** : Conteneur Docker
   - Scripts automatisés
   - Backups
   - Migrations

---

## 🧹 Nettoyage

### Arrêter les interfaces web

```bash
# Avec Docker Compose
docker-compose down

# Conteneur individuel
docker stop sqlite_web
docker rm sqlite_web
```

### Désinstaller les outils desktop

**DB Browser (Windows) :**
1. Panneau de configuration → Programmes → Désinstaller

**DBeaver :**
1. Suivre la procédure de désinstallation standard

**Linux :**
```bash
# DB Browser
sudo apt remove sqlitebrowser

# DBeaver
sudo dpkg -r dbeaver-ce
```

---

## 💡 Cas d'Usage Réels

### 1. Développement d'Application Mobile

```yaml
# Structure projet
mon-app-mobile/
├── docker-compose.yml
├── data/
│   ├── dev.db          # Base de développement
│   ├── test.db         # Base de tests
│   └── production.db   # Copie de prod pour debug
└── backups/
```

**Workflow :**
- Développez dans DB Browser
- Testez via sqlite-web
- Exportez pour l'app mobile

### 2. Analyse de Données

```sql
-- Requête d'analyse dans DBeaver
SELECT
  strftime('%Y-%m', date_creation) AS mois,
  COUNT(*) AS nb_nouveaux_utilisateurs,
  AVG(age) AS age_moyen
FROM utilisateurs
WHERE date_creation >= date('now', '-12 months')
GROUP BY mois
ORDER BY mois;

-- Export en CSV pour Excel/Python
```

### 3. Enseignement/Formation

```yaml
# Configuration pour cours SQL
services:
  sqlite-web:
    image: coleifer/sqlite-web
    ports:
      - "8080:8080"
    volumes:
      - ./cours/exercices:/data
    # Mode lecture seule pour les étudiants
    command: ["-H", "0.0.0.0", "-d", "/data", "-r"]
```

---

## 🎓 Pour Aller Plus Loin

- 📖 [Retour à SQLite en conteneur](01-sqlite-conteneur.md)
- 💾 [SQLite avec volume persistant](02-sqlite-volume-persistant.md)
- 🔧 [Annexe F : Outils de gestion de BDD](../annexes/F-outils-gestion.md)
- 📚 [Documentation DB Browser](https://sqlitebrowser.org/docs/)
- 🌐 [sqlite-web sur GitHub](https://github.com/coleifer/sqlite-web)

---

## ✅ Points Clés à Retenir

1. 🖥️ **DB Browser** = Meilleur pour débuter (gratuit, simple, complet)
2. 🌐 **sqlite-web** = Interface web sans installation
3. 🚀 **DBeaver** = Outil professionnel multi-BDD
4. ⚡ **Adminer** = Ultra-léger pour tests rapides
5. 📊 Tous ces outils peuvent ouvrir vos fichiers `.db` Docker
6. 🔐 N'exposez jamais ces interfaces sur Internet sans sécurité
7. 💾 Les modifications sont **directement écrites** dans le fichier `.db`

---

## ❓ Questions Fréquentes

**Q : Puis-je utiliser plusieurs outils en même temps ?**
R : Oui ! Mais attention aux **verrous de fichiers**. Si un outil écrit dans la base, les autres doivent attendre.

**Q : Quel outil consomme le moins de ressources ?**
R : **Adminer** (web) et **DB Browser** (desktop) sont les plus légers.

**Q : Puis-je modifier la structure de tables avec ces outils ?**
R : Oui, tous le permettent. DB Browser et DBeaver le font de manière très visuelle.

**Q : Les données modifiées avec ces outils sont-elles persistantes ?**
R : Oui ! Elles sont écrites directement dans le fichier `.db` de votre Docker volume.

**Q : Puis-je utiliser ces outils pour d'autres bases que SQLite ?**
R : **DBeaver** et **Adminer** supportent beaucoup d'autres BDD (PostgreSQL, MySQL, MongoDB...). DB Browser et sqlite-web sont **spécifiques à SQLite**.

**Q : Comment ouvrir un fichier .db crypté ?**
R : DB Browser supporte **SQLCipher** (SQLite crypté). Installez le plugin et fournissez la clé de chiffrement.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
