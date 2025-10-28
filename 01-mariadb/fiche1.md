Fiche complète pour mettre en place une instance MariaDB sur Docker avec une adresse IP fixe, l'activation du planificateur d'événements (pour les fonctions récurrentes), et la configuration de deux utilisateurs administrateurs.

-----

## 🚀 Fiche de déploiement : MariaDB sur Docker

Cette fiche couvre la configuration d'un réseau Docker pour une IP fixe, le déploiement via `docker-compose` (recommandé), la configuration des utilisateurs et la suppression propre de l'environnement.

### 1\. Prérequis

  * **Docker** installé sur votre machine.
  * **Docker Compose** (généralement inclus avec Docker Desktop).

### 2\. Étape 1 : Création du réseau Docker dédié

Pour assigner une adresse IP statique, nous devons d'abord créer notre propre réseau "bridge" sur Docker.

```bash
# Nous créons un réseau nommé 'mariadb_net' avec un sous-réseau défini
# Cela nous permettra d'assigner l'IP '172.18.0.10' à notre conteneur
docker network create --subnet=172.18.0.0/16 mariadb_net
```

### 3\. Étape 2 : Configuration avec `docker-compose.yml`

Créez un fichier `docker-compose.yml` dans un nouveau dossier. C'est la méthode la plus propre pour gérer la configuration.

**Points clés de cette configuration :**

  * **Triggers :** Ils sont activés par défaut sur MariaDB. Aucune action n'est requise.
  * **Fonctions récurrentes (Events) :** Celles-ci (le `event_scheduler`) ne le sont pas. Nous les activons en passant l'argument `command: ['mysqld', '--event-scheduler=ON']`.
  * **IP Fixe :** Définie dans la section `networks` en utilisant le réseau créé à l'étape 1.

<!-- end list -->

```yaml
version: '3.8'

services:
  db:
    image: mariadb:10.11 # Vous pouvez spécifier la version que vous souhaitez
    container_name: mariadb_with_ip
    restart: unless-stopped
    environment:
      # Définissez un mot de passe root sécurisé
      MARIADB_ROOT_PASSWORD: votre_mot_de_passe_root_solide
    ports:
      # Expose le port 3306 du conteneur sur le port 3306 de l'hôte
      - "3306:3306"
    volumes:
      # Crée un volume nommé 'mariadb_data' pour la persistance des données
      - mariadb_data:/var/lib/mysql
    command:
      # Active le planificateur d'événements pour les tâches récurrentes
      - 'mysqld'
      - '--event-scheduler=ON'
    networks:
      mariadb_net:
        # Assignation de l'adresse IP statique
        ipv4_address: 172.18.0.10

volumes:
  # Déclaration du volume pour qu'il soit géré par Docker
  mariadb_data:

networks:
  # Déclaration du réseau externe que nous avons créé
  mariadb_net:
    external: true
```

### 4\. Étape 3 : Lancement du conteneur

Ouvrez un terminal dans le dossier contenant votre `docker-compose.yml` et lancez :

```bash
docker-compose up -d
```

Votre conteneur MariaDB est maintenant en cours d'exécution, accessible via `localhost:3306` (depuis votre machine hôte) et `172.18.0.10:3306` (depuis d'autres conteneurs sur le même réseau Docker).

### 5\. Étape 4 : Configuration des utilisateurs SQL

Connectez-vous à votre base de données en tant que `root` pour créer les nouveaux utilisateurs.

```bash
# Se connecter au conteneur en mode interactif
docker-compose exec db mariadb -u root -p
```

Entrez le `votre_mot_de_passe_root_solide` défini dans le `docker-compose.yml`.

Une fois dans le shell MariaDB, exécutez les commandes SQL suivantes :

