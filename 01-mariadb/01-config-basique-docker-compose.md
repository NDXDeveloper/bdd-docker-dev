# Configuration basique de MariaDB avec Docker Compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous guide pas à pas pour déployer une instance MariaDB fonctionnelle avec Docker Compose. MariaDB est une base de données relationnelle (SQL) compatible avec MySQL, idéale pour le développement d'applications web.

**Ce que vous allez apprendre :**
- Créer un fichier `docker-compose.yml` pour MariaDB
- Démarrer et arrêter votre base de données
- Vous connecter à MariaDB depuis votre machine
- Gérer la persistance des données

**Durée estimée :** 10 minutes

---

## 🎯 Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ Docker installé sur votre machine ([Guide d'installation](https://docs.docker.com/get-docker/))
- ✅ Docker Compose installé (inclus avec Docker Desktop)
- ✅ Un éditeur de texte (VS Code, Notepad++, nano, vim...)
- ✅ Des connaissances basiques du terminal/ligne de commande

**Vérification rapide :**

```bash
# Vérifier que Docker fonctionne
docker --version

# Vérifier Docker Compose
docker-compose --version
```

Si ces commandes affichent un numéro de version, vous êtes prêt !

---

## 📝 Étape 1 : Créer le fichier docker-compose.yml

### 1.1 Créer un dossier pour votre projet

Créez un nouveau dossier pour organiser vos fichiers :

```bash
# Créer le dossier
mkdir mariadb-docker

# Se déplacer dans le dossier
cd mariadb-docker
```

### 1.2 Créer le fichier de configuration

Créez un fichier nommé `docker-compose.yml` dans ce dossier avec le contenu suivant :

```yaml
version: '3.8'

services:
  mariadb:
    # Image officielle MariaDB (version 10.11)
    image: mariadb:10.11

    # Nom du conteneur (facilite les commandes)
    container_name: mariadb_dev

    # Redémarrage automatique sauf si arrêt manuel
    restart: unless-stopped

    # Variables d'environnement
    environment:
      # ⚠️ IMPORTANT : Changez ce mot de passe !
      MYSQL_ROOT_PASSWORD: mon_mot_de_passe_securise

      # Optionnel : Créer une base de données au démarrage
      MYSQL_DATABASE: ma_base_dev

      # Optionnel : Créer un utilisateur (en plus de root)
      MYSQL_USER: dev_user
      MYSQL_PASSWORD: mot_de_passe_user

    # Exposition du port
    ports:
      # Format : port_hote:port_conteneur
      - "3306:3306"

    # Volume pour la persistance des données
    volumes:
      - mariadb_data:/var/lib/mysql

# Déclaration du volume
volumes:
  mariadb_data:
    # Volume géré par Docker (données persistantes)
```

### 📖 Explication du fichier

