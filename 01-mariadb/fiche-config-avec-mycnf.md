Fiche pour mettre en place une base de données MariaDB avec Docker, en activant le planificateur d'événements (fonctions récurrentes) et en configurant les utilisateurs .

L'utilisation de `docker-compose` est fortement recommandée par rapport à une simple commande `docker run`, car elle permet de gérer la configuration (comme le `my.cnf`) et les volumes de manière beaucoup plus propre et reproductible.

-----

## 🚀 1. Fichiers de Mise en Place

Créez un dossier pour votre projet (par exemple, `mariadb_dev`) et placez-y les deux fichiers suivants.

### A. Fichier `docker-compose.yml`

Ce fichier décrit votre service MariaDB, son mot de passe root, le port à exposer et les volumes pour les données et la configuration.

```yaml
version: '3.8'

services:
  mariadb:
    # Utiliser une version spécifique est une bonne pratique
    image: mariadb:10.11
    container_name: mariadb_local
    restart: always
    environment:
      # !! Changez ce mot de passe !!
      MYSQL_ROOT_PASSWORD: 'un_mot_de_passe_root_tres_securise'
    ports:
      # Port hôte : Port conteneur
      # Permet de se connecter depuis votre PC (DataGrip, DBeaver...) sur localhost:3306
      - "3306:3306"
    volumes:
      # Stockage persistant des données (bind mount)
      # Les données seront stockées dans un dossier 'data' à côté de ce fichier
      - ./data:/var/lib/mysql
      # Fichier de configuration personnalisé pour activer l'Event Scheduler
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf
```

### B. Fichier `my.cnf`

Ce fichier de configuration sera lu par MariaDB au démarrage pour activer les fonctionnalités requises.

```ini
[mysqld]
# C'est la ligne clé pour activer les "fonctions récurrentes programmées"
event_scheduler = ON
```

-----

## ▶️ 2. Lancement du Conteneur

1.  **Créez le dossier data** : Dans le même répertoire que vos fichiers, créez le dossier qui accueillera les données (MariaDB a besoin qu'il existe).
    ```bash
    mkdir data
    ```
2.  **Lancez Docker Compose** : Ouvrez un terminal dans ce dossier et exécutez :
    ```bash
    docker-compose up -d
    ```
      * `-d` signifie "detached" (le conteneur tourne en arrière-plan).

Votre base de données MariaDB est maintenant lancée, persistante et le planificateur d'événements est activé.

-----

## 🔐 3. Création des Utilisateurs

Nous allons maintenant nous connecter en `root` (à l'intérieur du conteneur) pour créer vos deux utilisateurs personnalisés.

1.  **Accédez au shell SQL du conteneur :**

    ```bash
    docker exec -it mariadb_local mariadb -u root -p
    ```

2.  **Entrez le mot de passe** : Saisissez le `MYSQL_ROOT_PASSWORD` que vous avez défini dans le `docker-compose.yml`.

3.  **Exécutez les commandes SQL suivantes** :

      * **Utilisateur 1 : Accès total depuis n'importe où (`%`)**

          * Idéal pour se connecter depuis votre machine hôte (DataGrip, HeidiSQL, etc.).
          * Remplacez `mot_de_passe_admin_all` par un mot de passe sécurisé.

        <!-- end list -->

        ```sql
        CREATE USER 'admin_all'@'%' IDENTIFIED BY 'mot_de_passe_admin_all';
        GRANT ALL PRIVILEGES ON *.* TO 'admin_all'@'%' WITH GRANT OPTION;
        ```

      * **Utilisateur 2 : Accès total local (`localhost`)**

          * Cet utilisateur ne peut se connecter que depuis `localhost` *à l'intérieur* du conteneur.
          * C'est l'utilisateur parfait à donner à une autre application (ex: un backend PHP/Node) qui tournerait dans le *même réseau Docker* et accéderait à la BDD via son nom de service (`mariadb_local`) ou `localhost` (si dans le même pod, par ex).
          * Remplacez `mot_de_passe_admin_local` par un mot de passe sécurisé.

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

## ✅ 4. Vérifications

### Triggers (Déclencheurs)

Les triggers sont **activés par défaut** sur MariaDB. Le privilège `TRIGGER` est inclus dans `ALL PRIVILEGES` que vous venez d'accorder à vos deux utilisateurs. Vous n'avez rien de plus à faire. Vos utilisateurs peuvent créer, voir et exécuter des triggers.

### Fonctions récurrentes (Event Scheduler)

Pour confirmer que le planificateur d'événements est bien activé (grâce à votre `my.cnf`) :

1.  Reconnectez-vous au shell SQL (avec `docker exec...` comme ci-dessus, ou via votre client BDD avec l'utilisateur `admin_all`).
2.  Exécutez :
    ```sql
    SHOW VARIABLES LIKE 'event_scheduler';
    ```
3.  Vous devriez voir :
    ```
    +-----------------+-------+
    | Variable_name   | Value |
    +-----------------+-------+
    | event_scheduler | ON    |
    +-----------------+-------+
    ```
    Si c'est `ON`, tout est prêt pour créer vos `CREATE EVENT...`.

-----

## 🛑 5. Suppression (Nettoyage Complet)

Lorsque vous n'avez plus besoin de cette base de données et que vous voulez *tout* effacer (y compris les données) :

1.  **Arrêtez et supprimez le conteneur :**

      * À l'endroit où se trouve votre `docker-compose.yml` :

    <!-- end list -->

    ```bash
    docker-compose down
    ```

2.  **Supprimez les fichiers de données et de configuration :**

      * **Attention :** Cette action est irréversible et supprime toutes vos données de BDD.

    <!-- end list -->

    ```bash
    # Supprime le dossier contenant les données de la BDD
    rm -rf ./data

    # Supprime les fichiers de configuration
    rm docker-compose.yml my.cnf
    ```