```sql
/*
 * Utilisateur 1 : Accès total depuis n'importe quelle adresse IP ('%')
 * Idéal pour une administration à distance (ex: DBeaver, DataGrip)
 */
CREATE USER 'admin_distant'@'%' IDENTIFIED BY 'votre_mot_de_passe_admin_distant';

GRANT ALL PRIVILEGES ON *.* TO 'admin_distant'@'%' WITH GRANT OPTION;

/*
 * Utilisateur 2 : Accès total depuis localhost (accès local au conteneur)
 * Utile pour les scripts ou applications tournant sur le MÊME conteneur
 * Note : 'localhost' ici signifie 'localhost' VU DE L'INTÉRIEUR du conteneur.
 * Si vous voulez un admin local depuis votre PC hôte, utilisez 'admin_local'@'127.0.0.1'
 */
CREATE USER 'admin_local'@'localhost' IDENTIFIED BY 'votre_mot_de_passe_admin_local';

GRANT ALL PRIVILEGES ON *.* TO 'admin_local'@'localhost' WITH GRANT OPTION;

/*
 * Appliquer les changements
 */
FLUSH PRIVILEGES;

/*
 * Quitter
 */
EXIT;
```

### 6\. Étape 5 : Vérification

Vous pouvez vérifier que tout fonctionne :

1.  **Vérifier l'IP :**

    ```bash
    docker inspect mariadb_with_ip | grep "IPAddress"
    ```

    Vous devriez voir `"IPAddress": "172.18.0.10"`.

2.  **Vérifier l'Event Scheduler :**
    Connectez-vous (avec `root` ou votre nouvel `admin_distant`) et exécutez :

    ```sql
    SHOW VARIABLES LIKE 'event_scheduler';
    ```

    La valeur (`Value`) doit être sur `ON`.

