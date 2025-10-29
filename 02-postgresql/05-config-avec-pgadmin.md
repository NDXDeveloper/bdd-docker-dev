# PostgreSQL - Configuration avec pgAdmin

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

pgAdmin est l'**interface graphique officielle** pour gérer PostgreSQL. Au lieu de taper des commandes SQL dans un terminal, vous pouvez gérer vos bases de données, utilisateurs, tables et requêtes via une interface web moderne et intuitive.

Cette fiche vous guide pour déployer pgAdmin aux côtés de PostgreSQL avec Docker, et les connecter ensemble.

**Ce que vous allez apprendre :**
- Installer pgAdmin avec Docker Compose
- Connecter pgAdmin à PostgreSQL
- Naviguer dans l'interface pgAdmin
- Gérer bases de données et utilisateurs graphiquement
- Exécuter des requêtes SQL avec l'éditeur intégré
- Sauvegarder et restaurer des bases

**Durée estimée :** 15-20 minutes

---

## 🎯 Objectif

À la fin de cette fiche, vous aurez :
- ✅ pgAdmin fonctionnel et accessible via navigateur
- ✅ PostgreSQL connecté à pgAdmin
- ✅ Une interface moderne pour gérer vos bases
- ✅ La capacité d'exécuter des requêtes visuellement
- ✅ Des outils pour explorer et visualiser vos données

---

## 📦 Prérequis

Avant de commencer :
- ✅ PostgreSQL installé et fonctionnel ([Configuration basique](01-config-basique-docker-compose.md))
- ✅ Docker Compose configuré
- ✅ Un navigateur web moderne

---

## 🧠 Qu'est-ce que pgAdmin ?

### Présentation

**pgAdmin** est l'outil officiel d'administration pour PostgreSQL, développé par la communauté PostgreSQL.

| Aspect | Description |
|--------|-------------|
| **Type** | Interface web (accessible via navigateur) |
| **Langage** | Python (Flask + JavaScript) |
| **Licence** | Open source (PostgreSQL License) |
| **Plateforme** | Multi-plateforme (Windows, macOS, Linux) |
| **Version Docker** | Conteneur officiel pgAdmin 4 |

### Pourquoi Utiliser pgAdmin ?

| Avantage | Description |
|----------|-------------|
| 🖱️ **Interface Graphique** | Plus accessible que la ligne de commande |
| 📊 **Visualisation** | Voir la structure des tables, relations, index |
| ⚡ **Éditeur SQL** | Autocomplétion, coloration syntaxique |
| 📈 **Dashboard** | Monitoring des performances en temps réel |
| 🔧 **Outils intégrés** | Import/Export CSV, backup/restore |
| 🌐 **Accessible partout** | Via navigateur, pas d'installation locale |

### pgAdmin vs Autres Outils

| Outil | Avantage | Inconvénient |
|-------|----------|--------------|
| **pgAdmin** | Officiel, complet, gratuit | Plus lourd, interface dense |
| **DBeaver** | Universel (toutes BDD), léger | Moins spécialisé PostgreSQL |
| **DataGrip** | Professionnel, puissant | Payant |
| **psql** | Rapide, léger | Ligne de commande uniquement |

---

## 🚀 Étape 1 : Configuration Docker Compose

### docker-compose.yml Complet

Modifiez votre `docker-compose.yml` pour ajouter pgAdmin :

```yaml
version: '3.8'

services:
  # PostgreSQL
  postgres:
    image: postgres:15
    container_name: postgres_dev
    restart: unless-stopped

    environment:
      POSTGRES_PASSWORD: changez_moi_123!
      POSTGRES_DB: ma_base
      POSTGRES_USER: mon_utilisateur

    ports:
      - "5432:5432"

    volumes:
      - ./data/postgres:/var/lib/postgresql/data

    networks:
      - postgres_network

  # ✨ pgAdmin
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin_dev
    restart: unless-stopped

    environment:
      # Email de connexion à pgAdmin
      PGADMIN_DEFAULT_EMAIL: admin@example.com

      # Mot de passe de connexion à pgAdmin
      # ⚠️ CHANGEZ CE MOT DE PASSE !
      PGADMIN_DEFAULT_PASSWORD: admin_password_123!

      # Désactiver le mode serveur (plus simple pour dev)
      PGADMIN_CONFIG_SERVER_MODE: 'False'

      # Désactiver la vérification du mot de passe maître
      PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED: 'False'

    ports:
      # pgAdmin sera accessible sur http://localhost:5050
      - "5050:80"

    volumes:
      # Persistance des paramètres pgAdmin
      - ./data/pgadmin:/var/lib/pgadmin

    networks:
      - postgres_network

    # Attendre que PostgreSQL soit prêt
    depends_on:
      - postgres

networks:
  postgres_network:
    driver: bridge
```

