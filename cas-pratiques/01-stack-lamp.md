# Stack LAMP avec Docker (Apache + MariaDB + PHP)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous guide dans la création d'une **stack LAMP complète** avec Docker. LAMP est l'acronyme de **Linux + Apache + MySQL/MariaDB + PHP**, une combinaison classique et éprouvée pour développer des applications web dynamiques.

**Ce que vous allez apprendre :**
- Comprendre l'architecture d'une stack LAMP
- Déployer Apache, MariaDB et PHP avec Docker Compose
- Faire communiquer les différents services entre eux
- Configurer un environnement de développement web complet
- Tester votre stack avec une application PHP simple
- Gérer et maintenir votre environnement

**Durée estimée :** 30-40 minutes

---

## 🎯 Qu'est-ce qu'une Stack LAMP ?

### Définition

Une **stack LAMP** est un ensemble de logiciels open source utilisés ensemble pour héberger des sites web et applications web dynamiques.

### Les 4 composants

| Composant | Rôle | Dans notre stack |
|-----------|------|------------------|
| **L**inux | Système d'exploitation | Fourni par Docker (conteneurs Linux) |
| **A**pache | Serveur web | Sert les pages web aux visiteurs |
| **M**ySQL / **M**ariaDB | Base de données | Stocke les données (utilisateurs, articles, etc.) |
| **P**HP | Langage de programmation | Génère les pages web dynamiques |

### Schéma de fonctionnement

```
┌─────────────────────────────────────────────────────────┐
│                    NAVIGATEUR                           │
│              (http://localhost:8080)                    │
└─────────────────────┬───────────────────────────────────┘
                      │
                      │ Requête HTTP
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              CONTENEUR APACHE + PHP                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │ Apache écoute sur le port 80                      │  │
│  │ - Reçoit la requête                               │  │
│  │ - Exécute le script PHP                           │  │
│  │ - Renvoie la page HTML générée                    │  │
│  └───────────────┬───────────────────────────────────┘  │
└──────────────────┼──────────────────────────────────────┘
                   │
                   │ Requête SQL
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│              CONTENEUR MARIADB                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │ MariaDB écoute sur le port 3306                   │  │
│  │ - Reçoit les requêtes SQL                         │  │
│  │ - Renvoie les résultats                           │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔧 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ Docker installé (version 20.10+)
- ✅ Docker Compose installé (version 2.0+)
- ✅ Un éditeur de texte (VS Code, Sublime Text, etc.)
- ✅ Avoir lu les [concepts Docker de base](../00-introduction/02-concepts-docker.md) (recommandé)

**Vérification rapide :**

```bash
docker --version
docker-compose --version
```

---

## 📁 Étape 1 : Structure du projet

### 1.1 Créer l'arborescence

Créez un nouveau dossier pour votre projet LAMP et organisez-le ainsi :

```bash
# Créer le dossier principal
mkdir lamp-stack
cd lamp-stack

# Créer les sous-dossiers
mkdir -p www/public
mkdir -p mariadb/data
mkdir -p mariadb/init
```

**Votre structure sera :**

```
lamp-stack/
├── docker-compose.yml        # Configuration Docker
├── www/                       # Code source de votre site
│   └── public/               # Racine web (accessible par Apache)
│       └── index.php         # Page d'accueil (à créer)
└── mariadb/
    ├── data/                 # Données de la base (généré automatiquement)
    └── init/                 # Scripts SQL d'initialisation
        └── 01-init-db.sql    # Script d'init (à créer)
```

### 1.2 Pourquoi cette organisation ?

| Dossier | Rôle |
|---------|------|
| `www/public/` | Contient vos fichiers PHP/HTML accessibles via le navigateur |
| `mariadb/data/` | Stockage persistant de la base de données |
| `mariadb/init/` | Scripts SQL exécutés automatiquement au premier démarrage |

---

## 🐳 Étape 2 : Configuration Docker Compose

### 2.1 Créer le fichier docker-compose.yml

Dans le dossier `lamp-stack`, créez un fichier `docker-compose.yml` :

```yaml
version: '3.8'

