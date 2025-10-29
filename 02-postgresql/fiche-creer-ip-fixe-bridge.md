Fiche pour mettre en place une base de données PostgreSQL sur Docker avec un réseau bridge dédié, l'extension `pg_cron` pour les tâches planifiées, et des utilisateurs aux droits distincts.

-----

## 1\. Objectif

Configurer un service PostgreSQL dans Docker qui :

  * Tourne sur son **propre réseau bridge** avec une **IP fixe**.
  * Active l'extension **`pg_cron`** pour les tâches récurrentes.
  * Persiste les données sur la machine hôte.
  * Définit deux utilisateurs :
    1.  `admin_remote` : Accès total depuis n'importe quelle adresse IP.
    2.  `admin_local` : Accès total uniquement depuis le réseau Docker local (autres conteneurs) ou l'intérieur du conteneur.

-----

## 2\. Structure des Fichiers

Créez un dossier principal (par ex. `my-postgres`) avec la structure suivante :

```
my-postgres/
├── docker-compose.yml        # Fichier principal d'orchestration
├── conf/
│   └── postgresql.custom.conf  # Configuration pour activer pg_cron & listen_addresses
└── init/
    ├── 01-setup-hba.sh       # Script pour ajouter les règles d'accès (HBA)
    └── 02-init-db.sql        # Script SQL pour créer les users, BDD, et activer pg_cron
```

-----

## 3\. Fichier `docker-compose.yml`

Ce fichier définit le service, le réseau, les volumes et l'IP fixe.

```yaml
# my-postgres/docker-compose.yml
version: '3.8'

services:
  postgres-db:
    # Nous utilisons une image incluant déjà l'extension pg_cron
    image: timescale/pg_cron:postgresql-15
    container_name: my_postgres_db
    environment:
      # Ces identifiants sont pour le super-utilisateur initial (utilisé pour l'init)
      POSTGRES_USER: pg_superuser
      POSTGRES_PASSWORD: super_strong_password
      POSTGRES_DB: default_db
    volumes:
      # 1. Persistance des données de la BDD sur l'hôte
      - ./pg_data:/var/lib/postgresql/data
      # 2. Scripts d'initialisation (exécutés au premier démarrage)
      - ./init:/docker-entrypoint-initdb.d
      # 3. Fichier de configuration additionnel (pour pg_cron)
      - ./conf/postgresql.custom.conf:/etc/postgresql/conf.d/99-custom.conf
    ports:
      # Mappe le port 5432 du conteneur au port 5432 de l'hôte
      # Permet de se connecter depuis la machine hôte (ex: DBeaver, psql)
      - "5432:5432"
    networks:
      my_pg_network:
        # Attribution de l'IP fixe sur ce réseau
        ipv4_address: 172.20.0.10
    restart: unless-stopped

networks:
  my_pg_network:
    # Définition de notre réseau bridge personnalisé
    driver: bridge
    ipam:
      config:
        # Définition du sous-réseau (subnet)
        - subnet: 172.20.0.0/24
```

**Points clés :**

  * **Image :** `timescale/pg_cron:postgresql-15` est basée sur l'image officielle `postgres:15` mais inclut l'extension `pg_cron`.
  * **Volumes :**
      * `./pg_data` : Stocke les données de la base.
      * `./init` : Le point d'entrée Docker de Postgres exécute les scripts `.sh` puis `.sql` de ce dossier par ordre alphabétique.
      * `./conf` : Le fichier `99-custom.conf` sera chargé *après* la configuration de base de Postgres pour ajouter nos directives.
  * **Networks :** Nous créons un réseau `my_pg_network` de type `bridge` et assignons l'IP statique `172.20.0.10` à notre conteneur.

-----

## 4\. Fichiers de Configuration et d'Initialisation

### `conf/postgresql.custom.conf`

Ce fichier ajoute les configurations nécessaires pour `pg_cron` et l'écoute réseau.

```ini
# my-postgres/conf/postgresql.custom.conf

# 1. Écouter sur toutes les interfaces (nécessaire pour l'accès externe)
listen_addresses = '*'

# 2. Charger l'extension pg_cron au démarrage du serveur
shared_preload_libraries = 'pg_cron'

# 3. Spécifier la BDD où pg_cron stocke ses métadonnées (tâches)
cron.database_name = 'default_db'
```

### `init/01-setup-hba.sh`

Ce script *ajoute* nos règles d'accès personnalisées au fichier `pg_hba.conf` (Host-Based Authentication) généré par le point d'entrée Docker.

**Important :** Rendez ce fichier exécutable : `chmod +x init/01-setup-hba.sh`

```bash
#!/bin/sh
# my-postgres/init/01-setup-hba.sh
set -e

# Variable $PGDATA pointant vers le dossier de données (ex: /var/lib/postgresql/data)
PG_HBA_CONF="$PGDATA/pg_hba.conf"

echo "--- Ajout des règles HBA personnalisées ---"

# $PG_HBA_CONF est le fichier de conf généré par le script d'entrée Docker
# Nous ajoutons nos règles à la fin de ce fichier.

# Règle pour admin_remote : Accepte les connexions de PARTOUT (0.0.0.0/0)
echo "host    all             admin_remote    0.0.0.0/0               md5" >> "$PG_HBA_CONF"

# Règle pour admin_local : Accepte les connexions DEPUIS LE SUBNET DOCKER (172.20.0.0/24)
echo "host    all             admin_local     172.20.0.0/24           md5" >> "$PG_HBA_CONF"

# Règle pour admin_local : Accepte aussi les connexions DEPUIS L'INTÉRIEUR (localhost du conteneur)
echo "host    all             admin_local     127.0.0.1/32            md5" >> "$PG_HBA_CONF"
```

