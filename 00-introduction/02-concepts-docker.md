# Concepts Docker de Base

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Docker peut sembler complexe au premier abord, mais une fois les concepts de base compris, tout devient limpide. Cette fiche vous explique les notions essentielles pour comprendre comment Docker fonctionne, avec des analogies simples et des exemples concrets.

**Ce que vous allez apprendre :**
- Qu'est-ce que Docker et pourquoi l'utiliser
- Les concepts fondamentaux (images, conteneurs, volumes, réseaux)
- Comment ces éléments interagissent ensemble
- La différence avec les machines virtuelles

---

## 🐳 Qu'est-ce que Docker ?

### Définition Simple

Docker est un **outil de conteneurisation** qui permet d'empaqueter une application et toutes ses dépendances dans un **conteneur** isolé et portable.

**Analogie : Le conteneur maritime**

Imaginez que vous devez transporter des marchandises à travers le monde :
- **Sans conteneur** : Vous devez emballer chaque objet différemment (fragile, liquide, périssable...), les charger un par un, les décharger, les recharger... C'est long et complexe.
- **Avec conteneur** : Vous mettez tout dans des conteneurs standardisés. Peu importe ce qu'il y a dedans, ils se manipulent tous de la même façon avec les mêmes outils (grues, camions, bateaux).

**Docker fait la même chose pour les applications** :
- Au lieu de devoir installer manuellement MariaDB sur chaque ordinateur (avec des versions différentes de Linux, Windows, macOS)
- Vous "emballez" MariaDB dans un conteneur Docker
- Ce conteneur fonctionne **exactement de la même façon** sur n'importe quelle machine équipée de Docker

---

## 🎯 Pourquoi Utiliser Docker ?

### Problèmes Classiques en Développement

**Scénario sans Docker :**

Vous travaillez sur un projet qui nécessite :
- PostgreSQL 15
- Redis 7
- Node.js 18

**Les problèmes :**
1. **Installation fastidieuse** : Télécharger, installer, configurer chaque logiciel
2. **Pollution du système** : Votre ordinateur se retrouve avec plein de services installés
3. **Conflits de versions** : Un autre projet a besoin de PostgreSQL 14 → Conflit !
4. **"Ça marche sur ma machine"** : Votre collègue a une version différente → Bugs incompréhensibles
5. **Désinstallation compliquée** : Difficile de tout nettoyer proprement

**Avec Docker :**

```yaml
# Un seul fichier docker-compose.yml
services:
  postgres:
    image: postgres:15
  redis:
    image: redis:7
  node:
    image: node:18
```

✅ Installation en 30 secondes : `docker-compose up -d`
✅ Environnement isolé : N'affecte pas votre système
✅ Versions garanties : Toujours les mêmes versions
✅ Reproductible : Même environnement pour toute l'équipe
✅ Nettoyage facile : `docker-compose down -v` et tout disparaît

---

## 🧱 Les 4 Concepts Fondamentaux

Docker repose sur 4 concepts clés que vous devez comprendre.

---

### 1️⃣ Images Docker

#### Qu'est-ce qu'une Image ?

Une **image** est un **modèle en lecture seule** qui contient tout ce dont une application a besoin pour fonctionner :
- Le système d'exploitation (souvent une version minimale de Linux)
- L'application elle-même (MariaDB, PostgreSQL, Redis...)
- Toutes les dépendances
- Les fichiers de configuration par défaut

**Analogie : Le moule à gâteau**

Une image Docker, c'est comme un moule à gâteau :
- Le moule (image) définit la forme du gâteau
- Vous pouvez créer autant de gâteaux (conteneurs) que vous voulez avec le même moule
- Le moule ne change jamais, mais chaque gâteau est une instance unique

#### Caractéristiques des Images

| Caractéristique | Description |
|----------------|-------------|
| **Lecture seule** | Une image ne peut pas être modifiée une fois créée |
| **Composée de couches** | Chaque modification crée une nouvelle couche (optimisation) |
| **Versionnable** | Les images ont des tags (versions) : `mariadb:10.11`, `mariadb:latest` |
| **Stockées localement** | Téléchargées une fois, réutilisées ensuite |

#### D'où Viennent les Images ?