services:
  # ==========================================
  # SERVICE 1 : BASE DE DONNÉES MARIADB
  # ==========================================
  mariadb:
    image: mariadb:10.11
    container_name: lamp_mariadb
    restart: unless-stopped

    environment:
      # Mot de passe root (⚠️ CHANGEZ-LE !)
      MYSQL_ROOT_PASSWORD: root_password_secure

      # Base de données créée automatiquement
      MYSQL_DATABASE: lamp_db

      # Utilisateur applicatif (non-root)
      MYSQL_USER: lamp_user
      MYSQL_PASSWORD: lamp_password_secure

    volumes:
      # Données persistantes
      - ./mariadb/data:/var/lib/mysql

      # Scripts d'initialisation (exécutés au 1er démarrage)
      - ./mariadb/init:/docker-entrypoint-initdb.d

    networks:
      - lamp_network

    # Vérification de santé (optionnel mais recommandé)
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      timeout: 5s
      retries: 3

  # ==========================================
  # SERVICE 2 : SERVEUR WEB APACHE + PHP
  # ==========================================
  apache-php:
    image: php:8.2-apache
    container_name: lamp_apache
    restart: unless-stopped

    # Le service Apache démarre après que MariaDB soit prêt
    depends_on:
      - mariadb

    ports:
      # Port externe:port interne
      # Accès via http://localhost:8080
      - "8080:80"

    volumes:
      # Montage du code source
      # Tout ce qui est dans ./www/public/ sera accessible via Apache
      - ./www/public:/var/www/html

    networks:
      - lamp_network

    # Variables d'environnement pour PHP
    environment:
      # Configuration de la connexion à MariaDB
      # Ces variables seront accessibles dans vos scripts PHP
      DB_HOST: mariadb          # Nom du service MariaDB
      DB_PORT: 3306
      DB_NAME: lamp_db
      DB_USER: lamp_user
      DB_PASSWORD: lamp_password_secure

# ==========================================
# RÉSEAU DOCKER PARTAGÉ
# ==========================================
networks:
  lamp_network:
    driver: bridge
```

### 2.2 Explication détaillée

#### Section MariaDB

```yaml
mariadb:
  image: mariadb:10.11
```
- Utilise l'image officielle MariaDB version 10.11 (version LTS stable)

```yaml
environment:
  MYSQL_ROOT_PASSWORD: root_password_secure
  MYSQL_DATABASE: lamp_db
  MYSQL_USER: lamp_user
  MYSQL_PASSWORD: lamp_password_secure
```
- `MYSQL_ROOT_PASSWORD` : Mot de passe du super-administrateur (root)
- `MYSQL_DATABASE` : Nom de la base de données créée automatiquement
- `MYSQL_USER` / `MYSQL_PASSWORD` : Utilisateur applicatif avec droits limités (meilleure pratique)

```yaml
volumes:
  - ./mariadb/data:/var/lib/mysql
```
- **Persistance des données** : Les données de la base sont stockées sur votre machine, pas dans le conteneur
- Si vous supprimez le conteneur, les données restent

```yaml
networks:
  - lamp_network
```
- Connecte MariaDB au réseau `lamp_network` pour communiquer avec Apache

#### Section Apache + PHP

```yaml
apache-php:
  image: php:8.2-apache
```
- Image officielle PHP 8.2 avec Apache déjà intégré
- Alternative : `php:8.1-apache` ou `php:7.4-apache` selon vos besoins

```yaml
depends_on:
  - mariadb
```
- Docker démarre MariaDB **avant** Apache
- Évite les erreurs de connexion au démarrage

```yaml
ports:
  - "8080:80"
```
- **Port mapping** : Le port 80 du conteneur (Apache) est accessible via le port 8080 de votre machine
- Accès : `http://localhost:8080`

```yaml
volumes:
  - ./www/public:/var/www/html
```
- **Bind mount** : Votre dossier `www/public/` est directement accessible par Apache
- Les modifications dans vos fichiers PHP sont instantanées (pas besoin de redémarrer)

```yaml
environment:
  DB_HOST: mariadb
```
- **Important** : On utilise le **nom du service** (`mariadb`) comme hôte, pas une IP
- Docker résout automatiquement ce nom vers l'IP du conteneur MariaDB

#### Section Réseau

```yaml
networks:
  lamp_network:
    driver: bridge
```
- Crée un réseau privé pour que les conteneurs communiquent entre eux
- Isolé du reste de Docker

---

## 📄 Étape 3 : Créer le script d'initialisation SQL

### 3.1 Script SQL de base

Créez le fichier `mariadb/init/01-init-db.sql` :