### 📖 Explication des Paramètres pgAdmin

| Paramètre | Description | Valeur Recommandée |
|-----------|-------------|-------------------|
| `PGADMIN_DEFAULT_EMAIL` | Email pour se connecter à pgAdmin | Votre email |
| `PGADMIN_DEFAULT_PASSWORD` | Mot de passe pgAdmin | **À changer absolument** |
| `PGADMIN_CONFIG_SERVER_MODE` | Mode multi-utilisateurs | `False` (dev), `True` (prod) |
| `PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED` | Mot de passe maître | `False` (dev) |
| `ports: "5050:80"` | Port d'accès web | `5050` (évite conflit avec port 80) |
| `./data/pgadmin` | Données pgAdmin | Conserve les connexions/préférences |

**⚠️ Sécurité** : Changez `admin@example.com` et `admin_password_123!` par vos propres valeurs.

---

## ▶️ Étape 2 : Lancement de la Stack

### Créer les Dossiers de Données

```bash
# Créer les dossiers pour les volumes
mkdir -p data/postgres data/pgadmin

# Définir les permissions pour pgAdmin (important sur Linux)
sudo chown -R 5050:5050 data/pgadmin
# Ou alternative (moins sécurisé mais fonctionne)
chmod -R 777 data/pgadmin
```

**Note Windows/macOS** : Pas besoin de `chown`, Docker gère automatiquement.

### Démarrer les Services

```bash
# Arrêter l'ancienne configuration (si existante)
docker-compose down

# Démarrer PostgreSQL + pgAdmin
docker-compose up -d
```

**Sortie attendue :**
```
Creating network "postgres-docker_postgres_network" with driver "bridge"
Creating postgres_dev ... done
Creating pgadmin_dev  ... done
```

### Vérifier que Tout Tourne

```bash
# Voir les conteneurs
docker-compose ps
```

**Résultat attendu :**
```
    Name                  Command               State           Ports
--------------------------------------------------------------------------------
pgadmin_dev    /entrypoint.sh                   Up      443/tcp, 0.0.0.0:5050->80/tcp
postgres_dev   docker-entrypoint.sh postgres    Up      0.0.0.0:5432->5432/tcp
```

✅ Les deux services doivent être `Up`.

### Voir les Logs (si Problème)

```bash
# Logs pgAdmin
docker-compose logs -f pgadmin

# Logs PostgreSQL
docker-compose logs -f postgres
```

---

## 🌐 Étape 3 : Accéder à pgAdmin

### Se Connecter à l'Interface

1. **Ouvrez votre navigateur** (Chrome, Firefox, Edge, Safari...)

2. **Accédez à l'URL** : [http://localhost:5050](http://localhost:5050)

3. **Page de connexion** :
   - **Email** : `admin@example.com` (celui défini dans docker-compose.yml)
   - **Password** : `admin_password_123!` (celui défini dans docker-compose.yml)

4. **Cliquez sur "Login"**

![Interface de connexion pgAdmin]

**Si tout fonctionne** : Vous arrivez sur le tableau de bord pgAdmin ! 🎉

### Interface Principale

L'interface pgAdmin est organisée en 3 zones :

```
┌────────────────────────────────────────────────────┐
│  [Menu Bar]  Dashboard | Tools | Help             │
├──────────┬─────────────────────────────────────────┤
│          │                                         │
│  Object  │         Main Panel                      │
│  Browser │      (Tableaux, graphiques, SQL...)     │
│  (Arbre) │                                         │
│          │                                         │
│  Servers │                                         │
│  ├─ DB1  │                                         │
│  ├─ DB2  │                                         │
│  └─ ...  │                                         │
│          │                                         │
└──────────┴─────────────────────────────────────────┘
```

---

## 🔌 Étape 4 : Connecter PostgreSQL à pgAdmin

### Ajouter une Connexion (Serveur)

