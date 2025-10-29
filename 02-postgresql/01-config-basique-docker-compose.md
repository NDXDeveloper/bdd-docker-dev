# PostgreSQL - Configuration Basique avec Docker Compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

PostgreSQL est un système de gestion de base de données relationnelle (SGBD) open source très populaire, réputé pour sa fiabilité, sa robustesse et ses fonctionnalités avancées. Cette fiche vous guide pour déployer PostgreSQL avec Docker en quelques minutes.

**Ce que vous allez apprendre :**
- Démarrer PostgreSQL avec Docker Compose
- Se connecter à votre base de données
- Créer votre première base et votre premier utilisateur
- Gérer le conteneur (démarrer, arrêter, supprimer)

**Durée estimée :** 10-15 minutes

---

## 🎯 Objectif

À la fin de cette fiche, vous aurez :
- ✅ Un serveur PostgreSQL fonctionnel
- ✅ Une base de données persistante (les données survivent aux redémarrages)
- ✅ La capacité de vous connecter depuis votre PC
- ✅ Les commandes essentielles pour gérer votre conteneur

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :
- ✅ Docker installé et fonctionnel ([voir fiche Prérequis](/00-introduction/01-prerequis.md))
- ✅ Docker Compose installé
- ✅ Un éditeur de texte (VS Code, Sublime Text, ou autre)
- ✅ 10 minutes devant vous

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🚀 Étape 1 : Préparation du Projet

### Créer la Structure de Dossiers

```bash
# Créer un dossier pour votre projet
mkdir postgres-docker
cd postgres-docker

# Créer le dossier pour les données
mkdir data
```

**Structure obtenue :**
```
postgres-docker/
├── data/                  # Stockera les données PostgreSQL
└── docker-compose.yml     # À créer à l'étape suivante
```

---

## 📝 Étape 2 : Créer le Fichier docker-compose.yml

Créez un fichier `docker-compose.yml` dans le dossier `postgres-docker` avec le contenu suivant :

```yaml
version: '3.8'

services:
  postgres:
    # Image officielle PostgreSQL version 15
    image: postgres:15

    # Nom du conteneur (plus facile à identifier)
    container_name: postgres_dev

    # Redémarrage automatique sauf si arrêt manuel
    restart: unless-stopped

    # Variables d'environnement pour la configuration
    environment:
      # Mot de passe du super-utilisateur 'postgres'
      # ⚠️ CHANGEZ CE MOT DE PASSE !
      POSTGRES_PASSWORD: changez_moi_123!

      # Créer automatiquement une base de données au démarrage
      POSTGRES_DB: ma_base

      # Créer automatiquement un utilisateur
      POSTGRES_USER: mon_utilisateur

    # Exposition du port PostgreSQL
    ports:
      # Format: "port_sur_votre_PC:port_dans_le_conteneur"
      - "5432:5432"

    # Volume pour persister les données
    volumes:
      # Les données PostgreSQL seront stockées dans ./data
      - ./data:/var/lib/postgresql/data

# Note: Pas besoin de déclarer le volume car on utilise un bind mount (./data)
```

### 📖 Explication des Paramètres

| Paramètre | Explication | Valeur Recommandée |
|-----------|-------------|-------------------|
| `image: postgres:15` | Version de PostgreSQL à utiliser | `postgres:15` (stable) |
| `container_name` | Nom du conteneur | Descriptif (ex: `mon_projet_postgres`) |
| `POSTGRES_PASSWORD` | Mot de passe du super-utilisateur | ⚠️ **À CHANGER ABSOLUMENT** |
| `POSTGRES_DB` | Nom de la base créée automatiquement | Nom de votre projet |
| `POSTGRES_USER` | Utilisateur créé automatiquement | Nom descriptif |
| `ports: "5432:5432"` | Port d'accès | `5432` (port par défaut de PostgreSQL) |
| `./data:/var/lib/postgresql/data` | Stockage des données | `./data` (dans votre dossier) |

### 🔐 Important : Sécurité du Mot de Passe

**Pour cette configuration basique**, le mot de passe est directement dans le fichier. C'est acceptable pour débuter, mais :

