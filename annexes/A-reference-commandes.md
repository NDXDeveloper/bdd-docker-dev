# Annexe A - Référence des Commandes Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette annexe est votre **guide de référence complet** pour toutes les commandes Docker et Docker Compose que vous utiliserez au quotidien. Elle est organisée par catégories pour faciliter la recherche.

**Ce que vous allez trouver :**
- 🐳 Toutes les commandes Docker essentielles expliquées
- 🎼 Commandes Docker Compose complètes
- 🐛 Commandes de débogage et diagnostic
- 💡 Exemples concrets pour chaque commande
- ⚡ Raccourcis et astuces de productivité

**💡 Comment utiliser cette annexe :**
- Utilisez le sommaire pour trouver rapidement ce que vous cherchez
- Chaque commande est expliquée avec sa syntaxe et des exemples
- Les options les plus utiles sont mises en évidence

---

## 📑 Table des Matières

1. [Commandes Docker Essentielles](#-1-commandes-docker-essentielles)
2. [Commandes Docker Compose](#-2-commandes-docker-compose)
3. [Commandes de Débogage](#-3-commandes-de-débogage)
4. [Commandes Avancées](#-4-commandes-avancées)
5. [Raccourcis et Alias Utiles](#-5-raccourcis-et-alias-utiles)

---

## 🐳 1. Commandes Docker Essentielles

### 1.1 Gestion des Conteneurs

#### `docker run` - Créer et démarrer un conteneur

**Syntaxe de base :**
```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

**Options les plus courantes :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `-d` | Mode détaché (arrière-plan) | `docker run -d nginx` |
| `-it` | Mode interactif avec terminal | `docker run -it ubuntu bash` |
| `--name` | Nom du conteneur | `docker run --name mon_app nginx` |
| `-p` | Mapping de port | `docker run -p 8080:80 nginx` |
| `-v` | Monter un volume | `docker run -v /data:/app/data nginx` |
| `-e` | Variable d'environnement | `docker run -e DB_PASSWORD=secret mariadb` |
| `--rm` | Supprimer après arrêt | `docker run --rm ubuntu echo "test"` |
| `--restart` | Politique de redémarrage | `docker run --restart=always nginx` |

**Exemples pratiques :**

```bash
# Démarrer un conteneur MariaDB en arrière-plan
docker run -d \
  --name ma_base \
  -e MYSQL_ROOT_PASSWORD=secret \
  -p 3306:3306 \
  -v mariadb_data:/var/lib/mysql \
  mariadb:10.11

# Conteneur temporaire pour tester une commande
docker run --rm ubuntu:20.04 cat /etc/os-release

# Conteneur interactif pour explorer
docker run -it --rm python:3.9 python
```

---

#### `docker ps` - Lister les conteneurs

**Syntaxe :**
```bash
docker ps [OPTIONS]
```

**Options importantes :**

| Option | Description | Exemple |
|--------|-------------|---------|
| *(aucune)* | Conteneurs en cours d'exécution | `docker ps` |
| `-a` ou `--all` | Tous les conteneurs (même arrêtés) | `docker ps -a` |
| `-q` ou `--quiet` | Afficher uniquement les IDs | `docker ps -q` |
| `-f` ou `--filter` | Filtrer les résultats | `docker ps -f "name=maria"` |
| `--format` | Format de sortie personnalisé | `docker ps --format "table {{.Names}}\t{{.Status}}"` |

**Exemples :**

```bash
# Voir tous les conteneurs en cours d'exécution
docker ps

# Voir tous les conteneurs (actifs et arrêtés)
docker ps -a

# Obtenir uniquement les IDs des conteneurs actifs
docker ps -q

# Filtrer par nom
docker ps -f "name=mariadb"

# Affichage personnalisé (nom, état, ports)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

#### `docker start` / `docker stop` / `docker restart` - Contrôler les conteneurs

**Démarrer un conteneur arrêté :**
```bash
docker start <nom_ou_id>

# Exemples
docker start ma_base
docker start 8f3d9a2c1b45
```

**Arrêter un conteneur en cours d'exécution :**
```bash
docker stop <nom_ou_id>

# Exemples
docker stop ma_base
docker stop 8f3d9a2c1b45

# Arrêter avec timeout (secondes avant SIGKILL)
docker stop -t 30 ma_base
```

**Redémarrer un conteneur :**
```bash
docker restart <nom_ou_id>

# Exemples
docker restart ma_base
```

**Astuce - Actions multiples :**
```bash
# Arrêter tous les conteneurs en cours d'exécution
docker stop $(docker ps -q)

# Redémarrer tous les conteneurs arrêtés
docker start $(docker ps -aq -f "status=exited")
```

---

#### `docker rm` - Supprimer un conteneur

**Syntaxe :**
```bash
docker rm [OPTIONS] <nom_ou_id>
```

**Options :**

| Option | Description |
|--------|-------------|
| `-f` | Forcer la suppression (même si actif) |
| `-v` | Supprimer aussi les volumes anonymes associés |

**Exemples :**

```bash
# Supprimer un conteneur arrêté
docker rm ma_base

# Forcer la suppression d'un conteneur actif
docker rm -f ma_base

# Supprimer plusieurs conteneurs
docker rm conteneur1 conteneur2 conteneur3

# Supprimer tous les conteneurs arrêtés
docker rm $(docker ps -aq -f "status=exited")

# Supprimer conteneur + volumes anonymes
docker rm -v ma_base
```

---

#### `docker exec` - Exécuter une commande dans un conteneur actif

**Syntaxe :**
```bash
docker exec [OPTIONS] <nom_ou_id> <commande>
```

**Options courantes :**

| Option | Description |
|--------|-------------|
| `-it` | Mode interactif avec terminal |
| `-u` | Utilisateur (ex: `-u root`) |
| `-w` | Répertoire de travail |
| `-e` | Variable d'environnement |

**Exemples pratiques :**

```bash
# Ouvrir un shell bash dans un conteneur
docker exec -it ma_base bash

# Exécuter une commande SQL dans MariaDB
docker exec -it ma_base mariadb -u root -p

# Exécuter une commande en tant que root
docker exec -it -u root ma_base bash

# Vérifier une variable d'environnement
docker exec ma_base env | grep MYSQL

# Lire un fichier de config
docker exec ma_base cat /etc/mysql/my.cnf
```

---

#### `docker logs` - Afficher les logs d'un conteneur

**Syntaxe :**
```bash
docker logs [OPTIONS] <nom_ou_id>
```

**Options utiles :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `-f` | Suivre en temps réel (comme `tail -f`) | `docker logs -f ma_base` |
| `--tail` | Nombre de lignes à afficher | `docker logs --tail 50 ma_base` |
| `--since` | Logs depuis un moment | `docker logs --since 1h ma_base` |
| `--until` | Logs jusqu'à un moment | `docker logs --until 2023-10-01 ma_base` |
| `-t` | Afficher les timestamps | `docker logs -t ma_base` |

**Exemples :**

```bash
# Voir les logs complets
docker logs ma_base

# Suivre les logs en temps réel
docker logs -f ma_base

# Voir les 100 dernières lignes
docker logs --tail 100 ma_base

# Logs de la dernière heure
docker logs --since 1h ma_base

# Logs avec timestamps
docker logs -ft --tail 50 ma_base
```

---

### 1.2 Gestion des Images

#### `docker images` - Lister les images

**Syntaxe :**
```bash
docker images [OPTIONS]
```

**Options :**

| Option | Description |
|--------|-------------|
| `-a` | Toutes les images (même intermédiaires) |
| `-q` | Uniquement les IDs |
| `--filter` | Filtrer les résultats |
| `--format` | Format personnalisé |

**Exemples :**

```bash
# Lister toutes les images
docker images

# Uniquement les IDs
docker images -q

# Filtrer par nom
docker images mariadb

# Images avec un certain tag
docker images --filter "reference=mariadb:10.*"
```

---

#### `docker pull` - Télécharger une image

**Syntaxe :**
```bash
docker pull <image>[:<tag>]
```

**Exemples :**

```bash
# Télécharger une image spécifique
docker pull mariadb:10.11

# Télécharger la dernière version (tag 'latest')
docker pull mariadb

# Télécharger depuis un registre privé
docker pull registry.exemple.com/mon_image:v1.0
```

---

#### `docker rmi` - Supprimer une image

**Syntaxe :**
```bash
docker rmi [OPTIONS] <image>
```

**Options :**

| Option | Description |
|--------|-------------|
| `-f` | Forcer la suppression |

**Exemples :**

```bash
# Supprimer une image
docker rmi mariadb:10.11

# Supprimer plusieurs images
docker rmi mariadb:10.11 postgres:15 redis:7

# Supprimer toutes les images non utilisées
docker rmi $(docker images -q -f "dangling=true")

# Forcer la suppression
docker rmi -f mariadb:10.11
```

---

#### `docker tag` - Créer un tag pour une image

**Syntaxe :**
```bash
docker tag <source> <cible>
```

**Exemples :**

```bash
# Créer un alias pour une image
docker tag mariadb:10.11 ma_mariadb:stable

# Tag pour un registre privé
docker tag mon_app:latest registry.exemple.com/mon_app:v1.0
```

---

### 1.3 Gestion des Volumes

#### `docker volume ls` - Lister les volumes

```bash
# Lister tous les volumes
docker volume ls

# Filtrer les volumes
docker volume ls -f "name=mariadb"
```

---

#### `docker volume create` - Créer un volume

```bash
# Créer un volume nommé
docker volume create mariadb_data

# Créer avec un driver spécifique
docker volume create --driver local mon_volume
```

---

#### `docker volume inspect` - Inspecter un volume

```bash
# Voir les détails d'un volume
docker volume inspect mariadb_data

# Format JSON plus lisible
docker volume inspect mariadb_data --format '{{json .}}' | jq
```

---

#### `docker volume rm` - Supprimer un volume

```bash
# Supprimer un volume
docker volume rm mariadb_data

# Supprimer plusieurs volumes
docker volume rm vol1 vol2 vol3

# ⚠️ Supprimer tous les volumes non utilisés
docker volume prune
```

---

### 1.4 Gestion des Réseaux

#### `docker network ls` - Lister les réseaux

```bash
# Lister tous les réseaux
docker network ls

# Filtrer par driver
docker network ls -f "driver=bridge"
```

---

#### `docker network create` - Créer un réseau

```bash
# Réseau simple
docker network create mon_reseau

# Réseau avec subnet (pour IP fixes)
docker network create --subnet=172.20.0.0/16 mon_reseau_fixe

# Réseau avec driver spécifique
docker network create --driver overlay mon_reseau_swarm
```

---

#### `docker network inspect` - Inspecter un réseau

```bash
# Voir les détails d'un réseau
docker network inspect mon_reseau

# Voir quels conteneurs sont connectés
docker network inspect mon_reseau --format '{{range .Containers}}{{.Name}} {{end}}'
```

---

#### `docker network connect` / `disconnect` - Gérer les connexions

```bash
# Connecter un conteneur à un réseau
docker network connect mon_reseau ma_base

# Connecter avec une IP spécifique
docker network connect --ip 172.20.0.10 mon_reseau ma_base

# Déconnecter
docker network disconnect mon_reseau ma_base
```

---

#### `docker network rm` - Supprimer un réseau

```bash
# Supprimer un réseau
docker network rm mon_reseau

# Supprimer tous les réseaux non utilisés
docker network prune
```

---

### 1.5 Commandes de Nettoyage

#### `docker system prune` - Nettoyage global

**Syntaxe :**
```bash
docker system prune [OPTIONS]
```

**Options :**

| Option | Description |
|--------|-------------|
| `-a` | Supprimer aussi les images non utilisées |
| `--volumes` | Supprimer aussi les volumes |
| `-f` | Ne pas demander confirmation |

**Exemples :**

```bash
# Nettoyage standard (conteneurs arrêtés, réseaux, images pendantes)
docker system prune

# Nettoyage complet (ATTENTION: supprime beaucoup)
docker system prune -a --volumes

# Nettoyage sans confirmation
docker system prune -f
```

---

#### Commandes de nettoyage spécifiques

```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Supprimer les images non utilisées
docker image prune

# Supprimer toutes les images
docker image prune -a

# Supprimer les volumes non utilisés
docker volume prune

# Supprimer les réseaux non utilisés
docker network prune
```

---

### 1.6 Informations Système

#### `docker info` - Informations sur Docker

```bash
# Afficher toutes les infos Docker
docker info

# Version Docker
docker --version

# Version détaillée
docker version
```

---

#### `docker stats` - Statistiques en temps réel

```bash
# Statistiques de tous les conteneurs actifs
docker stats

# Stats d'un conteneur spécifique
docker stats ma_base

# Stats sans streaming (snapshot)
docker stats --no-stream
```

**Informations affichées :**
- CPU %
- Mémoire utilisée / limite
- Mémoire %
- Net I/O (entrée/sortie réseau)
- Block I/O (entrée/sortie disque)

---

#### `docker inspect` - Inspecter un objet Docker

```bash
# Inspecter un conteneur
docker inspect ma_base

# Inspecter une image
docker inspect mariadb:10.11

# Inspecter un volume
docker inspect mariadb_data

# Extraire une info spécifique (IP du conteneur)
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ma_base

# État d'un conteneur
docker inspect -f '{{.State.Status}}' ma_base
```

---

## 🎼 2. Commandes Docker Compose

### 2.1 Commandes Principales

#### `docker-compose up` - Démarrer les services

**Syntaxe :**
```bash
docker-compose up [OPTIONS] [SERVICE...]
```

**Options courantes :**

| Option | Description | Exemple |
|--------|-------------|---------|
| `-d` | Mode détaché (arrière-plan) | `docker-compose up -d` |
| `--build` | Reconstruire les images | `docker-compose up --build` |
| `--force-recreate` | Forcer la recréation des conteneurs | `docker-compose up --force-recreate` |
| `--no-deps` | Ne pas démarrer les services liés | `docker-compose up --no-deps web` |
| `--scale` | Nombre d'instances | `docker-compose up --scale web=3` |

**Exemples :**

```bash
# Démarrer tous les services
docker-compose up -d

# Démarrer un service spécifique
docker-compose up -d mariadb

# Reconstruire et démarrer
docker-compose up -d --build

# Forcer la recréation d'un service
docker-compose up -d --force-recreate mariadb

# Démarrer avec logs visibles
docker-compose up
```

---

#### `docker-compose down` - Arrêter et supprimer

**Syntaxe :**
```bash
docker-compose down [OPTIONS]
```

**Options importantes :**

| Option | Description | ⚠️ Attention |
|--------|-------------|--------------|
| `-v` ou `--volumes` | Supprimer aussi les volumes | **Perte de données !** |
| `--rmi` | Supprimer les images (`all` ou `local`) | |
| `--remove-orphans` | Supprimer les conteneurs orphelins | |

**Exemples :**

```bash
# Arrêter et supprimer les conteneurs/réseaux
docker-compose down

# ⚠️ Supprimer aussi les volumes (PERTE DE DONNÉES)
docker-compose down -v

# Supprimer images + volumes
docker-compose down -v --rmi all

# Nettoyage complet des orphelins
docker-compose down --remove-orphans
```

---

#### `docker-compose start` / `stop` / `restart`

```bash
# Démarrer des services arrêtés (sans recréer)
docker-compose start

# Démarrer un service spécifique
docker-compose start mariadb

# Arrêter les services (sans supprimer)
docker-compose stop

# Arrêter un service spécifique
docker-compose stop mariadb

# Redémarrer tous les services
docker-compose restart

# Redémarrer un service spécifique
docker-compose restart mariadb
```

---

#### `docker-compose ps` - Lister les services

```bash
# Voir l'état de tous les services
docker-compose ps

# Format de sortie personnalisé
docker-compose ps --services

# Uniquement les services actifs
docker-compose ps --filter "status=running"
```

---

#### `docker-compose logs` - Afficher les logs

**Syntaxe :**
```bash
docker-compose logs [OPTIONS] [SERVICE...]
```

**Options :**

| Option | Description |
|--------|-------------|
| `-f` | Suivre en temps réel |
| `--tail` | Nombre de lignes |
| `--since` | Depuis un moment |
| `-t` | Avec timestamps |

**Exemples :**

```bash
# Logs de tous les services
docker-compose logs

# Logs d'un service spécifique
docker-compose logs mariadb

# Suivre les logs en temps réel
docker-compose logs -f

# Dernières 50 lignes de tous les services
docker-compose logs --tail=50

# Logs de plusieurs services
docker-compose logs mariadb redis

# Logs avec timestamps
docker-compose logs -f -t --tail=100
```

---

#### `docker-compose exec` - Exécuter une commande

```bash
# Ouvrir un shell dans un service
docker-compose exec mariadb bash

# Exécuter une commande dans un service
docker-compose exec mariadb mariadb -u root -p

# Exécuter en tant qu'un utilisateur spécifique
docker-compose exec -u root mariadb bash
```

---

### 2.2 Commandes de Gestion

#### `docker-compose build` - Construire les images

```bash
# Construire toutes les images
docker-compose build

# Construire un service spécifique
docker-compose build web

# Construire sans cache
docker-compose build --no-cache

# Construire en parallèle
docker-compose build --parallel
```

---

#### `docker-compose pull` - Télécharger les images

```bash
# Télécharger toutes les images
docker-compose pull

# Télécharger une image spécifique
docker-compose pull mariadb

# Télécharger sans échec si image manquante
docker-compose pull --ignore-pull-failures
```

---

#### `docker-compose config` - Valider et afficher la config

```bash
# Valider le fichier docker-compose.yml
docker-compose config

# Voir la config résolue (avec variables env)
docker-compose config

# Vérifier sans afficher
docker-compose config -q
```

---

#### `docker-compose top` - Processus en cours

```bash
# Voir les processus de tous les services
docker-compose top

# Processus d'un service spécifique
docker-compose top mariadb
```

---

### 2.3 Commandes de Scaling

#### `docker-compose scale` - Mettre à l'échelle (déprécié)

```bash
# ⚠️ Déprécié - utilisez --scale avec up
docker-compose scale web=3

# Méthode moderne
docker-compose up -d --scale web=3
```

---

### 2.4 Fichiers de Configuration Multiples

```bash
# Utiliser plusieurs fichiers compose
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Spécifier un fichier de config alternatif
docker-compose -f mon-compose.yml up -d

# Utiliser un fichier .env spécifique
docker-compose --env-file .env.prod up -d
```

---

## 🐛 3. Commandes de Débogage

### 3.1 Diagnostic de Problèmes

#### Vérifier l'état global

```bash
# Vue d'ensemble du système Docker
docker info

# Vérifier les services en cours
docker-compose ps

# Vérifier l'état d'un conteneur
docker inspect ma_base | grep -i status

# Voir les événements Docker en temps réel
docker events
```

---

#### Analyser les logs

```bash
# Logs détaillés d'un conteneur
docker logs --details ma_base

# Logs avec timestamps pour chronologie
docker logs -t ma_base

# Filtrer les logs par date
docker logs --since "2023-10-01T10:00:00" ma_base
docker logs --until "2023-10-01T12:00:00" ma_base

# Logs des dernières 5 minutes
docker logs --since 5m ma_base
```

---

#### Vérifier les ressources

```bash
# Utilisation des ressources en temps réel
docker stats

# Snapshot des ressources
docker stats --no-stream

# Disk usage (espace disque)
docker system df

# Détails de l'utilisation disque
docker system df -v
```

---

### 3.2 Inspection Réseau

#### Vérifier la connectivité

```bash
# Lister les réseaux
docker network ls

# Détails d'un réseau
docker network inspect mon_reseau

# IP d'un conteneur
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ma_base

# Ports exposés d'un conteneur
docker port ma_base

# Tester la connectivité depuis un conteneur
docker exec ma_base ping -c 3 google.com

# Tester une connexion TCP
docker exec ma_base telnet localhost 3306
```

---

#### DNS et résolution de noms

```bash
# Vérifier la résolution DNS
docker exec ma_base nslookup google.com

# Fichier hosts du conteneur
docker exec ma_base cat /etc/hosts

# Tester la résolution entre conteneurs
docker exec conteneur1 ping conteneur2
```

---

### 3.3 Inspection des Volumes

```bash
# Lister les volumes
docker volume ls

# Détails d'un volume
docker volume inspect mariadb_data

# Trouver l'emplacement physique
docker volume inspect mariadb_data --format '{{.Mountpoint}}'

# Vérifier l'utilisation d'un volume
docker system df -v | grep mariadb_data
```

---

### 3.4 Vérification des Variables d'Environnement

```bash
# Voir toutes les variables d'un conteneur
docker exec ma_base env

# Filtrer une variable spécifique
docker exec ma_base env | grep MYSQL

# Vérifier les variables depuis compose
docker-compose config
```

---

### 3.5 Accès aux Fichiers de Configuration

```bash
# Lire un fichier de config
docker exec ma_base cat /etc/mysql/my.cnf

# Vérifier les permissions
docker exec ma_base ls -la /etc/mysql/

# Copier un fichier du conteneur vers l'hôte
docker cp ma_base:/etc/mysql/my.cnf ./my.cnf.backup

# Copier un fichier de l'hôte vers le conteneur
docker cp ./my.cnf.new ma_base:/etc/mysql/conf.d/
```

---

### 3.6 Processus et Performance

```bash
# Voir les processus dans un conteneur
docker top ma_base

# Format détaillé
docker top ma_base aux

# Processus via compose
docker-compose top

# Vérifier les limites de ressources
docker inspect ma_base | grep -A 10 "Resources"
```

---

### 3.7 Commandes de Dépannage Avancées

#### Problèmes de démarrage

```bash
# Voir pourquoi un conteneur s'est arrêté
docker inspect ma_base | grep -A 20 "State"

# Code de sortie (exit code)
docker inspect ma_base --format='{{.State.ExitCode}}'

# Raison de l'arrêt
docker inspect ma_base --format='{{.State.Error}}'

# Relancer avec logs visibles (debugging)
docker-compose up mariadb
# (observer les erreurs en temps réel)
```

---

#### Problèmes de performance

```bash
# Statistiques détaillées
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Historique des événements d'un conteneur
docker events --filter 'container=ma_base' --since 1h

# Vérifier les I/O disque
docker exec ma_base iotop
```

---

#### Tests de connectivité

```bash
# Tester un port depuis l'hôte
telnet localhost 3306

# Tester depuis un autre conteneur
docker run --rm --network mon_reseau busybox telnet ma_base 3306

# Scanner les ports ouverts
docker run --rm --network mon_reseau nicolaka/netshoot nmap ma_base
```

---

## 🚀 4. Commandes Avancées

### 4.1 Gestion Avancée des Conteneurs

#### Pause et unpause

```bash
# Mettre en pause (freeze les processus)
docker pause ma_base

# Reprendre l'exécution
docker unpause ma_base
```

---

#### Renommer un conteneur

```bash
# Renommer
docker rename ancien_nom nouveau_nom
```

---

#### Attacher à un conteneur

```bash
# S'attacher à un conteneur en cours d'exécution
docker attach ma_base

# Détacher sans arrêter : Ctrl+P puis Ctrl+Q
```

---

#### Copier des fichiers

```bash
# Du conteneur vers l'hôte
docker cp ma_base:/chemin/fichier.txt ./fichier.txt

# De l'hôte vers le conteneur
docker cp ./fichier.txt ma_base:/chemin/

# Copier un dossier entier
docker cp ma_base:/var/lib/mysql ./backup_mysql/
```

---

### 4.2 Export et Import

#### Sauvegarder et restaurer des images

```bash
# Exporter une image vers un fichier tar
docker save -o mariadb_backup.tar mariadb:10.11

# Importer une image depuis un fichier tar
docker load -i mariadb_backup.tar
```

---

#### Export et import de conteneurs

```bash
# Exporter le système de fichiers d'un conteneur
docker export ma_base > conteneur_backup.tar

# Importer comme image
docker import conteneur_backup.tar ma_base:snapshot
```

---

### 4.3 Commit (créer une image depuis un conteneur)

```bash
# Créer une image depuis un conteneur modifié
docker commit ma_base ma_mariadb_custom:v1

# Avec message et auteur
docker commit -m "Config personnalisée" -a "Nicolas" ma_base ma_mariadb:v2
```

---

### 4.4 Limitations de Ressources

#### Lors de la création

```bash
# Limiter la mémoire
docker run -m 512m mariadb:10.11

# Limiter le CPU (en parts)
docker run --cpus="1.5" mariadb:10.11

# CPU shares (priorité relative)
docker run --cpu-shares=512 mariadb:10.11
```

---

#### Modifier les limites d'un conteneur existant

```bash
# Mettre à jour la mémoire
docker update --memory 1g ma_base

# Mettre à jour les CPUs
docker update --cpus 2 ma_base

# Politique de redémarrage
docker update --restart=always ma_base
```

---

## ⚡ 5. Raccourcis et Alias Utiles

### 5.1 Alias Bash/Zsh

Ajoutez ces alias dans votre `~/.bashrc` ou `~/.zshrc` :

```bash
# Alias Docker de base
alias d='docker'
alias dc='docker-compose'

# Conteneurs
alias dps='docker ps'
alias dpsa='docker ps -a'
alias drmall='docker rm $(docker ps -aq)'

# Images
alias di='docker images'
alias drmiall='docker rmi $(docker images -q)'

# Logs
alias dlog='docker logs -f --tail 100'

# Compose
alias dcu='docker-compose up -d'
alias dcd='docker-compose down'
alias dcl='docker-compose logs -f --tail 100'
alias dcps='docker-compose ps'

# Nettoyage
alias dclean='docker system prune -a --volumes -f'
alias dcleanc='docker container prune -f'
alias dcleani='docker image prune -a -f'
alias dcleanv='docker volume prune -f'

# Stats
alias dst='docker stats --no-stream'

# Shell rapide
alias dex='docker exec -it'
alias dbash='docker exec -it $1 bash'
```

**Utilisation :**
```bash
# Au lieu de : docker-compose up -d
dcu

# Au lieu de : docker exec -it ma_base bash
dex ma_base bash

# Au lieu de : docker ps -a
dpsa
```

---

### 5.2 Fonctions Shell Utiles

Ajoutez ces fonctions dans votre `~/.bashrc` ou `~/.zshrc` :

```bash
# Entrer rapidement dans un conteneur
dbash() {
    docker exec -it "$1" bash
}

# Logs avec filtre de date
dlogdate() {
    docker logs --since "$1" "$2"
}

# IP d'un conteneur
dip() {
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' "$1"
}

# Arrêter tous les conteneurs
dstopall() {
    docker stop $(docker ps -q)
}

# Supprimer tous les conteneurs arrêtés
drmexited() {
    docker rm $(docker ps -aq -f status=exited)
}

# Taille des volumes
dvolsize() {
    docker system df -v | grep "$1"
}
```

**Utilisation :**
```bash
# Entrer dans un conteneur
dbash ma_base

# IP du conteneur
dip ma_base

# Arrêter tout
dstopall
```

---

### 5.3 Scripts Shell Pratiques

#### Script de backup de volume

```bash
#!/bin/bash
# backup-volume.sh

VOLUME_NAME=$1
BACKUP_FILE="${VOLUME_NAME}_$(date +%Y%m%d_%H%M%S).tar.gz"

docker run --rm \
  -v "$VOLUME_NAME":/data \
  -v "$(pwd)":/backup \
  alpine tar czf "/backup/$BACKUP_FILE" -C /data .

echo "Backup créé : $BACKUP_FILE"
```

**Utilisation :**
```bash
./backup-volume.sh mariadb_data
```

---

#### Script de restauration de volume

```bash
#!/bin/bash
# restore-volume.sh

VOLUME_NAME=$1
BACKUP_FILE=$2

docker run --rm \
  -v "$VOLUME_NAME":/data \
  -v "$(pwd)":/backup \
  alpine sh -c "cd /data && tar xzf /backup/$BACKUP_FILE"

echo "Volume $VOLUME_NAME restauré depuis $BACKUP_FILE"
```

**Utilisation :**
```bash
./restore-volume.sh mariadb_data mariadb_data_20241029.tar.gz
```

---

## 📊 Tableaux de Référence Rapide

### Codes de sortie (Exit Codes) courants

| Code | Signification |
|------|---------------|
| 0 | Succès (sortie normale) |
| 1 | Erreur générale d'application |
| 125 | Erreur Docker (commande invalide) |
| 126 | Commande invoquée ne peut pas être exécutée |
| 127 | Commande non trouvée |
| 130 | Conteneur arrêté par Ctrl+C |
| 137 | Conteneur tué (SIGKILL) - souvent OOM |
| 139 | Segmentation fault |
| 143 | Conteneur arrêté normalement (SIGTERM) |

---

### Variables d'Environnement Docker

| Variable | Description | Exemple |
|----------|-------------|---------|
| `DOCKER_HOST` | Hôte Docker | `tcp://192.168.1.50:2375` |
| `DOCKER_TLS_VERIFY` | Activer TLS | `1` |
| `DOCKER_CERT_PATH` | Chemin des certificats | `/home/user/.docker` |
| `COMPOSE_PROJECT_NAME` | Nom du projet Compose | `mon_projet` |
| `COMPOSE_FILE` | Fichier compose à utiliser | `docker-compose.prod.yml` |

---

### Formats de sortie utiles

#### Format personnalisé pour docker ps

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"
```

#### Format JSON pour parsing

```bash
docker inspect ma_base --format='{{json .State}}' | jq
```

#### Template Go (filtres avancés)

```bash
# Tous les ports mappés
docker inspect ma_base --format='{{range $p, $conf := .NetworkSettings.Ports}}{{$p}} -> {{(index $conf 0).HostPort}} {{end}}'
```

---

## 🎓 Patterns et Astuces

### Pattern 1 : One-liner pour tout nettoyer

```bash
# Nettoyage complet (conteneurs + images + volumes)
docker stop $(docker ps -aq) && \
docker rm $(docker ps -aq) && \
docker rmi $(docker images -q) && \
docker volume prune -f && \
docker network prune -f
```

---

### Pattern 2 : Backup complet d'un projet Compose

```bash
# Script de backup
#!/bin/bash
PROJECT_NAME="mon_projet"
BACKUP_DIR="./backups/$(date +%Y%m%d)"

mkdir -p "$BACKUP_DIR"

# Exporter la config
docker-compose config > "$BACKUP_DIR/docker-compose.yml"

# Sauvegarder chaque volume
for volume in $(docker volume ls -q | grep "$PROJECT_NAME"); do
    docker run --rm \
        -v "$volume":/data \
        -v "$BACKUP_DIR":/backup \
        alpine tar czf "/backup/$volume.tar.gz" -C /data .
done

echo "Backup terminé dans $BACKUP_DIR"
```

---

### Pattern 3 : Monitoring rapide

```bash
# Boucle de monitoring (rafraîchit toutes les 5 secondes)
watch -n 5 'docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"'
```

---

### Pattern 4 : Test de santé automatique

```bash
# Vérifier qu'un service répond
until docker exec ma_base mysqladmin ping -h localhost -u root -proot_pass &> /dev/null; do
    echo "En attente de MariaDB..."
    sleep 2
done
echo "MariaDB est prêt !"
```

---

## ✅ Checklist des Commandes à Connaître

### 🟢 Niveau Débutant (Essential)

- [ ] `docker run`
- [ ] `docker ps` / `docker ps -a`
- [ ] `docker stop` / `docker start`
- [ ] `docker rm`
- [ ] `docker logs`
- [ ] `docker exec -it`
- [ ] `docker-compose up -d`
- [ ] `docker-compose down`
- [ ] `docker-compose logs -f`

### 🟡 Niveau Intermédiaire

- [ ] `docker inspect`
- [ ] `docker stats`
- [ ] `docker volume ls` / `rm`
- [ ] `docker network ls` / `create`
- [ ] `docker system prune`
- [ ] `docker-compose restart`
- [ ] `docker-compose exec`
- [ ] `docker-compose config`

### 🔴 Niveau Avancé

- [ ] `docker commit`
- [ ] `docker save` / `load`
- [ ] `docker export` / `import`
- [ ] `docker update`
- [ ] Personnalisation avec `--format`
- [ ] Scripts de backup/restore
- [ ] Monitoring avec `watch` + `docker stats`

---

## 🔗 Ressources Complémentaires

### Documentation Officielle

- 📖 [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- 📖 [Docker Compose CLI Reference](https://docs.docker.com/compose/reference/)
- 📖 [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)

### Cheat Sheets

- 📄 [Docker Cheat Sheet (officiel)](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
- 📄 [Docker Compose Cheat Sheet](https://devhints.io/docker-compose)

### Outils Interactifs

- 🎮 [Play with Docker](https://labs.play-with-docker.com/) - Environnement Docker en ligne
- 🖥️ [Portainer](https://www.portainer.io/) - Interface graphique pour Docker

---

## 💡 Conseils d'Utilisation

### Bonnes Pratiques

1. **Toujours utiliser `-d` en production**
   ```bash
   docker-compose up -d  # Pas docker-compose up
   ```

2. **Vérifier avant de supprimer**
   ```bash
   docker ps -a  # Voir ce qui va être supprimé
   docker-compose down  # Puis supprimer
   ```

3. **Lire les logs en cas d'erreur**
   ```bash
   docker-compose logs -f --tail 100
   ```

4. **Utiliser des noms explicites**
   ```bash
   docker run --name ma_base_dev  # Pas un ID aléatoire
   ```

5. **Nettoyer régulièrement**
   ```bash
   docker system prune  # Une fois par semaine
   ```

---

## 🚀 Prochaines Étapes

Maintenant que vous connaissez toutes les commandes, explorez :

- **[Annexe B - Gestion des réseaux](B-gestion-reseaux.md)** - Approfondir les réseaux Docker
- **[Annexe C - Gestion des volumes](C-gestion-volumes.md)** - Maîtriser les volumes
- **[Annexe E - Dépannage](E-depannage.md)** - Résoudre les problèmes courants

---

## 📝 Notes Finales

Cette annexe est une référence vivante. N'hésitez pas à :
- 📌 L'imprimer et la garder à portée de main
- 🔖 La marquer dans vos favoris
- ✏️ Y ajouter vos propres notes et commandes

**💡 Astuce :** Créez votre propre fichier `my-docker-commands.md` avec vos commandes les plus utilisées et exemples spécifiques à vos projets !

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