1. **Clic droit sur "Servers"** (dans l'arbre à gauche)
   - Ou cliquez sur **"Add New Server"**

2. **Onglet "General"** :
   | Champ | Valeur |
   |-------|--------|
   | **Name** | `PostgreSQL Dev` (nom descriptif) |

3. **Onglet "Connection"** :
   | Champ | Valeur | Explication |
   |-------|--------|-------------|
   | **Host name/address** | `postgres` | Nom du service dans docker-compose |
   | **Port** | `5432` | Port par défaut PostgreSQL |
   | **Maintenance database** | `postgres` | Base système |
   | **Username** | `mon_utilisateur` | Défini dans docker-compose |
   | **Password** | `changez_moi_123!` | Défini dans docker-compose |
   | **Save password?** | ✅ Coché | Pour ne pas retaper à chaque fois |

4. **Cliquez sur "Save"**

**⚠️ Important** : Utilisez `postgres` comme hostname (nom du conteneur), **pas** `localhost` ou `127.0.0.1`. Les conteneurs communiquent via leurs noms de service Docker.

### Vérifier la Connexion

Dans l'arbre à gauche, vous devriez voir :

```
Servers
└── PostgreSQL Dev
    └── Databases (X)
        ├── ma_base
        ├── postgres
        ├── template0
        └── template1
```

**Si vous voyez cet arbre** → ✅ Connexion réussie !

**En cas d'erreur** → Voir [Section Dépannage](#-dépannage)

---

## 🎮 Étape 5 : Utiliser pgAdmin

### Naviguer dans l'Interface

#### Explorer une Base de Données

1. **Développez l'arbre** :
   ```
   PostgreSQL Dev
   └── Databases
       └── ma_base
           ├── Schemas
           │   └── public
           │       ├── Tables
           │       ├── Views
           │       ├── Functions
           │       └── ...
           ├── Extensions
           └── ...
   ```

2. **Cliquez sur "Tables"** pour voir vos tables

3. **Clic droit sur une table** → Options :
   - **View/Edit Data** : Voir les données
   - **Properties** : Structure de la table
   - **Maintenance** : VACUUM, ANALYZE
   - **Backup** : Sauvegarder
   - **Drop** : Supprimer (⚠️ dangereux)

### Créer une Base de Données

1. **Clic droit sur "Databases"** → **Create** → **Database...**

2. **Onglet "General"** :
   | Champ | Valeur |
   |-------|--------|
   | **Database** | `blog_db` |
   | **Owner** | `mon_utilisateur` |
   | **Comment** | Description (optionnel) |

3. **Onglet "Definition"** :
   | Champ | Valeur |
   |-------|--------|
   | **Encoding** | `UTF8` |
   | **Template** | `template0` |
   | **Collation** | `en_US.utf8` |

4. **Cliquez sur "Save"**

La nouvelle base apparaît dans l'arbre ! 🎉

### Créer une Table

1. **Développez** : `blog_db` → `Schemas` → `public` → **Clic droit sur "Tables"**

2. **Create** → **Table...**

3. **Onglet "General"** :
   - **Name** : `articles`

4. **Onglet "Columns"** → **Cliquez sur "+"** pour chaque colonne :

   | Name | Data type | Length | Not NULL? | Primary key? |
   |------|-----------|--------|-----------|--------------|
   | `id` | `integer` | - | ✅ | ✅ (via Constraints) |
   | `title` | `character varying` | 200 | ✅ | ❌ |
   | `content` | `text` | - | ❌ | ❌ |
   | `created_at` | `timestamp` | - | ❌ | ❌ |

5. **Onglet "Constraints"** → Ajouter la clé primaire :
   - **Primary Key** sur `id`
   - Ou cocher **"Primary key?"** directement dans l'onglet Columns

6. **Cliquez sur "Save"**

### Insérer des Données Visuellement

1. **Clic droit sur votre table** → **View/Edit Data** → **All Rows**

2. Une grille s'affiche (vide au début)

3. **Cliquez sur la dernière ligne** (icône `+`)

4. **Remplissez les champs** :
   - `id` : 1
   - `title` : Mon premier article
   - `content` : Ceci est le contenu...
   - `created_at` : 2024-10-29 14:30:00

5. **Cliquez sur "Save Data Changes"** (icône disquette)

6. Les données sont insérées !

**Alternative (plus rapide)** : Utilisez l'éditeur SQL (voir section suivante).

### Exécuter des Requêtes SQL

1. **Clic droit sur `blog_db`** → **Query Tool**

2. **Éditeur SQL s'ouvre** avec autocomplétion

3. **Tapez votre requête** :
   ```sql
   -- Créer une table
   CREATE TABLE utilisateurs (
       id SERIAL PRIMARY KEY,
       nom VARCHAR(100) NOT NULL,
       email VARCHAR(100) UNIQUE NOT NULL,
       created_at TIMESTAMP DEFAULT NOW()
   );

   -- Insérer des données
   INSERT INTO utilisateurs (nom, email) VALUES
       ('Alice', 'alice@example.com'),
       ('Bob', 'bob@example.com'),
       ('Charlie', 'charlie@example.com');

   -- Sélectionner
   SELECT * FROM utilisateurs;
   ```

4. **Exécutez** :
   - **F5** ou **Icône ▶️** : Exécute toute la requête
   - **Sélection + F5** : Exécute uniquement la partie sélectionnée

5. **Résultats s'affichent** dans l'onglet "Data Output"

**Raccourcis utiles** :
- `Ctrl + Space` : Autocomplétion
- `Ctrl + /` : Commenter/décommenter
- `F7` : Formater le SQL (indentation)
- `F5` : Exécuter
- `Ctrl + S` : Sauvegarder la requête

### Visualiser les Données

1. **Clic droit sur une table** → **View/Edit Data** → **All Rows**

2. **Fonctionnalités** :
   - 🔍 **Filtrer** : Cliquez sur l'icône filtre
   - 📊 **Trier** : Cliquez sur les en-têtes de colonnes
   - ✏️ **Modifier** : Double-cliquez sur une cellule
   - ➕ **Ajouter** : Cliquez sur la dernière ligne
   - ❌ **Supprimer** : Sélectionnez ligne + icône poubelle

3. **Export** :
   - **Icône téléchargement** : Export CSV, JSON, etc.

### Gérer les Utilisateurs (Rôles)

1. **Développez** : `PostgreSQL Dev` → **Login/Group Roles**

2. **Clic droit** → **Create** → **Login/Group Role...**

3. **Onglet "General"** :
   - **Name** : `blog_app`

4. **Onglet "Definition"** :
   - **Password** : `app_password_123!`

5. **Onglet "Privileges"** :
   - **Can login?** : ✅ Oui
   - **Superuser?** : ❌ Non
   - **Create databases?** : Selon besoin

6. **Cliquez sur "Save"**

7. **Accorder des permissions** :
   - Allez sur la base `blog_db`
   - Onglet "Security" → Cliquez sur "+"
   - Sélectionnez `blog_app`
   - Cochez les privilèges : `CONNECT`, `TEMPORARY`, `CREATE`

### Dashboard et Monitoring

1. **Cliquez sur votre serveur** dans l'arbre

2. **Onglet "Dashboard"** affiche :
   - 📊 Activité serveur (connexions, transactions)
   - 💾 Utilisation disque
   - 🔄 Activité des sessions
   - 📈 Graphiques temps réel

3. **Onglet "Statistics"** :
   - Détails par base de données
   - Nombre de connexions
   - Taille des tables

4. **Utile pour** :
   - Surveiller les performances
   - Identifier les requêtes lentes
   - Voir les connexions actives

---

## 💾 Étape 6 : Backup et Restore

### Sauvegarder une Base de Données

1. **Clic droit sur `blog_db`** → **Backup...**

2. **Onglet "General"** :
   | Champ | Valeur |
   |-------|--------|
   | **Filename** | `/tmp/blog_db_backup.sql` |
   | **Format** | Plain (SQL), Custom, Directory, Tar |
   | **Encoding** | UTF8 |

3. **Onglet "Data/Objects"** :
   - **Blobs** : ✅ (si vous avez des fichiers binaires)
   - **Only data** : Pour sauvegarder uniquement les données
   - **Only schema** : Pour sauvegarder uniquement la structure

4. **Cliquez sur "Backup"**

5. **Télécharger le fichier** depuis le conteneur :
   ```bash
   docker cp pgadmin_dev:/tmp/blog_db_backup.sql ./backup_blog_db.sql
   ```

### Restaurer une Base de Données

1. **Créez une nouvelle base** (si nécessaire) : `blog_db_restored`

2. **Clic droit sur `blog_db_restored`** → **Restore...**

3. **Onglet "General"** :
   | Champ | Valeur |
   |-------|--------|
   | **Filename** | `/tmp/blog_db_backup.sql` (uploader d'abord) |
   | **Format** | Custom (selon format de sauvegarde) |

4. **Uploader le backup dans le conteneur** :
   ```bash
   docker cp ./backup_blog_db.sql pgadmin_dev:/tmp/blog_db_backup.sql
   ```

5. **Dans pgAdmin** → **Restore...**

6. **Cliquez sur "Restore"**

7. La base est restaurée !

**Alternative (plus rapide via psql)** :
```bash
# Backup
docker exec postgres_dev pg_dump -U mon_utilisateur blog_db > backup.sql

# Restore
docker exec -i postgres_dev psql -U mon_utilisateur -d blog_db_restored < backup.sql
```

---

## 🛠️ Configuration Avancée

### Connexions Multiples

Vous pouvez connecter pgAdmin à plusieurs serveurs PostgreSQL :

```yaml
services:
  postgres_dev:
    # ... (comme avant)

  postgres_test:
    image: postgres:15
    container_name: postgres_test
    environment:
      POSTGRES_PASSWORD: test_password
    ports:
      - "5433:5432"
    networks:
      - postgres_network
```

Dans pgAdmin, ajoutez un deuxième serveur avec `postgres_test` comme hostname.

### Sauvegarder les Connexions

Les connexions pgAdmin sont sauvegardées dans `./data/pgadmin/`. Tant que ce dossier existe, vos connexions sont préservées.

### Mode Serveur (Multi-Utilisateurs)

Pour un usage en équipe :

```yaml
environment:
  PGADMIN_CONFIG_SERVER_MODE: 'True'
  PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED: 'True'
```

Chaque utilisateur aura son propre compte pgAdmin.

### Personnalisation de l'Interface

1. **File** → **Preferences**

2. **Options disponibles** :
   - **Browser** : Thème clair/sombre
   - **Query Tool** : Taille police, thème éditeur
   - **Display** : Langue (dont français)
   - **Miscellaneous** : Diverses options

3. **Thème sombre** :
   - **File** → **Preferences** → **Miscellaneous** → **Themes**
   - Sélectionnez "Dark"

---

## 🔍 Dépannage

### Erreur : "Unable to connect to server"

**Causes possibles** :

1. **PostgreSQL pas démarré** :
   ```bash
   docker ps
   # Vérifiez que postgres_dev est UP
   ```

2. **Mauvais hostname** :
   - Utilisez `postgres` (nom du service), pas `localhost`

3. **Mauvais identifiants** :
   - Vérifiez `POSTGRES_USER` et `POSTGRES_PASSWORD` dans docker-compose.yml

4. **Pare-feu/Réseau** :
   - Les deux conteneurs doivent être sur le même réseau Docker

**Solution** :
```bash
# Voir les logs PostgreSQL
docker-compose logs postgres

# Voir les logs pgAdmin
docker-compose logs pgadmin

# Tester la connexion depuis pgAdmin
docker exec -it pgadmin_dev ping postgres
```

### Erreur : "Permission denied" sur ./data/pgadmin

**Cause** : Problème de permissions (Linux).

**Solution** :
```bash
# Option 1 : Changer le propriétaire
sudo chown -R 5050:5050 data/pgadmin

# Option 2 : Permissions larges (moins sécurisé)
chmod -R 777 data/pgadmin

# Redémarrer
docker-compose restart pgadmin
```

### pgAdmin Très Lent au Démarrage

**Cause** : pgAdmin met 10-20 secondes à démarrer (normal).

**Solution** : Patience ! Surveillez les logs :
```bash
docker-compose logs -f pgadmin
# Attendez "Listening at: http://0.0.0.0:80"
```

### Erreur : "Password authentication failed"

**Cause** : Identifiants incorrects.

**Solution** :
1. Vérifiez dans docker-compose.yml :
   - `POSTGRES_USER`
   - `POSTGRES_PASSWORD`

2. Dans pgAdmin, lors de l'ajout du serveur, utilisez exactement ces valeurs

3. Si vous avez changé le mot de passe :
   ```bash
   docker-compose down
   rm -rf data/postgres
   docker-compose up -d
   ```

### Interface pgAdmin en Anglais (vouloir Français)

**Solution** :
1. **File** → **Preferences**
2. **Miscellaneous** → **User Language**
3. Sélectionnez **"French"**
4. Rafraîchissez la page (F5)

---

## 📊 Comparaison : pgAdmin vs Ligne de Commande

| Tâche | pgAdmin | psql (CLI) | Recommandation |
|-------|---------|------------|----------------|
| **Explorer la structure** | ⭐⭐⭐⭐⭐ Visuel | ⭐⭐ `\dt`, `\d table` | pgAdmin |
| **Créer des tables** | ⭐⭐⭐⭐ Formulaires | ⭐⭐⭐⭐⭐ CREATE TABLE | Selon préférence |
| **Requêtes simples** | ⭐⭐⭐⭐ Éditeur | ⭐⭐⭐⭐⭐ Rapide | psql |
| **Requêtes complexes** | ⭐⭐⭐⭐⭐ Autocomplétion | ⭐⭐⭐ Basique | pgAdmin |
| **Voir les données** | ⭐⭐⭐⭐⭐ Grille | ⭐⭐⭐ SELECT | pgAdmin |
| **Backup/Restore** | ⭐⭐⭐⭐ GUI | ⭐⭐⭐⭐⭐ Scripts | Selon contexte |
| **Performance** | ⭐⭐⭐ Plus lourd | ⭐⭐⭐⭐⭐ Léger | psql |
| **Débutants** | ⭐⭐⭐⭐⭐ Très accessible | ⭐⭐ Intimidant | pgAdmin |

**Verdict** : Utilisez les deux ! pgAdmin pour explorer et visualiser, psql pour les opérations rapides et scripts.

---

## ✅ Checklist de Configuration

Avant de considérer votre configuration pgAdmin prête :

### Installation
- [ ] pgAdmin démarré (`docker ps` montre le conteneur)
- [ ] Accessible sur http://localhost:5050
- [ ] Connexion à l'interface réussie

### Connexion PostgreSQL
- [ ] Serveur PostgreSQL ajouté dans pgAdmin
- [ ] Connexion réussie (arbre des bases visible)
- [ ] Toutes les bases de données apparaissent

### Fonctionnalités
- [ ] Query Tool fonctionne (exécution de requêtes)
- [ ] View/Edit Data fonctionne (visualisation)
- [ ] Backup fonctionne (sauvegarde testée)
- [ ] Permissions correctes sur ./data/pgadmin

### Sécurité
- [ ] Mot de passe pgAdmin changé (pas admin)
- [ ] Mot de passe PostgreSQL fort
- [ ] Données dans ./data (persistance)
- [ ] Port 5050 non exposé publiquement (dev uniquement)

---

## 🚀 Prochaines Étapes

Maintenant que pgAdmin est configuré :

1. **Explorer les annexes** :
   - [Annexe A - Référence des commandes](/annexes/A-reference-commandes.md)
   - [Annexe F - Outils de gestion](/annexes/F-outils-gestion.md)

2. **Déployer des stacks complètes** :
   - [Cas Pratiques](/cas-pratiques/)

3. **Découvrir d'autres bases de données** :
   - [MariaDB](/01-mariadb/)
   - [MongoDB](/05-mongodb/)
   - [Redis](/06-redis/)

4. **Approfondir PostgreSQL** :
   - [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
   - [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## 📚 Ressources Complémentaires

### Documentation Officielle
- **pgAdmin Documentation** : [https://www.pgadmin.org/docs/](https://www.pgadmin.org/docs/)
- **pgAdmin Docker** : [https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html)
- **pgAdmin GitHub** : [https://github.com/pgadmin-org/pgadmin4](https://github.com/pgadmin-org/pgadmin4)

### Tutoriels Vidéo
- **pgAdmin 4 Tutorial** (YouTube)
- **PostgreSQL + pgAdmin Complete Guide** (Udemy, Coursera)

### Alternatives à pgAdmin
- **DBeaver** (universel, gratuit) : [https://dbeaver.io/](https://dbeaver.io/)
- **DataGrip** (JetBrains, payant) : [https://www.jetbrains.com/datagrip/](https://www.jetbrains.com/datagrip/)
- **TablePlus** (macOS/Windows, freemium) : [https://tableplus.com/](https://tableplus.com/)
- **Adminer** (PHP, ultra léger) : [https://www.adminer.org/](https://www.adminer.org/)

---

## 🎓 Récapitulatif

**Ce que vous avez appris :**

✅ Déployer pgAdmin avec Docker Compose
✅ Connecter pgAdmin à PostgreSQL
✅ Naviguer dans l'interface graphique
✅ Créer bases de données et tables visuellement
✅ Exécuter des requêtes SQL avec l'éditeur intégré
✅ Gérer les utilisateurs et permissions
✅ Sauvegarder et restaurer des bases

🔝 Retour au [Sommaire](/SOMMAIRE.md)