### `init/02-init-db.sql`

Ce script SQL crée les utilisateurs, leur donne les droits, et active `pg_cron` avec un exemple de tâche.

```sql
-- my-postgres/init/02-init-db.sql

-- Désactive les messages 'NOTICE' pour un log plus propre
SET client_min_messages TO WARNING;

--- 1. CRÉATION DES UTILISATEURS
CREATE USER admin_remote WITH PASSWORD 'remote_pass_123';
CREATE USER admin_local WITH PASSWORD 'local_pass_456';

--- 2. ATTRIBUTION DES DROITS
-- 'accède à tout' est le plus simple à gérer avec SUPERUSER en dev.
-- ATTENTION: En production, utilisez des GRANTs plus fins.
ALTER USER admin_remote WITH SUPERUSER;
ALTER USER admin_local WITH SUPERUSER;

--- 3. ACTIVATION DE PG_CRON
-- L'extension est créée par le super-utilisateur (pg_superuser) qui exécute ce script
CREATE EXTENSION IF NOT EXISTS pg_cron;

-- Donne aux nouveaux utilisateurs le droit de voir et gérer les tâches cron
GRANT USAGE ON SCHEMA cron TO admin_remote;
GRANT USAGE ON SCHEMA cron TO admin_local;

--- 4. EXEMPLE DE TÂCHE RÉCURRENTE (pg_cron)
-- Crée une table de log pour le test
CREATE TABLE IF NOT EXISTS cron_logs (
  id serial primary key,
  message text,
  logged_at timestamptz default now()
);

-- Planifie une tâche qui s'exécute toutes les minutes
-- La tâche insère un log dans la table 'cron_logs'
-- La tâche s'exécutera avec les droits de l'utilisateur qui la planifie (pg_superuser)
SELECT cron.schedule(
  'job-toutes-les-minutes', -- Nom de la tâche
  '* * * * *',                -- Expression Cron (toutes les minutes)
  $$ INSERT INTO cron_logs (message) VALUES ('Tâche cron exécutée !'); $$ -- Commande SQL
);

-- Pour voir les tâches planifiées : SELECT * FROM cron.job;
-- Pour voir l'historique :         SELECT * FROM cron.job_run_details;

RESET client_min_messages;
```

-----

## 5\. Cycle de Vie : Déploiement et Suppression

### Démarrage et Vérification

1.  **Placez-vous dans le dossier `my-postgres/`** :

    ```bash
    cd /chemin/vers/my-postgres
    ```

2.  **Lancez les services** (le `-d` signifie "detached" pour tourner en arrière-plan) :

    ```bash
    docker-compose up -d
    ```

3.  **Vérifiez les logs** pour vous assurer que tout a démarré correctement (scripts d'init, `pg_cron` chargé) :

    ```bash
    docker-compose logs -f postgres-db
    ```

    *Vous devriez voir les messages de `01-setup-hba.sh` et la création des tables/extensions de `02-init-db.sql`.*

4.  **Vérifiez l'IP du conteneur** (facultatif) :

    ```bash
    docker network inspect my-postgres_my_pg_network
    ```

    *Vous verrez le conteneur `my_postgres_db` listé avec l'IP `172.20.0.10`.*

### Connexion et Test

Vous pouvez maintenant vous connecter :

  * **Test `admin_remote` (devrait fonctionner)** :
    *Depuis votre machine hôte (qui est "remote" / "partout" pour le conteneur).*

    ```bash
    psql -h localhost -p 5432 -U admin_remote -d default_db
    # Mot de passe : remote_pass_123
    ```

  * **Test `admin_local` (ne devrait PAS fonctionner depuis l'hôte)** :
    *Votre machine hôte (ex: 192.168.1.50) n'est pas dans le subnet `172.20.0.0/24`, donc la règle HBA doit la rejeter.*

    ```bash
    psql -h localhost -p 5432 -U admin_local -d default_db
    # Mot de passe : local_pass_456
    # DEVRAIT ÉCHOUER : psql: error: connection to server... FATAL: no pg_hba.conf entry for host...
    ```

  * **Test `admin_local` (depuis le réseau Docker)** :
    *Pour prouver que la règle "locale" fonctionne, lancez un conteneur temporaire sur le même réseau.*

    ```bash
    docker run -it --rm --network=my-postgres_my_pg_network postgres:15 \
      psql -h 172.20.0.10 -U admin_local -d default_db
    # Mot de passe : local_pass_456
    # DEVRAIT FONCTIONNER
    ```

  * **Vérifier la tâche `pg_cron`** :
    *Attendez 1 ou 2 minutes, puis connectez-vous (avec `admin_remote`) et vérifiez la table de log.*

    ```bash
    psql -h localhost -p 5432 -U admin_remote -d default_db -c "SELECT * FROM cron_logs;"
    ```

    *Résultat attendu :*

    ```
     id |       message        |          logged_at
    ----+----------------------+-------------------------------
      1 | Tâche cron exécutée ! | 2025-10-29 00:07:00.048281+01
      2 | Tâche cron exécutée ! | 2025-10-29 00:08:00.039127+01
    (2 rows)
    ```

### Arrêt et Suppression

1.  **Arrêter les conteneurs** (sans supprimer les données) :

    ```bash
    docker-compose stop
    ```

2.  **Redémarrer les conteneurs** :

    ```bash
    docker-compose start
    ```

3.  **Suppression Complète (Nettoyage)** 🧹 :
    *Cette commande arrête les conteneurs, supprime le réseau, ET **supprime le volume de données (`-v`)**.*

    ```bash
    docker-compose down -v
    ```
