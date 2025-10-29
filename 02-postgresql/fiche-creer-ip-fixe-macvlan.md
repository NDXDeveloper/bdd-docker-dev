Fiche pour mettre en place une base de données PostgreSQL sur Docker avec sa propre adresse IP (via Macvlan), l'extension `pg_cron` pour les tâches planifiées, et la configuration de sécurité pour deux utilisateurs.


-----

## Prérequis

  * **Docker** et **Docker Compose** installés sur votre machine hôte.
  * **Connaître votre réseau local** : Vous aurez besoin de connaître la plage d'adresses IP de votre sous-réseau (ex: `192.168.1.0/24`), votre passerelle (ex: `192.168.1.1`) et le nom de votre interface réseau physique (ex: `eth0`, `enp3s0`). Vous pouvez trouver cela avec la commande `ip route` ou `ifconfig` sur Linux/macOS.

-----

## Étape 1 : Création du réseau Docker Macvlan

Pour que votre conteneur ait sa **propre adresse IP sur votre LAN** (et non une simple redirection de port), nous devons utiliser un réseau `macvlan`. Cela le fera apparaître comme un appareil distinct sur votre réseau.

1.  Ouvrez un terminal sur votre machine hôte.
2.  Créez le réseau Docker. **Adaptez les valeurs `subnet`, `gateway`, `ip-range` et `parent` à votre propre réseau.**

> **Attention :** L'adresse `ip-range` doit être une adresse (ou plage) non utilisée sur votre réseau, en dehors de la plage DHCP si possible, pour éviter les conflits. L'interface `parent` doit être votre interface réseau principale (ex: `eth0`).

```bash
docker network create -d macvlan \
    --subnet=192.168.1.0/24 \
    --gateway=192.168.1.1 \
    --ip-range=192.168.1.50/32 \
    -o parent=eth0 \
    my_macvlan_net
```

  * `--ip-range`: Je réserve ici une seule IP, `192.168.1.50`. C'est celle que notre conteneur utilisera.
  * `my_macvlan_net`: C'est le nom que nous donnons à ce réseau.

-----

## Étape 2 : Préparation de l'arborescence du projet

Créez un dossier pour votre projet (ex: `my-postgres-project`) et placez-y les fichiers suivants.

```
my-postgres-project/
├── docker-compose.yml
├── init-db.sh
├── pg_hba.custom.conf
└── postgresql.custom.conf
```

-----

## Étape 3 : Fichiers de configuration

### 1\. `docker-compose.yml`

Ce fichier orchestre la création de notre conteneur.

```yaml
version: '3.8'

services:
  postgres-db:
    # On utilise l'image supabase/postgres qui inclut pg_cron par défaut
    image: supabase/postgres:15.1.0.117
    container_name: postgres_custom_ip
    restart: unless-stopped

    environment:
      # Ceci crée l'Utilisateur 1 (Super Admin Global)
      # Il aura TOUS les droits et sera le super-utilisateur par défaut.
      POSTGRES_USER: pgadmin
      POSTGRES_PASSWORD: "ChangeMe_SuperAdminPassword123"
      # POSTGRES_DB: main_db # Décommentez pour créer une DB spécifique au démarrage

    volumes:
      # Volume pour la persistance des données
      - pg_data:/var/lib/postgresql/data
      # Montage de nos fichiers de configuration personnalisés
      - ./postgresql.custom.conf:/etc/postgresql/postgresql.conf
      - ./pg_hba.custom.conf:/etc/postgresql/pg_hba.conf
      # Script d'initialisation (pour créer User 2 et activer pg_cron)
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh

    networks:
      # Assignation au réseau macvlan avec l'IP statique
      pg_net:
        ipv4_address: 192.168.1.50 # L'IP que nous avons réservée !

    command: postgres -c config_file=/etc/postgresql/postgresql.conf -c hba_file=/etc/postgresql/pg_hba.conf

volumes:
  pg_data:

networks:
  pg_net:
    # On indique à Compose d'utiliser le réseau externe créé à l'étape 1
    external: true
    name: my_macvlan_net
```

### 2\. `postgresql.custom.conf`

Ce fichier configure PostgreSQL pour écouter sur toutes les interfaces et charger `pg_cron`.

```ini
# Inclure la configuration de base de l'image
include = '/usr/share/postgresql/postgresql.conf.sample'

# --- Connexions ---
listen_addresses = '*'       # Écouter sur toutes les interfaces IP

# --- Extensions ---
# Activer pg_cron pour les tâches planifiées
shared_preload_libraries = 'pg_cron'

# --- Fichiers de configuration ---
# Spécifier le HBA (Host-Based Authentication)
hba_file = '/etc/postgresql/pg_hba.conf'
```

### 3\. `pg_hba.custom.conf`

C'est le fichier **crucial** pour la sécurité. Il définit *qui* peut se connecter, *d'où*, et *comment*. Nous le remplaçons entièrement.