⚠️ **NE JAMAIS** commiter ce fichier dans Git avec un vrai mot de passe
✅ Pour un usage sérieux, utilisez un fichier `.env` (voir [Bonnes Pratiques](/00-introduction/03-bonnes-pratiques.md))

**Exemple de mot de passe acceptable :**
```yaml
POSTGRES_PASSWORD: Dev2024!PostgreSQL#Secure
```

---

## ▶️ Étape 3 : Lancer PostgreSQL

Dans le dossier `postgres-docker`, exécutez :

```bash
docker-compose up -d
```

**Que fait cette commande ?**
- `up` : Démarre les services définis dans `docker-compose.yml`
- `-d` : Mode "detached" (tourne en arrière-plan)

**Ce qui se passe :**
1. Docker télécharge l'image PostgreSQL 15 (première fois uniquement, ~80 MB)
2. Docker crée le conteneur `postgres_dev`
3. PostgreSQL démarre et initialise la base de données
4. La base `ma_base` et l'utilisateur `mon_utilisateur` sont créés automatiquement

**Sortie attendue :**
```
Creating network "postgres-docker_default" with the default driver
Creating postgres_dev ... done
```

### Vérifier que le Conteneur Tourne

```bash
docker ps
```

**Vous devriez voir :**
```
CONTAINER ID   IMAGE         COMMAND                  STATUS         PORTS                    NAMES
a1b2c3d4e5f6   postgres:15   "docker-entrypoint.s…"   Up 10 seconds  0.0.0.0:5432->5432/tcp   postgres_dev
```

Si vous voyez `Up X seconds` ou `Up X minutes`, c'est bon ! 🎉

---

## 🔌 Étape 4 : Se Connecter à PostgreSQL

### Méthode 1 : Ligne de Commande (psql)

Connectez-vous directement depuis le terminal :

```bash
docker exec -it postgres_dev psql -U mon_utilisateur -d ma_base
```

**Explication :**
- `docker exec` : Exécute une commande dans le conteneur
- `-it` : Mode interactif (vous pouvez taper des commandes)
- `postgres_dev` : Nom de votre conteneur
- `psql` : Client PostgreSQL en ligne de commande
- `-U mon_utilisateur` : Nom d'utilisateur
- `-d ma_base` : Nom de la base de données

**Invite de commande attendue :**
```
ma_base=#
```

**Testez quelques commandes SQL :**
```sql
-- Voir les bases de données
\l

-- Créer une table
CREATE TABLE utilisateurs (
    id SERIAL PRIMARY KEY,
    nom VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Insérer des données
INSERT INTO utilisateurs (nom, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com');

-- Voir les données
SELECT * FROM utilisateurs;

-- Quitter psql
\q
```

### Méthode 2 : Client Graphique (Recommandé pour Débutants)

Utilisez un client GUI comme **DBeaver**, **pgAdmin**, ou **DataGrip**.

#### Exemple avec DBeaver (Gratuit)