```sql
-- ==========================================
-- Script d'initialisation de la base LAMP
-- Exécuté automatiquement au 1er démarrage
-- ==========================================

-- Utiliser la base de données créée par Docker
USE lamp_db;

-- ==========================================
-- TABLE : users (utilisateurs)
-- ==========================================
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ==========================================
-- TABLE : articles (articles de blog)
-- ==========================================
CREATE TABLE IF NOT EXISTS articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    author_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_author (author_id),
    INDEX idx_created (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ==========================================
-- DONNÉES DE TEST
-- ==========================================

-- Insérer des utilisateurs de test
-- Note : Les mots de passe sont hashés (ne JAMAIS stocker en clair !)
INSERT INTO users (username, email, password_hash) VALUES
    ('admin', 'admin@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi'), -- password
    ('john_doe', 'john@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi'),
    ('jane_smith', 'jane@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');

-- Insérer des articles de test
INSERT INTO articles (title, content, author_id) VALUES
    (
        'Bienvenue sur notre site LAMP',
        'Ceci est le premier article de notre site web développé avec la stack LAMP. Apache sert les pages, PHP les génère dynamiquement, et MariaDB stocke les données.',
        1
    ),
    (
        'Introduction à Docker',
        'Docker permet de conteneuriser les applications. Avec Docker Compose, nous pouvons orchestrer plusieurs services comme Apache, PHP et MariaDB ensemble.',
        2
    ),
    (
        'Les avantages de MariaDB',
        'MariaDB est un fork open source de MySQL. Il offre de meilleures performances et des fonctionnalités avancées tout en restant compatible avec MySQL.',
        3
    );

-- ==========================================
-- VÉRIFICATIONS
-- ==========================================

-- Afficher le nombre d'utilisateurs créés
SELECT 'Users created:' AS Info, COUNT(*) AS Count FROM users;

-- Afficher le nombre d'articles créés
SELECT 'Articles created:' AS Info, COUNT(*) AS Count FROM articles;

-- Confirmer la création des tables
SHOW TABLES;
```

### 3.2 Comprendre le script

| Section | Explication |
|---------|-------------|
| `USE lamp_db;` | Sélectionne la base de données créée par Docker |
| `CREATE TABLE IF NOT EXISTS` | Crée la table uniquement si elle n'existe pas déjà |
| `ENGINE=InnoDB` | Utilise le moteur InnoDB (transactionnel, clés étrangères) |
| `CHARSET=utf8mb4` | Support complet des caractères Unicode (émojis inclus) |
| `FOREIGN KEY` | Lie les articles aux utilisateurs (intégrité référentielle) |
| `INDEX` | Optimise les recherches sur certaines colonnes |
| `ON DELETE CASCADE` | Si un utilisateur est supprimé, ses articles le sont aussi |

---

## 💻 Étape 4 : Créer la page PHP de test

### 4.1 Page d'accueil (index.php)