3.  **Tester les connexions :**

      * Essayez de vous connecter depuis votre machine hôte (via un client SQL) avec `admin_distant` sur `127.0.0.1:3306`.
      * Essayez de vous connecter avec `admin_local` (cela ne fonctionnera *que* depuis l'intérieur du conteneur).

-----

### 🧽 Gestion et Suppression

#### Commandes utiles

```bash
# Arrêter le conteneur (sans le supprimer)
docker-compose stop

# Redémarrer le conteneur (après un stop)
docker-compose start

# Voir les logs en direct
docker-compose logs -f
```

#### Suppression complète (Nettoyage)

Pour tout supprimer proprement (conteneur, volume de données, réseau) :

```bash
# 1. Arrêter et supprimer le conteneur défini dans le compose
docker-compose down

# 2. Supprimer le volume (ATTENTION : CECI SUPPRIME TOUTES LES DONNÉES)
docker volume rm <nom_du_dossier>_mariadb_data
# (Remplacez <nom_du_dossier> par le nom du dossier où se trouve votre compose)
# Si vous ne connaissez pas le nom, listez-les avec : docker volume ls

# 3. Supprimer le réseau que nous avons créé
docker network rm mariadb_net
```

-----

### 💡 Alternative (Méthode `docker run` uniquement)

Si vous ne souhaitez pas utiliser Docker Compose, voici la commande `docker run` équivalente (moins facile à gérer) :

```bash
docker run -d \
  --name mariadb_with_ip \
  --network mariadb_net \
  --ip 172.18.0.10 \
  -p "3306:3306" \
  -v mariadb_data:/var/lib/mysql \
  -e MARIADB_ROOT_PASSWORD=votre_mot_de_passe_root_solide \
  mariadb:10.11 \
  mysqld --event-scheduler=ON

# Pour se connecter (en utilisant le nom du conteneur) :
docker exec -it mariadb_with_ip mariadb -u root -p

# Pour supprimer (plus manuel) :
docker stop mariadb_with_ip
docker rm mariadb_with_ip
docker volume rm mariadb_data
docker network rm mariadb_net
```

## Acceder depuis un autre poste du réseau

Pour accéder à votre base de données MariaDB depuis un autre ordinateur de votre réseau local (par exemple, un autre PC portable, une Smart TV, ou un autre serveur physique), vous devez vous assurer de **trois choses** :

1.  **Le service MariaDB écoute sur le port 3306 (déjà fait par le `ports` de Docker Compose).**
2.  **L'ordinateur hôte autorise les connexions externes sur ce port (configuration du pare-feu).**
3.  **L'ordinateur client utilise l'adresse IP de l'ordinateur hôte pour se connecter.**

Voici les étapes à suivre pour permettre cet accès :

-----

## 💻 1. Identifier l'adresse IP de l'hôte

L'ordinateur distant n'accédera pas à l'adresse IP interne du conteneur (`172.18.0.10`) ni à `127.0.0.1` (localhost). Il doit utiliser l'adresse IP que votre box internet a attribuée à votre machine hôte (le PC sur lequel tourne Docker).

1.  **Ouvrez le terminal/invite de commandes** sur l'ordinateur qui exécute Docker.
2.  **Trouvez l'adresse IP locale** de l'hôte :
      * **Windows :** tapez `ipconfig` et cherchez l'adresse IPv4 de votre connexion (souvent du type `192.168.x.x` ou `10.0.x.x`).
      * **Linux/macOS :** tapez `ip addr` ou `ifconfig` et cherchez l'adresse de votre interface réseau active (ex: `eth0` ou `en0`).

> **Exemple :** Supposons que l'adresse IP de votre ordinateur hôte est **`192.168.1.50`**.

-----

## 🔥 2. Configurer le Pare-feu (Étape Cruciale)

C'est l'étape la plus courante qui bloque les connexions distantes. Par défaut, les systèmes d'exploitation bloquent les connexions entrantes, même sur le réseau local.

Vous devez créer une règle pour **autoriser le trafic entrant sur le port 3306 TCP**.

### Pour Windows (Pare-feu Windows Defender)

1.  Recherchez et ouvrez **"Pare-feu Windows Defender avec fonctions avancées de sécurité"**.
2.  Dans le panneau de gauche, cliquez sur **"Règles de trafic entrant"**.
3.  Dans le panneau de droite, cliquez sur **"Nouvelle règle..."**.
4.  Choisissez :
      * Type de règle : **Port**.
      * Protocole : **TCP**.
      * Ports locaux spécifiques : **3306**.
      * Action : **Autoriser la connexion**.
      * Profil : Cochez **Domaine, Privé** (et Public, si vous faites des tests, mais Privé est suffisant pour un réseau maison).
      * Nommez la règle : `MariaDB Docker 3306`.

### Pour macOS / Linux (avec `ufw` par exemple)

Si vous utilisez un pare-feu comme `ufw` (Uncomplicated Firewall) sur Linux, la commande sera :

```bash
sudo ufw allow 3306/tcp
```

-----

## 🛠️ 3. Connexion depuis l'ordinateur distant

Maintenant que l'ordinateur hôte écoute et que le pare-feu est ouvert, vous pouvez vous connecter depuis n'importe quel autre appareil de votre réseau maison.

Dans votre client SQL (HeidiSQL, DBeaver, MySQL Workbench, etc.) sur l'ordinateur distant :

| Paramètre | Valeur à utiliser |
| :--- | :--- |
| **Hôte/IP** | **`192.168.1.50`** (l'IP de votre machine hôte) |
| **Port** | `3306` |
| **Utilisateur** | `admin_distant` |
| **Mot de passe** | `votre_mot_de_passe_admin_distant` |

### Rappel sur les utilisateurs :

  * **`admin_distant`@`%`** : Cet utilisateur est le seul que vous pouvez utiliser, car le `%` signifie qu'il accepte les connexions depuis **n'importe quelle adresse IP** (y compris les adresses de votre réseau local, comme `192.168.1.50`).
  * **`admin_local`@`localhost`** : Cet utilisateur est strictement limité aux connexions provenant de l'intérieur du conteneur lui-même, il ne fonctionnera pas depuis un autre ordinateur du réseau maison.