| Section | Rôle |
|---------|------|
| `version: '3.8'` | Version du format Docker Compose |
| `services:` | Liste des conteneurs à créer |
| `image: mariadb:10.11` | Image Docker à utiliser (version 10.11 de MariaDB) |
| `container_name:` | Nom personnalisé du conteneur |
| `restart: unless-stopped` | Le conteneur redémarre automatiquement (sauf si vous l'arrêtez) |
| `environment:` | Variables de configuration |
| `ports:` | Correspondance des ports (permet l'accès depuis votre PC) |
| `volumes:` | Stockage persistant des données |

**💡 Comprendre les variables d'environnement :**

- `MYSQL_ROOT_PASSWORD` : Mot de passe du super-administrateur (obligatoire)
- `MYSQL_DATABASE` : Nom d'une base de données créée automatiquement (optionnel)
- `MYSQL_USER` : Nom d'un utilisateur supplémentaire (optionnel)
- `MYSQL_PASSWORD` : Mot de passe de cet utilisateur (optionnel)

---

## ▶️ Étape 2 : Démarrer MariaDB

### 2.1 Lancer le conteneur

Dans votre terminal, depuis le dossier contenant le `docker-compose.yml`, exécutez :

```bash
docker-compose up -d
```

**Signification de la commande :**
- `up` : Créer et démarrer les conteneurs
- `-d` : Mode "detached" (en arrière-plan)

**Ce qui se passe :**
1. Docker télécharge l'image MariaDB (si première utilisation)
2. Création du conteneur `mariadb_dev`
3. Création du volume `mariadb_data`
4. Démarrage de MariaDB

### 2.2 Vérifier que tout fonctionne

```bash
# Voir les conteneurs en cours d'exécution
docker-compose ps

# Vous devriez voir quelque chose comme :
#     Name                Command             State          Ports
# --------------------------------------------------------------------
# mariadb_dev   docker-entrypoint.sh ...   Up      0.0.0.0:3306->3306/tcp
```

Si le `State` est `Up`, félicitations, votre base de données est lancée ! 🎉

---

## 🔌 Étape 3 : Se connecter à MariaDB

### 3.1 Connexion depuis le terminal

Vous pouvez vous connecter directement au shell MariaDB du conteneur :

```bash
# Se connecter en tant que root
docker exec -it mariadb_dev mariadb -u root -p
```

**Détail de la commande :**
- `docker exec` : Exécuter une commande dans un conteneur
- `-it` : Mode interactif avec terminal
- `mariadb_dev` : Nom du conteneur
- `mariadb -u root -p` : Commande MariaDB (se connecter en tant que root)

Entrez le mot de passe (`MYSQL_ROOT_PASSWORD` défini dans votre `docker-compose.yml`).

**Une fois connecté, vous pouvez tester :**

```sql
-- Afficher les bases de données
SHOW DATABASES;

-- Utiliser votre base de données (si créée)
USE ma_base_dev;

-- Créer une table de test
CREATE TABLE test (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100)
);

-- Insérer des données
INSERT INTO test (nom) VALUES ('Premier test');

-- Afficher les données
SELECT * FROM test;

-- Quitter
EXIT;
```

### 3.2 Connexion depuis un client graphique

Vous pouvez utiliser des outils comme :
- **DBeaver** (gratuit, multi-plateformes)
- **HeidiSQL** (gratuit, Windows)
- **MySQL Workbench** (gratuit, multi-plateformes)
- **DataGrip** (payant, JetBrains)

**Paramètres de connexion :**

| Paramètre | Valeur |
|-----------|--------|
| Hôte | `localhost` ou `127.0.0.1` |
| Port | `3306` |
| Utilisateur | `root` (ou `dev_user` si créé) |
| Mot de passe | Celui défini dans `docker-compose.yml` |
| Base de données | `ma_base_dev` (ou autre) |

---

## 💾 Étape 4 : Comprendre la persistance des données

### 4.1 Qu'est-ce qu'un volume Docker ?

Les volumes Docker permettent de **sauvegarder les données** même quand le conteneur est supprimé.

**Dans notre configuration :**
```yaml
volumes:
  - mariadb_data:/var/lib/mysql
```

- `mariadb_data` : Nom du volume (géré par Docker)
- `/var/lib/mysql` : Emplacement des données dans le conteneur

### 4.2 Vérifier les volumes

```bash
# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect mariadb_data
```

### 4.3 Où sont stockées les données ?

Les données sont stockées dans un emplacement géré par Docker :
- **Linux :** `/var/lib/docker/volumes/`
- **Windows/Mac (Docker Desktop) :** Dans la VM de Docker Desktop

**💡 Important :** Vous n'avez pas besoin d'accéder directement à cet emplacement. Docker gère tout pour vous !

---

## 🛑 Étape 5 : Gérer le conteneur

### 5.1 Commandes utiles

```bash
# Voir les logs en temps réel
docker-compose logs -f

# Arrêter le conteneur (les données sont conservées)
docker-compose stop

# Redémarrer le conteneur
docker-compose start

# Redémarrer après modification du docker-compose.yml
docker-compose restart

# Arrêter ET supprimer le conteneur (les données restent dans le volume)
docker-compose down

# Supprimer conteneur ET volume (⚠️ SUPPRIME TOUTES LES DONNÉES)
docker-compose down -v
```

### 5.2 Cycle de vie typique

```bash
# Premier lancement
docker-compose up -d

# Travailler avec la BDD...

# Arrêt pour la nuit
docker-compose stop

# Relance le lendemain
docker-compose start

# Nettoyage complet (fin de projet)
docker-compose down -v
```

---

## 🧹 Étape 6 : Nettoyage complet (optionnel)

Si vous souhaitez tout supprimer proprement :

```bash
# 1. Arrêter et supprimer le conteneur
docker-compose down

# 2. Supprimer le volume (⚠️ SUPPRIME LES DONNÉES)
docker volume rm mariadb_data

# 3. Supprimer l'image (optionnel, pour libérer de l'espace)
docker rmi mariadb:10.11
```

---

## ✅ Récapitulatif

Vous avez appris à :

- ✅ Créer un fichier `docker-compose.yml` pour MariaDB
- ✅ Démarrer et arrêter votre base de données
- ✅ Vous connecter via terminal ou client graphique
- ✅ Comprendre la persistance des données avec les volumes
- ✅ Gérer le cycle de vie du conteneur

**Fichiers créés :**
- `docker-compose.yml` : Configuration du service
- Volume `mariadb_data` : Stockage des données (géré par Docker)

---

## 🚀 Prochaines étapes

Maintenant que vous maîtrisez la configuration basique, vous pouvez explorer :

- **[1.2 Configuration avec fichier my.cnf](02-config-avec-mycnf.md)** - Personnaliser la configuration MariaDB
- **[1.3 Configuration avec IP fixe](03-config-ip-fixe.md)** - Assigner une adresse IP statique au conteneur
- **[1.4 Gestion des utilisateurs](04-gestion-utilisateurs.md)** - Créer et gérer les utilisateurs SQL
- **[1.5 Accès réseau local](05-acces-reseau-local.md)** - Permettre l'accès depuis d'autres machines

---

## 📚 Ressources complémentaires

- [Documentation officielle MariaDB](https://mariadb.org/documentation/)
- [Documentation Docker Compose](https://docs.docker.com/compose/)
- [Image Docker MariaDB](https://hub.docker.com/_/mariadb)

---

## ❓ FAQ - Questions fréquentes

**Q : Puis-je utiliser une autre version de MariaDB ?**
R : Oui, remplacez `mariadb:10.11` par `mariadb:11.2` (ou autre version disponible sur [Docker Hub](https://hub.docker.com/_/mariadb/tags)).

**Q : Pourquoi utiliser Docker plutôt qu'installer MariaDB directement ?**
R : Docker permet l'isolation (pas de conflit avec d'autres versions), la portabilité (même config partout) et le nettoyage facile (suppression propre).

**Q : Le port 3306 est déjà utilisé sur ma machine, que faire ?**
R : Changez le port hôte dans `docker-compose.yml` : `"3307:3306"` (utilisez le port 3307 sur votre machine).

**Q : Comment sauvegarder ma base de données ?**
R : Utilisez `mysqldump` depuis le conteneur :
```bash
docker exec mariadb_dev mysqldump -u root -p ma_base_dev > backup.sql
```

**Q : Comment restaurer une sauvegarde ?**
R : Copiez le fichier SQL dans le conteneur et importez-le :
```bash
docker cp backup.sql mariadb_dev:/backup.sql
docker exec -it mariadb_dev mariadb -u root -p ma_base_dev < /backup.sql
```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