1. **Téléchargez DBeaver** : [https://dbeaver.io/download/](https://dbeaver.io/download/)

2. **Créez une nouvelle connexion** :
   - Cliquez sur "Nouvelle connexion" (icône prise électrique)
   - Choisissez "PostgreSQL"

3. **Configurez la connexion** :
   | Paramètre | Valeur |
   |-----------|--------|
   | **Hôte** | `localhost` ou `127.0.0.1` |
   | **Port** | `5432` |
   | **Base de données** | `ma_base` |
   | **Nom d'utilisateur** | `mon_utilisateur` |
   | **Mot de passe** | Celui défini dans `docker-compose.yml` |

4. **Testez la connexion** : Cliquez sur "Test Connection"
   - Si succès : Vous verrez "Connected" ✅
   - Si échec : Vérifiez que le conteneur tourne (`docker ps`)

5. **Utilisez votre base** : Vous pouvez maintenant exécuter des requêtes SQL dans l'interface graphique

---

## 🎮 Étape 5 : Gérer le Conteneur

### Commandes Essentielles

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Arrêter PostgreSQL (données conservées)
docker-compose stop

# Redémarrer PostgreSQL
docker-compose start

# Voir le statut
docker-compose ps

# Entrer dans le conteneur (shell bash)
docker exec -it postgres_dev bash

# Arrêter ET supprimer le conteneur (données conservées dans ./data)
docker-compose down
```

### Tableau Récapitulatif

| Commande | Action | Données Affectées ? |
|----------|--------|-------------------|
| `docker-compose stop` | Arrête le conteneur | ❌ Non (conservées) |
| `docker-compose start` | Redémarre le conteneur | ❌ Non |
| `docker-compose restart` | Redémarre rapidement | ❌ Non |
| `docker-compose down` | Supprime le conteneur | ❌ Non (dans `./data`) |
| `docker-compose down -v` | Supprime conteneur + volumes | ⚠️ **OUI (perte totale)** |
| Supprimer `./data/` | Supprime les fichiers de données | ⚠️ **OUI (perte totale)** |

---

## 🧹 Étape 6 : Nettoyage Complet (Optionnel)

Si vous voulez **tout supprimer** (conteneur + données) :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer les données (ATTENTION : IRRÉVERSIBLE)
rm -rf ./data

# 3. (Optionnel) Supprimer l'image PostgreSQL
docker rmi postgres:15
```

⚠️ **ATTENTION** : Après ces commandes, **toutes vos données sont perdues** !

---

## ✅ Vérification Finale

Checklist pour vérifier que tout fonctionne :

- [ ] Le conteneur est en cours d'exécution (`docker ps` montre `postgres_dev`)
- [ ] Vous pouvez vous connecter avec `psql` depuis le terminal
- [ ] Vous pouvez vous connecter avec un client GUI (DBeaver, pgAdmin...)
- [ ] Vous pouvez créer des tables et insérer des données
- [ ] Après un `docker-compose stop` puis `start`, vos données sont toujours là
- [ ] Le dossier `./data` contient des fichiers (données PostgreSQL)

---

## 🔍 Dépannage

### Problème : "Port 5432 already in use"

**Cause** : Un autre service PostgreSQL (ou autre) utilise déjà le port 5432.

**Solution 1** : Arrêter l'autre service
```bash
# Voir qui utilise le port (Linux/macOS)
sudo lsof -i :5432

# Windows
netstat -ano | findstr :5432
```

**Solution 2** : Changer le port dans `docker-compose.yml`
```yaml
ports:
  - "5433:5432"  # Utilisez 5433 sur votre PC au lieu de 5432
```

Puis connectez-vous sur le port 5433.

### Problème : "Connection refused"

**Cause** : Le conteneur n'est pas démarré ou PostgreSQL n'a pas fini d'initialiser.

**Solutions** :
```bash
# Vérifier que le conteneur tourne
docker ps

# Voir les logs pour détecter les erreurs
docker-compose logs

# Attendre quelques secondes (PostgreSQL met ~10-20s à démarrer la première fois)
```

### Problème : "Authentication failed"

**Cause** : Mauvais mot de passe ou mauvais utilisateur.

**Solutions** :
1. Vérifiez les identifiants dans `docker-compose.yml`
2. Si vous avez changé le mot de passe après le premier démarrage, il faut recréer la base :
   ```bash
   docker-compose down
   rm -rf ./data
   docker-compose up -d
   ```

### Problème : Données perdues après redémarrage

**Cause** : Le volume n'est pas correctement configuré.

**Solution** : Vérifiez que cette ligne est bien présente dans `docker-compose.yml` :
```yaml
volumes:
  - ./data:/var/lib/postgresql/data
```

Et que le dossier `./data` existe et contient des fichiers.

---

## 📊 Informations Complémentaires

### Versions de PostgreSQL Disponibles

| Version | Statut | Recommandation |
|---------|--------|----------------|
| `postgres:17` | Dernière | Fonctionnalités récentes, moins testé |
| `postgres:16` | Stable | Excellent choix |
| `postgres:15` | Stable | **Recommandé** (équilibre stabilité/fonctionnalités) |
| `postgres:14` | Maintenance | Toujours supporté |
| `postgres:13` | Maintenance | Fin de support en 2025 |
| `postgres:latest` | Variable | ❌ À éviter (version imprévisible) |

**Choisir une version :**
- Pour débuter : `postgres:15`
- Pour production : La version utilisée par votre hébergeur
- Pour apprendre les nouveautés : `postgres:16` ou `postgres:17`

### Consommation de Ressources

| Ressource | Minimum | Recommandé | Production |
|-----------|---------|------------|-----------|
| **RAM** | 256 MB | 512 MB - 1 GB | 2-4 GB+ |
| **CPU** | 1 core | 2 cores | 4+ cores |
| **Disque** | 100 MB | 1-10 GB | 50-500 GB+ |

**Note** : Ces valeurs sont pour le développement. La production nécessite bien plus de ressources selon la charge.

### Commandes PostgreSQL Utiles (psql)

Une fois connecté avec `psql`, ces commandes sont très utiles :

```sql
-- Commandes méta (avec \)
\l              -- Lister les bases de données
\c ma_base      -- Se connecter à une base
\dt             -- Lister les tables
\d nom_table    -- Décrire une table
\du             -- Lister les utilisateurs
\q              -- Quitter

-- Commandes SQL classiques
SELECT version();                    -- Version de PostgreSQL
SELECT current_database();           -- Base actuelle
SELECT current_user;                 -- Utilisateur actuel
SHOW ALL;                            -- Toutes les configurations
```

---

## 🎓 Concepts Clés à Retenir

### PostgreSQL vs MariaDB/MySQL

| Aspect | PostgreSQL | MariaDB/MySQL |
|--------|------------|---------------|
| **Standards SQL** | Très strict (conforme ANSI) | Plus permissif |
| **Types de données** | Plus riches (JSON, Arrays, UUID...) | Standards |
| **Performances** | Excellent en lecture complexe | Excellent en écriture simple |
| **Extensions** | Riche écosystème (PostGIS...) | Moins d'extensions |
| **Cas d'usage** | Applications complexes, analytics | Applications web, CMS |

### Utilisateurs PostgreSQL

PostgreSQL crée automatiquement :
1. **Utilisateur `postgres`** : Super-utilisateur par défaut (comme `root` sur MySQL)
2. **Votre utilisateur personnalisé** : Défini par `POSTGRES_USER`

**Hiérarchie des privilèges :**
```
postgres (super-admin)
    └── mon_utilisateur (admin de ma_base)
            └── Autres utilisateurs (à créer manuellement)
```

### Bases de Données PostgreSQL

Chaque serveur PostgreSQL peut contenir plusieurs bases de données, complètement isolées les unes des autres.

**Bases créées automatiquement :**
- `postgres` : Base système (ne pas supprimer)
- `template0` : Template de base (ne pas modifier)
- `template1` : Template pour nouvelles bases
- `ma_base` : Votre base (définie par `POSTGRES_DB`)

---

## 🚀 Prochaines Étapes

Maintenant que votre PostgreSQL fonctionne, vous pouvez :

1. **Approfondir la configuration** → [Configuration avec postgresql.conf](02-config-avec-postgresqlconf.md)
2. **Configurer une IP fixe** → [Configuration IP fixe](03-config-ip-fixe.md)
3. **Gérer les utilisateurs** → [Gestion des utilisateurs et BDD](04-gestion-utilisateurs-bdd.md)
4. **Ajouter une interface graphique** → [Configuration avec pgAdmin](05-config-avec-pgadmin.md)
5. **Apprendre les bonnes pratiques** → [Bonnes Pratiques](/00-introduction/03-bonnes-pratiques.md)

---

## 📚 Ressources Complémentaires

### Documentation Officielle
- **PostgreSQL Documentation** : [https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)
- **Docker Hub PostgreSQL** : [https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres)
- **PostgreSQL Tutorial** : [https://www.postgresqltutorial.com/](https://www.postgresqltutorial.com/)

### Outils Recommandés
- **DBeaver** (client universel gratuit) : [https://dbeaver.io/](https://dbeaver.io/)
- **pgAdmin** (client officiel PostgreSQL) : [https://www.pgadmin.org/](https://www.pgadmin.org/)
- **TablePlus** (macOS/Windows, freemium) : [https://tableplus.com/](https://tableplus.com/)

### Formation et Tutoriels
- **PostgreSQL Exercises** : [https://pgexercises.com/](https://pgexercises.com/)
- **PostgreSQL Wiki** : [https://wiki.postgresql.org/](https://wiki.postgresql.org/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