Les images proviennent de **registres** (comme des magasins d'applications) :

- **Docker Hub** (le plus populaire) : [https://hub.docker.com](https://hub.docker.com)
  - Images officielles vérifiées : `postgres`, `mariadb`, `redis`...
  - Images communautaires

Quand vous faites `docker run mariadb`, Docker :
1. Cherche l'image `mariadb` localement
2. Si elle n'existe pas, la télécharge depuis Docker Hub
3. La stocke localement pour les prochaines utilisations

#### Commandes Images

```bash
# Télécharger une image
docker pull mariadb:10.11

# Lister les images locales
docker images

# Supprimer une image
docker rmi mariadb:10.11

# Voir les détails d'une image
docker inspect mariadb:10.11
```

---

### 2️⃣ Conteneurs Docker

#### Qu'est-ce qu'un Conteneur ?

Un **conteneur** est une **instance en cours d'exécution** d'une image. C'est l'application qui tourne réellement.

**Analogie : Le processus**

Si l'image est le programme installé sur votre ordinateur, le conteneur est le processus lancé :
- Image = Programme Word installé (inactif)
- Conteneur = Word ouvert et en cours d'utilisation (actif)

Vous pouvez avoir plusieurs conteneurs (plusieurs fenêtres Word ouvertes) à partir de la même image (le même programme Word).

#### Caractéristiques des Conteneurs

| Caractéristique | Description |
|----------------|-------------|
| **Éphémère** | Par défaut, les données sont perdues quand le conteneur est supprimé |
| **Isolé** | Chaque conteneur a son propre système de fichiers, réseau, processus |
| **Léger** | Démarre en quelques secondes (vs plusieurs minutes pour une VM) |
| **Modifiable** | Vous pouvez modifier les fichiers dans un conteneur (mais ce n'est pas recommandé) |

#### États d'un Conteneur

Un conteneur peut être dans différents états :

```
Créé → Démarré → En cours d'exécution → Arrêté → Supprimé
```

- **Créé** : Le conteneur existe mais n'est pas démarré
- **En cours d'exécution** : L'application tourne
- **Arrêté** : L'application est stoppée mais le conteneur existe encore (données préservées temporairement)
- **Supprimé** : Le conteneur n'existe plus

#### Cycle de Vie Typique

```bash
# Créer et démarrer un conteneur
docker run -d --name ma_base mariadb:10.11

# Le conteneur tourne...

# Arrêter le conteneur (données conservées)
docker stop ma_base

# Redémarrer le conteneur
docker start ma_base

# Supprimer le conteneur (ATTENTION : perte de données sans volume)
docker rm ma_base
```

#### Commandes Conteneurs

```bash
# Créer et démarrer un conteneur
docker run -d --name mon_conteneur mariadb

# Lister les conteneurs en cours d'exécution
docker ps

# Lister TOUS les conteneurs (même arrêtés)
docker ps -a

# Arrêter un conteneur
docker stop mon_conteneur

# Démarrer un conteneur arrêté
docker start mon_conteneur

# Redémarrer un conteneur
docker restart mon_conteneur

# Supprimer un conteneur (doit être arrêté)
docker rm mon_conteneur

# Supprimer un conteneur en cours d'exécution (force)
docker rm -f mon_conteneur

# Voir les logs d'un conteneur
docker logs mon_conteneur

# Entrer dans un conteneur (mode interactif)
docker exec -it mon_conteneur bash
```

---

### 3️⃣ Volumes Docker

#### Le Problème de la Persistance

**Problème :** Par défaut, quand vous supprimez un conteneur, **toutes ses données sont perdues**.

Imaginez que vous ayez une base de données MariaDB dans un conteneur avec 1000 utilisateurs. Si vous supprimez le conteneur → **tous les utilisateurs disparaissent** ! 😱

**Solution :** Les **volumes** permettent de **persister les données** en dehors du conteneur.

#### Qu'est-ce qu'un Volume ?

Un **volume** est un espace de stockage géré par Docker qui existe **indépendamment** des conteneurs.

**Analogie : Le disque dur externe**

- Le conteneur = Un ordinateur portable
- Le volume = Un disque dur externe branché sur l'ordinateur
- Si l'ordinateur tombe en panne (conteneur supprimé), le disque dur externe (volume) conserve toutes vos données

#### Types de Stockage

Docker propose 3 façons de persister des données :

| Type | Description | Use Case |
|------|-------------|----------|
| **Volume nommé** | Géré par Docker, stocké dans `/var/lib/docker/volumes/` | **Recommandé** pour les BDD |
| **Bind mount** | Lien direct vers un dossier de votre ordinateur | Configuration, développement |
| **tmpfs mount** | Stockage en RAM (volatile) | Données temporaires |

#### Volume Nommé (Recommandé)

```yaml
# docker-compose.yml
services:
  db:
    image: mariadb
    volumes:
      - mariadb_data:/var/lib/mysql  # Volume nommé

volumes:
  mariadb_data:  # Déclaration du volume
```

**Avantages :**
- ✅ Géré par Docker (backups faciles)
- ✅ Portable entre conteneurs
- ✅ Meilleures performances
- ✅ Fonctionne sur tous les OS

#### Bind Mount (Pour Configuration)

```yaml
services:
  db:
    image: mariadb
    volumes:
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf  # Bind mount
      - mariadb_data:/var/lib/mysql             # Volume nommé
```

**Avantages :**
- ✅ Facile à éditer (fichier directement sur votre PC)
- ✅ Partageable via Git

**Inconvénients :**
- ❌ Problèmes de permissions possibles (Linux)
- ❌ Chemins différents selon les OS

#### Commandes Volumes

```bash
# Lister les volumes
docker volume ls

# Créer un volume
docker volume create mon_volume

# Inspecter un volume
docker volume inspect mon_volume

# Supprimer un volume (ATTENTION : perte de données)
docker volume rm mon_volume

# Supprimer tous les volumes non utilisés
docker volume prune
```

---

### 4️⃣ Réseaux Docker

#### Qu'est-ce qu'un Réseau Docker ?

Un **réseau** permet aux conteneurs de **communiquer entre eux** et avec l'extérieur (votre PC, Internet).

**Analogie : Le réseau Wi-Fi de votre maison**

- Chaque appareil (téléphone, ordinateur) = Un conteneur
- Le réseau Wi-Fi = Un réseau Docker
- Les appareils sur le même Wi-Fi peuvent se parler
- Votre box (routeur) = Docker qui gère les connexions

#### Types de Réseaux

Docker crée automatiquement 3 réseaux par défaut :

| Type | Description | Use Case |
|------|-------------|----------|
| **bridge** (par défaut) | Réseau interne isolé | Conteneurs d'une même application |
| **host** | Utilise le réseau de l'hôte directement | Performances maximales (moins d'isolation) |
| **none** | Aucun réseau | Conteneurs complètement isolés |

#### Communication entre Conteneurs

**Scénario :** Vous avez une application web (Node.js) qui doit se connecter à une base de données (PostgreSQL).

**Sans Docker Compose :**
```bash
# Créer un réseau personnalisé
docker network create mon_reseau

# Lancer PostgreSQL sur ce réseau
docker run -d --name postgres --network mon_reseau postgres

# Lancer l'application sur le même réseau
docker run -d --name app --network mon_reseau mon_app
```

L'application peut maintenant se connecter à PostgreSQL avec le nom `postgres` (pas besoin d'IP !).

**Avec Docker Compose (automatique) :**

```yaml
version: '3.8'
services:
  postgres:
    image: postgres

  app:
    image: mon_app
    environment:
      DB_HOST: postgres  # Utilise le nom du service !
```

Docker Compose crée automatiquement un réseau et connecte tous les services. Magique ! ✨

#### Exposition de Ports

Pour accéder à un conteneur depuis votre PC, vous devez **exposer des ports** :

```yaml
services:
  mariadb:
    image: mariadb
    ports:
      - "3306:3306"  # Port_Hôte:Port_Conteneur
```

**Explication :**
- `3306` (à gauche) : Port sur **votre PC**
- `3306` (à droite) : Port **dans le conteneur**
- Vous pouvez vous connecter depuis votre PC sur `localhost:3306`

**Exemple avec port différent :**
```yaml
ports:
  - "33060:3306"
```
Vous vous connectez sur `localhost:33060` (pratique pour éviter les conflits).

#### Commandes Réseaux

```bash
# Lister les réseaux
docker network ls

# Créer un réseau
docker network create mon_reseau

# Inspecter un réseau (voir les conteneurs connectés)
docker network inspect mon_reseau

# Connecter un conteneur à un réseau
docker network connect mon_reseau mon_conteneur

# Déconnecter un conteneur d'un réseau
docker network disconnect mon_reseau mon_conteneur

# Supprimer un réseau
docker network rm mon_reseau
```

---

## 🔄 Comment Tout S'Articule Ensemble

### Exemple Concret : Base de Données MariaDB

Voyons comment tous ces concepts interagissent dans un cas réel.

```yaml
# docker-compose.yml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11           # 1️⃣ IMAGE
    container_name: ma_base        # 2️⃣ CONTENEUR (nom)
    ports:
      - "3306:3306"                 # 4️⃣ RÉSEAU (exposition)
    volumes:
      - mariadb_data:/var/lib/mysql # 3️⃣ VOLUME (persistance)
    environment:
      MARIADB_ROOT_PASSWORD: secret
    networks:
      - mon_reseau                  # 4️⃣ RÉSEAU (connexion)

volumes:
  mariadb_data:                     # 3️⃣ VOLUME (déclaration)

networks:
  mon_reseau:                       # 4️⃣ RÉSEAU (déclaration)
```

**Quand vous lancez `docker-compose up -d` :**

1. **Image** : Docker télécharge (si nécessaire) l'image `mariadb:10.11`
2. **Réseau** : Docker crée le réseau `mon_reseau`
3. **Volume** : Docker crée le volume `mariadb_data`
4. **Conteneur** : Docker crée et démarre le conteneur `ma_base` à partir de l'image
5. Le conteneur :
   - Est connecté au réseau `mon_reseau`
   - Monte le volume `mariadb_data` dans `/var/lib/mysql`
   - Expose le port 3306
   - Démarre MariaDB avec le mot de passe `secret`

**Résultat :**
- Vous pouvez vous connecter sur `localhost:3306`
- Les données sont persistées dans le volume
- Si vous supprimez le conteneur, les données restent
- Si vous recréez le conteneur, il retrouve ses données

---

## 🆚 Docker vs Machines Virtuelles

### Différences Fondamentales

| Aspect | Machine Virtuelle | Docker (Conteneur) |
|--------|-------------------|-------------------|
| **Système d'exploitation** | OS complet (Windows, Linux...) | Partage l'OS de l'hôte |
| **Taille** | Plusieurs GB | Quelques MB |
| **Démarrage** | 1-2 minutes | 1-2 secondes |
| **Ressources** | Lourdes (RAM, CPU pré-allouées) | Légères (ressources partagées) |
| **Isolation** | Très forte (hyperviseur) | Forte (namespaces Linux) |
| **Use case** | Systèmes complets, OS différents | Applications, microservices |

### Illustration Visuelle

**Machine Virtuelle :**
```
┌─────────────────────────────────────┐
│      Ordinateur Physique            │
│  ┌───────────────────────────────┐  │
│  │      Hyperviseur              │  │
│  │  ┌─────────┐    ┌─────────┐   │  │
│  │  │ VM 1    │    │ VM 2    │   │  │
│  │  │ OS      │    │ OS      │   │  │
│  │  │ App     │    │ App     │   │  │
│  │  └─────────┘    └─────────┘   │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Docker :**
```
┌─────────────────────────────────────┐
│      Ordinateur Physique            │
│  ┌───────────────────────────────┐  │
│  │   OS de l'Hôte (Linux/Win)    │  │
│  │  ┌───────────────────────────┐│  │
│  │  │    Docker Engine          ││  │
│  │  │  ┌────┐ ┌────┐ ┌────┐     ││  │
│  │  │  │App1│ │App2│ │App3│     ││  │
│  │  │  └────┘ └────┘ └────┘     ││  │
│  │  └───────────────────────────┘│  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Pourquoi Docker est Plus Léger :**
- Pas besoin d'OS complet par conteneur
- Partage du noyau Linux de l'hôte
- Moins de duplication

---

## 📊 Docker Compose : Le Chef d'Orchestre

### Qu'est-ce que Docker Compose ?

**Docker Compose** est un outil pour définir et gérer des applications **multi-conteneurs**.

**Problème sans Compose :**

Pour lancer une application avec 3 services (web + BDD + cache), vous devez :
```bash
docker network create mon_reseau
docker volume create postgres_data
docker volume create redis_data
docker run -d --name postgres --network mon_reseau -v postgres_data:/var/lib/postgresql/data postgres
docker run -d --name redis --network mon_reseau -v redis_data:/data redis
docker run -d --name web --network mon_reseau -p 80:80 mon_app
```

C'est long, répétitif, et source d'erreurs.

**Solution avec Compose :**

Un seul fichier `docker-compose.yml` :
```yaml
version: '3.8'
services:
  postgres:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis
    volumes:
      - redis_data:/data

  web:
    image: mon_app
    ports:
      - "80:80"

volumes:
  postgres_data:
  redis_data:
```

Une seule commande :
```bash
docker-compose up -d
```

✨ Tout est créé automatiquement : réseau, volumes, conteneurs, connexions !

### Avantages de Docker Compose

| Avantage | Description |
|----------|-------------|
| **Déclaratif** | Vous décrivez CE QUE vous voulez, pas COMMENT le faire |
| **Reproductible** | Le même fichier fonctionne sur tous les PC |
| **Versionnable** | Le fichier peut être commit dans Git |
| **Simple** | Une commande pour tout démarrer/arrêter |

---

## 🎓 Récapitulatif des Concepts

### Tableau Récapitulatif

| Concept | Analogie | Rôle | Commande Clé |
|---------|----------|------|--------------|
| **Image** | Moule à gâteau | Modèle en lecture seule | `docker pull` |
| **Conteneur** | Gâteau sorti du moule | Instance en cours d'exécution | `docker run` |
| **Volume** | Disque dur externe | Persistance des données | `docker volume` |
| **Réseau** | Wi-Fi de la maison | Communication entre conteneurs | `docker network` |
| **Docker Compose** | Recette complète | Orchestration multi-conteneurs | `docker-compose` |

### Les 5 Commandes Essentielles

```bash
# 1. Lancer une stack complète
docker-compose up -d

# 2. Voir ce qui tourne
docker ps

# 3. Voir les logs
docker-compose logs -f

# 4. Arrêter tout
docker-compose down

# 5. Tout supprimer (conteneurs + volumes)
docker-compose down -v
```

---

## 🔍 Concepts Avancés (Aperçu)

Ces concepts seront détaillés plus tard, mais voici un aperçu :

### Dockerfile

Un **Dockerfile** est un fichier texte qui contient les instructions pour **créer une image personnalisée**.

```dockerfile
FROM mariadb:10.11
COPY my.cnf /etc/mysql/conf.d/
ENV MARIADB_ROOT_PASSWORD=secret
```

### Docker Registry

Un **registry** est un serveur qui stocke des images Docker. Docker Hub est le registry public par défaut, mais vous pouvez avoir des registries privés.

### Docker Swarm / Kubernetes

Des outils d'**orchestration** pour gérer des conteneurs en production sur plusieurs machines (clusters).

---

## 💡 Bonnes Pratiques de Base

### À Faire ✅

- **Nommer vos conteneurs** : Plus facile à identifier
  ```yaml
  container_name: ma_base_de_donnees
  ```

- **Utiliser des volumes nommés** : Pour la persistance des données
  ```yaml
  volumes:
    - postgres_data:/var/lib/postgresql/data
  ```

- **Exposer uniquement les ports nécessaires** : Sécurité
  ```yaml
  ports:
    - "127.0.0.1:3306:3306"  # Accessible uniquement en local
  ```

- **Utiliser des versions précises** : Évite les surprises
  ```yaml
  image: mariadb:10.11  # Pas mariadb:latest
  ```

### À Éviter ❌

- ❌ Modifier des fichiers dans un conteneur en cours d'exécution (non persistant)
- ❌ Stocker des données importantes sans volume (perte assurée)
- ❌ Utiliser `latest` en production (versions imprévisibles)
- ❌ Exposer des BDD sur 0.0.0.0 en production (risque de sécurité)

---

## 🚀 Prochaines Étapes

Maintenant que vous comprenez les concepts de base, vous pouvez :

1. **Apprendre les bonnes pratiques** → [Bonnes pratiques](03-bonnes-pratiques.md)
2. **Déployer votre première base de données** → Choisissez dans le [Sommaire](/SOMMAIRE.md)
3. **Explorer les configurations avancées** → Fiches spécifiques par BDD

---

## 📚 Ressources pour Aller Plus Loin

- **Documentation officielle Docker** : [https://docs.docker.com/](https://docs.docker.com/)
- **Docker Compose documentation** : [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
- **Play with Docker** (bac à sable en ligne) : [https://labs.play-with-docker.com/](https://labs.play-with-docker.com/)
- **Awesome Docker** (liste de ressources) : [https://github.com/veggiemonk/awesome-docker](https://github.com/veggiemonk/awesome-docker)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