```ini
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Autoriser les connexions locales (ex: psql) via socket Unix
local   all             all                                     peer

# Autoriser les connexions locales (ex: "localhost") via TCP/IP
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256

# --- Utilisateur 1 : pgadmin (Accès total depuis PARTOUT) ---
# Autorise 'pgadmin' à se connecter à n'importe quelle BDD depuis n'importe quelle IP (0.0.0.0/0)
host    all             pgadmin         0.0.0.0/0               scram-sha-256
host    all             pgadmin         ::/0                    scram-sha-256

# --- Utilisateur 2 : localadmin (Accès total en LOCAL SEULEMENT) ---
# Autorise 'localadmin' seulement depuis "localhost" (donc depuis l'intérieur du conteneur)
host    all             localadmin      127.0.0.1/32            scram-sha-256
host    all             localadmin      ::1/128                 scram-sha-256

# --- IMPORTANT : Bloquer localadmin de l'extérieur ---
# Toute autre tentative de 'localadmin' (ex: depuis 192.168.1.10) sera rejetée.
host    all             localadmin      0.0.0.0/0               reject
host    all             localadmin      ::/0                    reject
```

### 4\. `init-db.sh`

Ce script s'exécute **une seule fois** au premier démarrage du conteneur pour initialiser la base.

**Important :** Rendez ce fichier exécutable \!
`chmod +x init-db.sh`

```bash
#!/bin/bash
set -e

# $POSTGRES_USER est 'pgadmin' (défini dans docker-compose.yml)
# Ce script s'exécute en tant que 'pgadmin'
psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL

    -- === Création de l'Utilisateur 2 (Admin Local) ===
    -- Note : "accède à tout sur la base" est interprété comme SUPERUSER
    CREATE USER localadmin WITH PASSWORD 'ChangeMe_LocalAdminPassword456';
    ALTER USER localadmin WITH SUPERUSER;

    -- === Activation de pg_cron ===
    -- L'extension doit être créée dans la base où les jobs s'exécuteront
    -- (ici, la base par défaut 'postgres')
    CREATE EXTENSION IF NOT EXISTS pg_cron;

    -- Optionnel : Donner les droits à pgadmin pour gérer les jobs cron
    GRANT USAGE ON SCHEMA cron TO pgadmin;

EOSQL
```

-----

## Étape 4 : Lancement et Vérification

1.  **Rendez le script `init-db.sh` exécutable :**
    ```bash
    chmod +x init-db.sh
    ```
2.  **Démarrez le conteneur :**
    ```bash
    docker-compose up -d
    ```

### Vérifications

  * **Vérifier le conteneur :**
    `docker ps` (Vous devriez voir `postgres_custom_ip` en cours d'exécution).

  * **Pinger l'IP :**
    Depuis votre machine hôte *ou une autre machine sur le LAN*, vous devriez pouvoir pinger l'IP :
    `ping 192.168.1.50`
    *(Note : certains hôtes Docker ne peuvent pas pinger leur propre conteneur macvlan. Testez depuis un autre appareil.)*

  * **Test User 1 (pgadmin - Accès Global) :**
    Depuis n'importe quelle machine sur le réseau :

    ```bash
    psql -h 192.168.1.50 -U pgadmin -d postgres
    # Il vous demandera 'ChangeMe_SuperAdminPassword123'
    ```

    **Succès \!**

  * **Test User 2 (localadmin - Échec externe) :**
    Depuis n'importe quelle machine sur le réseau :

    ```bash
    psql -h 192.168.1.50 -U localadmin -d postgres
    # Il vous demandera 'ChangeMe_LocalAdminPassword456'
    # Sortie attendue : psql: error: connection to server... FATAL: ...
    ```

    **Échec (attendu) \!** Cela prouve que la règle `reject` fonctionne.

  * **Test User 2 (localadmin - Succès local) :**
    Connectez-vous à l'intérieur du conteneur :

    ```bash
    docker exec -it postgres_custom_ip psql -U localadmin -d postgres
    ```

    **Succès \!** Vous êtes connecté en tant que `localadmin`.

-----

## Étape 5 : Utiliser `pg_cron` (Tâches planifiées)

Maintenant que `pg_cron` est actif, vous pouvez planifier des tâches (en tant que super-utilisateur comme `pgadmin`).

```sql
-- Connectez-vous avec psql -h 192.168.1.50 -U pgadmin ...

-- Exemple : Insérer l'heure actuelle dans une table chaque minute
-- 1. Créer une table de test
CREATE TABLE IF NOT EXISTS cron_test (ts timestamp);

-- 2. Planifier la tâche
SELECT cron.schedule(
    'every-minute-job', -- Nom du job
    '* * * * *',        -- Expression CRON (chaque minute)
    $$ INSERT INTO cron_test (ts) VALUES (now()); $$ -- Commande SQL
);

-- 3. Vérifier après quelques minutes
-- SELECT * FROM cron_test;
```

-----

## Étape 6 : Suppression complète

Pour tout arrêter et nettoyer :

1.  **Arrêter et supprimer le conteneur :**
    (Dans le dossier `my-postgres-project`)

    ```bash
    docker-compose down
    ```

2.  **Supprimer le volume de données (ATTENTION : supprime toutes les données) :**

    ```bash
    docker volume rm my-postgres-project_pg_data
    ```

3.  **Supprimer le réseau Macvlan :**

    ```bash
    docker network rm my_macvlan_net
    ```

4.  **Supprimer vos fichiers de configuration :**

    ```bash
    rm -rf ../my-postgres-project
    ```
