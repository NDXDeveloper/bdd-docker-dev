Fiche pour utiliser la méthode du **fichier de configuration `my.cnf`** tout en intégrant la demande d'une **adresse IP statique** pour votre conteneur.

D'abord créer un réseau Docker dédié, puis configurer les fichiers `docker-compose.yml` et `my.cnf` pour qu'ils l'utilisent.

-----

## 🚀 Étape 1 : Création du Réseau Docker Dédié

Pour assigner une adresse IP fixe, nous devons d'abord créer un réseau Docker que le conteneur pourra rejoindre. Nous allons l'appeler `mariadb_net` et lui donner une plage d'adresses (subnet).

Ouvrez votre terminal et exécutez cette commande **une seule fois** :

```bash
docker network create --subnet=172.18.0.0/16 mariadb_net
```

  * `--subnet=172.18.0.0/16` : Définit la plage d'adresses IP disponibles.
  * `mariadb_net` : Le nom de notre réseau personnalisé.

-----

## 🚀 Étape 2 : Fichiers de Mise en Place

Créez un dossier pour votre projet (ex: `mariadb_ip_fixe`) et placez-y les deux fichiers suivants.

### A. Fichier `docker-compose.yml`

Ce fichier décrit le service, spécifie le réseau `mariadb_net` comme externe, et assigne l'adresse IP fixe `172.18.0.10` au service `db`.

```yaml
version: '3.8'

services:
  db:
    image: mariadb:10.11
    container_name: mariadb_with_ip
    restart: unless-stopped
    environment:
      # !! Changez ce mot de passe !!
      MYSQL_ROOT_PASSWORD: 'un_mot_de_passe_root_tres_securise'
    ports:
      # Port hôte : Port conteneur
      # Permet de se connecter depuis votre PC (DataGrip, DBeaver...) sur localhost:3306
      - "3306:3306"
    volumes:
      # Stockage persistant des données (bind mount)
      # Les données seront dans un dossier 'data' à côté de ce fichier
      - ./data:/var/lib/mysql
      # Fichier de configuration personnalisé pour l'Event Scheduler
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf
    networks:
      # Définit le réseau à utiliser et l'IP fixe souhaitée
      mariadb_net:
        ipv4_address: 172.18.0.10

volumes:
  # Cette section n'est plus nécessaire si on utilise un "bind mount" (./data)

networks:
  # Indique à Docker Compose que ce réseau existe déjà
  mariadb_net:
    external: true
```

### B. Fichier `my.cnf`

Comme dans la méthode précédente, ce fichier gère la configuration *interne* de MariaDB.

```ini
[mysqld]
# C'est la ligne clé pour activer les "fonctions récurrentes programmées"
event_scheduler = ON

# Vous pourriez ajouter ici d'autres configurations
# character-set-server = utf8mb4
# collation-server = utf8mb4_unicode_ci
```

-----

## ▶️ Étape 3 : Lancement du Conteneur

1.  **Créez le dossier `data`** : Puisque nous utilisons un "bind mount" (`./data:...`), le dossier doit exister sur votre machine hôte avant le lancement.
    ```bash
    mkdir data
    ```
2.  **Lancez Docker Compose** :
    ```bash
    docker-compose up -d
    ```

Votre base de données MariaDB est lancée, l'Event Scheduler est activé, les données sont persistantes dans le dossier `./data`, et le conteneur est accessible à l'adresse `172.18.0.10` (en plus de `localhost:3306` depuis votre machine).

-----

## 🔐 Étape 4 : Création des Utilisateurs

Connectez-vous à la base de données en tant que `root` pour créer vos utilisateurs.

1.  **Accédez au shell SQL du conteneur :**

    ```bash
    docker exec -it mariadb_with_ip mariadb -u root -p
    ```

2.  **Entrez le mot de passe** : Saisissez le `MYSQL_ROOT_PASSWORD` défini dans votre `docker-compose.yml`.

3.  **Exécutez les commandes SQL suivantes** :

      * **Utilisateur 1 : Accès total depuis n'importe où (`%`)**

          * Permet de se connecter depuis votre machine hôte (DataGrip, etc.) ou tout autre conteneur (même sur un autre réseau Docker, via l'IP `172.18.0.10`).
          * Remplacez `mot_de_passe_admin_all` par un mot de passe sécurisé.

        <!-- end list -->

        ```sql
        CREATE USER 'admin_all'@'%' IDENTIFIED BY 'mot_de_passe_admin_all';
        GRANT ALL PRIVILEGES ON *.* TO 'admin_all'@'%' WITH GRANT OPTION;
        ```

      * **Utilisateur 2 : Accès total local (`localhost`)**

          * Ne peut se connecter que depuis `localhost` *à l'intérieur* du conteneur.
          * Parfait pour une application (ex: un backend) tournant dans le *même réseau Docker* (`mariadb_net`) et qui se connecterait à la BDD en utilisant son nom de service (`db`) ou son IP (`172.18.0.10`).

        <!-- end list -->

        ```sql
        CREATE USER 'admin_local'@'localhost' IDENTIFIED BY 'mot_de_passe_admin_local';
        GRANT ALL PRIVILEGES ON *.* TO 'admin_local'@'localhost' WITH GRANT OPTION;
        ```

      * **Appliquez les changements et quittez :**

        ```sql
        FLUSH PRIVILEGES;
        EXIT;
        ```

-----

## ✅ Étape 5 : Vérifications

### Triggers (Déclencheurs)

Les triggers sont **activés par défaut**. Le privilège `TRIGGER` est inclus dans `ALL PRIVILEGES` que vous avez accordé. Vos utilisateurs peuvent les créer et les gérer sans action supplémentaire.

### Fonctions récurrentes (Event Scheduler)

Vérifiez que le `my.cnf` a bien été pris en compte :

1.  Reconnectez-vous au shell SQL (`docker exec...`).
2.  Exécutez :
    ```sql
    SHOW VARIABLES LIKE 'event_scheduler';
    ```
3.  Vous devez voir `Value: ON`.

-----

## 🛑 Étape 6 : Suppression (Nettoyage Complet)

Lorsque vous voulez tout effacer (conteneur, données, réseau) :

1.  **Arrêtez et supprimez le conteneur et ses volumes anonymes (s'il y en avait) :**

      * À l'endroit où se trouve votre `docker-compose.yml` :

    <!-- end list -->

    ```bash
    docker-compose down
    ```

2.  **Supprimez le réseau externe dédié :**

      * Le `docker-compose down` n'arrête pas le réseau, car nous l'avons marqué comme `external`. Il faut le faire manuellement.

    <!-- end list -->

    ```bash
    docker network rm mariadb_net
    ```

3.  **Supprimez les fichiers de données et de configuration :**

      * **Attention :** Cette action est irréversible et supprime toutes vos données de BDD.

    <!-- end list -->

    ```bash
    # Supprime le dossier contenant les données de la BDD
    rm -rf ./data

    # Supprime les fichiers de configuration
    rm docker-compose.yml my.cnf
    ```