Créez le fichier `www/public/index.php` :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stack LAMP avec Docker</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: #333;
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 10px;
            box-shadow: 0 10px 50px rgba(0,0,0,0.2);
            padding: 40px;
        }

        h1 {
            color: #667eea;
            margin-bottom: 10px;
        }

        .success {
            background: #d4edda;
            border: 1px solid #c3e6cb;
            color: #155724;
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
        }

        .error {
            background: #f8d7da;
            border: 1px solid #f5c6cb;
            color: #721c24;
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
        }

        .info-box {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 5px;
            margin: 20px 0;
        }

        .info-box h2 {
            color: #495057;
            margin-bottom: 15px;
            font-size: 1.3em;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }

        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #dee2e6;
        }

        th {
            background: #667eea;
            color: white;
            font-weight: 600;
        }

        tr:hover {
            background: #f8f9fa;
        }

        .badge {
            display: inline-block;
            padding: 4px 10px;
            border-radius: 12px;
            font-size: 0.85em;
            font-weight: 600;
        }

        .badge-success {
            background: #28a745;
            color: white;
        }

        .badge-info {
            background: #17a2b8;
            color: white;
        }

        code {
            background: #f4f4f4;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🐳 Stack LAMP avec Docker</h1>
        <p style="color: #6c757d; margin-bottom: 30px;">
            Linux + Apache + MariaDB + PHP
        </p>

        <?php
        // ==========================================
        // CONFIGURATION DE LA CONNEXION
        // ==========================================

        // Récupération des variables d'environnement
        $db_host = getenv('DB_HOST') ?: 'mariadb';
        $db_port = getenv('DB_PORT') ?: '3306';
        $db_name = getenv('DB_NAME') ?: 'lamp_db';
        $db_user = getenv('DB_USER') ?: 'lamp_user';
        $db_password = getenv('DB_PASSWORD') ?: '';

        // ==========================================
        // TENTATIVE DE CONNEXION À LA BASE
        // ==========================================

        try {
            // Création de la connexion PDO
            $pdo = new PDO(
                "mysql:host=$db_host;port=$db_port;dbname=$db_name;charset=utf8mb4",
                $db_user,
                $db_password,
                [
                    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                    PDO::ATTR_EMULATE_PREPARES => false,
                ]
            );

            echo '<div class="success">';
            echo '✅ <strong>Connexion réussie à MariaDB !</strong><br>';
            echo 'Base de données : <code>' . htmlspecialchars($db_name) . '</code><br>';
            echo 'Hôte : <code>' . htmlspecialchars($db_host) . ':' . htmlspecialchars($db_port) . '</code>';
            echo '</div>';

            // ==========================================
            // INFORMATIONS SYSTÈME
            // ==========================================

            echo '<div class="info-box">';
            echo '<h2>📊 Informations du système</h2>';
            echo '<table>';
            echo '<tr><th>Composant</th><th>Valeur</th></tr>';
            echo '<tr><td>Version PHP</td><td><span class="badge badge-info">' . phpversion() . '</span></td></tr>';

            // Version de MariaDB
            $stmt = $pdo->query('SELECT VERSION() as version');
            $db_version = $stmt->fetch()['version'];
            echo '<tr><td>Version MariaDB</td><td><span class="badge badge-info">' . htmlspecialchars($db_version) . '</span></td></tr>';

            echo '<tr><td>Serveur Web</td><td><span class="badge badge-info">' . $_SERVER['SERVER_SOFTWARE'] . '</span></td></tr>';
            echo '</table>';
            echo '</div>';

            // ==========================================
            // STATISTIQUES DE LA BASE DE DONNÉES
            // ==========================================

            echo '<div class="info-box">';
            echo '<h2>📈 Statistiques de la base de données</h2>';

            // Compter les utilisateurs
            $stmt = $pdo->query('SELECT COUNT(*) as count FROM users');
            $users_count = $stmt->fetch()['count'];

            // Compter les articles
            $stmt = $pdo->query('SELECT COUNT(*) as count FROM articles');
            $articles_count = $stmt->fetch()['count'];

            echo '<p>👥 <strong>Utilisateurs :</strong> ' . $users_count . '</p>';
            echo '<p>📝 <strong>Articles :</strong> ' . $articles_count . '</p>';
            echo '</div>';

            // ==========================================
            // LISTE DES ARTICLES
            // ==========================================

            echo '<div class="info-box">';
            echo '<h2>📚 Derniers articles</h2>';

            $stmt = $pdo->query('
                SELECT
                    a.id,
                    a.title,
                    a.content,
                    u.username as author,
                    a.created_at
                FROM articles a
                JOIN users u ON a.author_id = u.id
                ORDER BY a.created_at DESC
                LIMIT 10
            ');

            $articles = $stmt->fetchAll();

            if (count($articles) > 0) {
                echo '<table>';
                echo '<tr><th>#</th><th>Titre</th><th>Auteur</th><th>Date</th></tr>';

                foreach ($articles as $article) {
                    echo '<tr>';
                    echo '<td>' . htmlspecialchars($article['id']) . '</td>';
                    echo '<td><strong>' . htmlspecialchars($article['title']) . '</strong><br>';
                    echo '<small>' . htmlspecialchars(substr($article['content'], 0, 100)) . '...</small></td>';
                    echo '<td>' . htmlspecialchars($article['author']) . '</td>';
                    echo '<td>' . date('d/m/Y H:i', strtotime($article['created_at'])) . '</td>';
                    echo '</tr>';
                }

                echo '</table>';
            } else {
                echo '<p>Aucun article trouvé.</p>';
            }

            echo '</div>';

        } catch (PDOException $e) {
            // ==========================================
            // GESTION DES ERREURS
            // ==========================================

            echo '<div class="error">';
            echo '❌ <strong>Erreur de connexion à la base de données</strong><br><br>';
            echo '<strong>Message d\'erreur :</strong><br>';
            echo '<code>' . htmlspecialchars($e->getMessage()) . '</code><br><br>';
            echo '<strong>Vérifications à effectuer :</strong><ul>';
            echo '<li>Le conteneur MariaDB est-il démarré ? <code>docker-compose ps</code></li>';
            echo '<li>Les variables d\'environnement sont-elles correctes ?</li>';
            echo '<li>Le réseau Docker fonctionne-t-il ? <code>docker network inspect lamp_network</code></li>';
            echo '</ul>';
            echo '</div>';
        }
        ?>

        <div style="margin-top: 40px; padding-top: 20px; border-top: 1px solid #dee2e6; color: #6c757d; text-align: center;">
            <p>🐳 Stack LAMP avec Docker - Guide pour développeurs</p>
            <p><small>Apache <?php echo $_SERVER['SERVER_SOFTWARE']; ?> | PHP <?php echo phpversion(); ?> | MariaDB</small></p>
        </div>
    </div>
</body>
</html>
```

### 4.2 Comprendre le code PHP

#### Connexion à la base de données

```php
$db_host = getenv('DB_HOST') ?: 'mariadb';
```
- `getenv()` récupère les variables d'environnement définies dans `docker-compose.yml`
- L'opérateur `?:` fournit une valeur par défaut si la variable n'existe pas

```php
$pdo = new PDO(
    "mysql:host=$db_host;port=$db_port;dbname=$db_name;charset=utf8mb4",
    $db_user,
    $db_password,
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);
```
- **PDO** (PHP Data Objects) : Interface moderne et sécurisée pour accéder aux bases de données
- `ERRMODE_EXCEPTION` : Les erreurs SQL lèvent des exceptions (facile à déboguer)
- `FETCH_ASSOC` : Les résultats sont des tableaux associatifs (accès par nom de colonne)

#### Requêtes SQL sécurisées

```php
$stmt = $pdo->query('SELECT COUNT(*) as count FROM users');
$users_count = $stmt->fetch()['count'];
```
- Toujours utiliser des **requêtes préparées** pour éviter les injections SQL
- `fetch()` récupère une seule ligne
- `fetchAll()` récupère toutes les lignes

---

## ▶️ Étape 5 : Démarrer la stack

### 5.1 Lancer tous les services

Depuis le dossier `lamp-stack`, exécutez :

```bash
docker-compose up -d
```

**Ce qui se passe :**
1. Docker télécharge les images (si première utilisation)
2. Création du réseau `lamp_network`
3. Démarrage de MariaDB
4. Exécution du script SQL d'initialisation
5. Démarrage d'Apache + PHP

### 5.2 Vérifier que tout fonctionne

```bash
# Voir les conteneurs actifs
docker-compose ps

# Résultat attendu :
# NAME            STATE     PORTS
# lamp_mariadb    Up        3306/tcp
# lamp_apache     Up        0.0.0.0:8080->80/tcp
```

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Appuyez sur Ctrl+C pour quitter
```

---

## 🌐 Étape 6 : Tester la stack

### 6.1 Accéder à la page web

Ouvrez votre navigateur et allez sur :

```
http://localhost:8080
```

**Vous devriez voir :**
- ✅ Message de connexion réussie à MariaDB
- 📊 Informations système (versions PHP, MariaDB, Apache)
- 📈 Statistiques de la base (3 utilisateurs, 3 articles)
- 📚 Liste des articles avec leurs auteurs

### 6.2 Si la page ne s'affiche pas

**Vérifications à effectuer :**

1. **Les conteneurs sont-ils démarrés ?**
   ```bash
   docker-compose ps
   # Les deux doivent être "Up"
   ```

2. **Le port 8080 est-il disponible ?**
   ```bash
   # Windows
   netstat -an | findstr :8080

   # Linux/macOS
   netstat -an | grep :8080
   ```
   Si le port est déjà utilisé, modifiez le port dans `docker-compose.yml` :
   ```yaml
   ports:
     - "8081:80"  # Utilisez le port 8081 au lieu de 8080
   ```

3. **Apache démarre-t-il correctement ?**
   ```bash
   docker-compose logs apache-php
   ```
   Cherchez des erreurs dans les logs.

4. **Le fichier index.php existe-t-il ?**
   ```bash
   ls -la www/public/
   # Vous devez voir index.php
   ```

---

## 🔍 Étape 7 : Explorer la base de données

### 7.1 Connexion au shell MariaDB

```bash
# Se connecter en ligne de commande
docker exec -it lamp_mariadb mariadb -u lamp_user -p

# Entrez le mot de passe : lamp_password_secure
```

### 7.2 Commandes SQL de base

```sql
-- Voir les bases de données
SHOW DATABASES;

-- Utiliser notre base
USE lamp_db;

-- Voir les tables
SHOW TABLES;

-- Afficher tous les utilisateurs
SELECT * FROM users;

-- Afficher tous les articles avec leurs auteurs
SELECT
    a.title,
    u.username as author,
    a.created_at
FROM articles a
JOIN users u ON a.author_id = u.id;

-- Quitter
EXIT;
```

### 7.3 Connexion avec un client graphique

Vous pouvez également vous connecter avec un client comme **DBeaver** ou **HeidiSQL** :

| Paramètre | Valeur |
|-----------|--------|
| Type | MariaDB ou MySQL |
| Hôte | `localhost` |
| Port | `3306` |
| Base de données | `lamp_db` |
| Utilisateur | `lamp_user` |
| Mot de passe | `lamp_password_secure` |

---

## 🛠️ Étape 8 : Développer votre application

### 8.1 Créer une nouvelle page

Créez `www/public/users.php` :

```php
<?php
// Connexion à la base
$pdo = new PDO(
    "mysql:host=mariadb;dbname=lamp_db;charset=utf8mb4",
    "lamp_user",
    "lamp_password_secure"
);

// Récupérer tous les utilisateurs
$stmt = $pdo->query('SELECT * FROM users ORDER BY created_at DESC');
$users = $stmt->fetchAll();
?>

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Liste des utilisateurs</title>
</head>
<body>
    <h1>👥 Liste des utilisateurs</h1>

    <table border="1">
        <tr>
            <th>ID</th>
            <th>Nom d'utilisateur</th>
            <th>Email</th>
            <th>Inscription</th>
        </tr>

        <?php foreach ($users as $user): ?>
        <tr>
            <td><?= htmlspecialchars($user['id']) ?></td>
            <td><?= htmlspecialchars($user['username']) ?></td>
            <td><?= htmlspecialchars($user['email']) ?></td>
            <td><?= date('d/m/Y', strtotime($user['created_at'])) ?></td>
        </tr>
        <?php endforeach; ?>
    </table>

    <p><a href="/">← Retour à l'accueil</a></p>
</body>
</html>
```

**Accès :** `http://localhost:8080/users.php`

### 8.2 Modifier le code en temps réel

**💡 Avantage du bind mount :**
- Vous modifiez `www/public/index.php` sur votre machine
- Rechargez la page dans le navigateur
- **Les changements sont immédiats** (pas besoin de redémarrer Docker)

### 8.3 Ajouter des données

Créez `www/public/add-article.php` :

```php
<?php
// Connexion à la base
$pdo = new PDO(
    "mysql:host=mariadb;dbname=lamp_db;charset=utf8mb4",
    "lamp_user",
    "lamp_password_secure"
);

// Traitement du formulaire
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $title = $_POST['title'] ?? '';
    $content = $_POST['content'] ?? '';
    $author_id = $_POST['author_id'] ?? 1;

    if ($title && $content) {
        $stmt = $pdo->prepare('
            INSERT INTO articles (title, content, author_id)
            VALUES (?, ?, ?)
        ');
        $stmt->execute([$title, $content, $author_id]);

        header('Location: /');
        exit;
    }
}

// Récupérer la liste des auteurs pour le formulaire
$authors = $pdo->query('SELECT id, username FROM users')->fetchAll();
?>

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Nouvel article</title>
</head>
<body>
    <h1>✍️ Créer un nouvel article</h1>

    <form method="POST">
        <div>
            <label>Titre :</label><br>
            <input type="text" name="title" required style="width: 100%; padding: 5px;">
        </div>

        <div style="margin-top: 10px;">
            <label>Contenu :</label><br>
            <textarea name="content" required rows="10" style="width: 100%; padding: 5px;"></textarea>
        </div>

        <div style="margin-top: 10px;">
            <label>Auteur :</label><br>
            <select name="author_id" required>
                <?php foreach ($authors as $author): ?>
                    <option value="<?= $author['id'] ?>">
                        <?= htmlspecialchars($author['username']) ?>
                    </option>
                <?php endforeach; ?>
            </select>
        </div>

        <div style="margin-top: 20px;">
            <button type="submit" style="padding: 10px 20px; cursor: pointer;">
                📝 Publier l'article
            </button>
            <a href="/" style="margin-left: 10px;">Annuler</a>
        </div>
    </form>
</body>
</html>
```

**Accès :** `http://localhost:8080/add-article.php`

---

## 📊 Étape 9 : Monitoring et gestion

### 9.1 Commandes utiles

```bash
# Voir les conteneurs actifs
docker-compose ps

# Logs de tous les services
docker-compose logs

# Logs d'un service spécifique
docker-compose logs mariadb
docker-compose logs apache-php

# Logs en temps réel
docker-compose logs -f

# Statistiques de ressources
docker stats lamp_mariadb lamp_apache

# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer tous les services
docker-compose start

# Redémarrer un service spécifique
docker-compose restart apache-php
```

### 9.2 Accéder aux conteneurs

```bash
# Shell dans le conteneur Apache
docker exec -it lamp_apache bash

# Une fois dedans :
ls -la /var/www/html/  # Voir les fichiers web
cat /etc/apache2/apache2.conf  # Config Apache
exit  # Quitter

# Shell dans le conteneur MariaDB
docker exec -it lamp_mariadb bash

# Une fois dedans :
mariadb -u lamp_user -p  # Se connecter à MariaDB
exit  # Quitter le shell
```

### 9.3 Vérifier l'utilisation des ressources

```bash
# Voir l'espace disque utilisé
docker system df

# Détails sur les volumes
docker volume ls
docker volume inspect lamp-stack_mariadb_data
```

---

## 🔧 Étape 10 : Configuration avancée

### 10.1 Personnaliser la configuration PHP

Créez un fichier `php.ini` personnalisé :

```bash
# Créer le fichier
mkdir -p config
nano config/php.ini
```

**Contenu de `config/php.ini` :**

```ini
; Configuration PHP personnalisée

; Taille maximale des uploads
upload_max_filesize = 50M
post_max_size = 50M

; Limite de mémoire
memory_limit = 256M

; Affichage des erreurs (développement uniquement)
display_errors = On
error_reporting = E_ALL

; Timezone
date.timezone = Europe/Paris
```

**Modifier `docker-compose.yml` :**

```yaml
apache-php:
  # ... (config existante)
  volumes:
    - ./www/public:/var/www/html
    - ./config/php.ini:/usr/local/etc/php/conf.d/custom.ini  # Ajoutez cette ligne
```

**Appliquer :**

```bash
docker-compose restart apache-php
```

### 10.2 Installer des extensions PHP

Si vous avez besoin d'extensions PHP supplémentaires (par exemple PDO pour PostgreSQL), créez un `Dockerfile` personnalisé :

**Créer `Dockerfile.php` :**

```dockerfile
FROM php:8.2-apache

# Installer les extensions PDO pour MySQL (déjà incluses dans l'image de base)
RUN docker-php-ext-install pdo pdo_mysql mysqli

# Installer d'autres extensions si nécessaire
# RUN docker-php-ext-install gd zip

# Activer mod_rewrite Apache
RUN a2enmod rewrite
```

**Modifier `docker-compose.yml` :**

```yaml
apache-php:
  build:
    context: .
    dockerfile: Dockerfile.php
  # ... (reste de la config)
```

**Reconstruire :**

```bash
docker-compose build
docker-compose up -d
```

### 10.3 Ajouter un fichier .htaccess

Pour des URLs propres (réécriture d'URL), créez `www/public/.htaccess` :

```apache
# Activer la réécriture d'URL
RewriteEngine On

# Rediriger tout vers index.php (sauf les fichiers existants)
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?route=$1 [L,QSA]

# Désactiver le listing des répertoires
Options -Indexes

# Configuration PHP locale
php_value upload_max_filesize 50M
php_value post_max_size 50M
```

---

## 🛑 Étape 11 : Arrêt et nettoyage

### 11.1 Arrêter proprement

```bash
# Arrêter tous les services (données conservées)
docker-compose stop

# Redémarrer plus tard
docker-compose start
```

### 11.2 Suppression complète

```bash
# Arrêter et supprimer les conteneurs
docker-compose down

# Supprimer également les volumes (⚠️ SUPPRIME LES DONNÉES)
docker-compose down -v

# Supprimer le réseau
docker network rm lamp-stack_lamp_network

# Supprimer les images (optionnel, pour libérer de l'espace)
docker rmi php:8.2-apache mariadb:10.11
```

### 11.3 Nettoyage manuel des données

```bash
# Supprimer les données MariaDB (⚠️ IRRÉVERSIBLE)
rm -rf mariadb/data/*

# Si vous voulez tout recommencer de zéro
rm -rf mariadb/data
mkdir -p mariadb/data
docker-compose up -d
# Le script d'initialisation sera réexécuté
```

---

## 🐛 Dépannage

### Problème 1 : "Connection refused" à MariaDB

**Symptôme :** La page PHP affiche une erreur de connexion à MariaDB.

**Solutions :**

1. **Vérifier que MariaDB est démarré :**
   ```bash
   docker-compose ps
   # lamp_mariadb doit être "Up"
   ```

2. **Vérifier les logs MariaDB :**
   ```bash
   docker-compose logs mariadb
   ```

3. **Attendre que MariaDB soit prêt :**
   - MariaDB peut mettre 10-20 secondes à démarrer complètement
   - Attendez, puis rechargez la page

4. **Vérifier le réseau :**
   ```bash
   docker network inspect lamp-stack_lamp_network
   # Les deux conteneurs doivent être listés
   ```

### Problème 2 : "Port 8080 already in use"

**Symptôme :** `docker-compose up` échoue avec une erreur de port.

**Solutions :**

1. **Changer le port dans docker-compose.yml :**
   ```yaml
   ports:
     - "8081:80"  # Utilisez 8081 au lieu de 8080
   ```

2. **Ou libérer le port 8080 :**
   ```bash
   # Voir ce qui utilise le port 8080
   # Windows
   netstat -ano | findstr :8080

   # Linux/macOS
   lsof -i :8080

   # Arrêter l'application qui utilise le port
   ```

### Problème 3 : "Access denied for user 'lamp_user'"

**Symptôme :** Erreur d'authentification dans les logs PHP.

**Solutions :**

1. **Vérifier les variables d'environnement :**
   - Dans `docker-compose.yml`, section `mariadb` : `MYSQL_USER` et `MYSQL_PASSWORD`
   - Dans `docker-compose.yml`, section `apache-php` : `DB_USER` et `DB_PASSWORD`
   - **Elles doivent correspondre !**

2. **Recréer les conteneurs :**
   ```bash
   docker-compose down -v
   docker-compose up -d
   ```

### Problème 4 : "Table 'users' doesn't exist"

**Symptôme :** Erreur SQL dans la page PHP.

**Causes possibles :**
- Le script d'initialisation ne s'est pas exécuté
- La base de données existait déjà (script ignoré)

**Solutions :**

1. **Vérifier que les tables existent :**
   ```bash
   docker exec -it lamp_mariadb mariadb -u lamp_user -p
   USE lamp_db;
   SHOW TABLES;
   ```

2. **Réinitialiser complètement la base :**
   ```bash
   # Supprimer les données
   docker-compose down -v
   rm -rf mariadb/data

   # Redémarrer
   docker-compose up -d

   # Le script d'init sera réexécuté
   ```

### Problème 5 : Modifications PHP non prises en compte

**Symptôme :** Vous modifiez `index.php` mais la page ne change pas.

**Solutions :**

1. **Vider le cache du navigateur :**
   - Ctrl + Shift + R (Windows/Linux)
   - Cmd + Shift + R (macOS)

2. **Vérifier le bind mount :**
   ```bash
   docker exec -it lamp_apache ls -la /var/www/html/
   # Vous devez voir index.php avec la bonne date de modification
   ```

3. **Redémarrer Apache :**
   ```bash
   docker-compose restart apache-php
   ```

---

## ✅ Récapitulatif

### Ce que vous avez appris

- ✅ Comprendre l'architecture d'une stack LAMP
- ✅ Déployer Apache, MariaDB et PHP avec Docker Compose
- ✅ Configurer la communication entre services Docker
- ✅ Créer et exécuter des scripts SQL d'initialisation
- ✅ Développer des pages PHP connectées à MariaDB
- ✅ Utiliser les variables d'environnement
- ✅ Monter des volumes pour la persistance et le développement
- ✅ Gérer et déboguer une stack multi-conteneurs

### Fichiers créés

```
lamp-stack/
├── docker-compose.yml          # Configuration Docker Compose
├── mariadb/
│   ├── data/                   # Données persistantes (généré)
│   └── init/
│       └── 01-init-db.sql      # Script d'initialisation
└── www/
    └── public/
        ├── index.php           # Page d'accueil
        ├── users.php           # Liste des utilisateurs (optionnel)
        └── add-article.php     # Ajout d'articles (optionnel)
```

### Commandes essentielles à retenir

```bash
# Démarrer la stack
docker-compose up -d

# Voir les logs
docker-compose logs -f

# Arrêter (données conservées)
docker-compose stop

# Redémarrer
docker-compose start

# Supprimer tout (⚠️ perte de données)
docker-compose down -v

# Accéder à MariaDB
docker exec -it lamp_mariadb mariadb -u lamp_user -p

# Accéder au shell Apache
docker exec -it lamp_apache bash
```

---

## 🚀 Pour aller plus loin

### Extensions possibles de cette stack

1. **Ajouter phpMyAdmin** pour gérer la base de données graphiquement
2. **Installer Composer** pour gérer les dépendances PHP
3. **Ajouter Redis** pour le cache et les sessions
4. **Configurer HTTPS** avec Let's Encrypt
5. **Mettre en place un système d'authentification** complet
6. **Ajouter Adminer** (alternative légère à phpMyAdmin)

### Frameworks compatibles

Cette stack peut faire tourner :
- **Laravel** (framework PHP moderne)
- **WordPress** (CMS le plus populaire)
- **Symfony** (framework PHP robuste)
- **CodeIgniter** (framework léger)
- **Drupal** (CMS avancé)

### Ressources complémentaires

- 📖 [Documentation PHP](https://www.php.net/manual/fr/)
- 📖 [Documentation MariaDB](https://mariadb.com/kb/en/)
- 📖 [Documentation Apache](https://httpd.apache.org/docs/)
- 📖 [PDO PHP](https://www.php.net/manual/fr/book.pdo.php)
- 🎓 [Tutoriel SQL](https://sql.sh/)

---

## 🎉 Félicitations !

Vous avez maintenant une **stack LAMP complète et fonctionnelle** ! Cette base vous permet de :

- 🌐 Développer des applications web dynamiques
- 🗄️ Gérer des données relationnelles avec MariaDB
- 🐳 Utiliser Docker pour un environnement reproductible
- 🚀 Déployer rapidement de nouveaux projets

**Prochain pas :** Explorez les autres cas pratiques du guide !

➡️ [Stack MEAN (MongoDB + Node.js)](02-stack-mean.md)
➡️ [Stack ELK (Elasticsearch + Logstash + Kibana)](03-stack-elk.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
