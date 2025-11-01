# Annexe E - Dépannage Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Même avec la meilleure configuration, vous rencontrerez des problèmes. Cette annexe vous guide pour **diagnostiquer** et **résoudre** les problèmes les plus courants avec Docker et les bases de données.

**Ce que vous allez apprendre :**
- 🔍 Identifier rapidement la source d'un problème
- 📝 Lire et interpréter les logs Docker
- 🐛 Résoudre les erreurs courantes
- ⚡ Diagnostiquer les problèmes de performances
- 🛠️ Utiliser les outils de diagnostic

**Philosophie du dépannage :**
```
1. Observer (que se passe-t-il ?)
2. Comprendre (pourquoi ça ne marche pas ?)
3. Agir (corriger le problème)
4. Vérifier (ça fonctionne maintenant ?)
```

**Niveau :** 🟡 Intermédiaire (expliqué pour débutants)

**Durée de lecture :** 45 minutes

---

## 📑 Table des Matières

1. [Méthodologie de Dépannage](#-1-méthodologie-de-dépannage)
2. [Problèmes de Démarrage](#-2-problèmes-de-démarrage)
3. [Problèmes de Connexion](#-3-problèmes-de-connexion)
4. [Problèmes de Volumes et Données](#-4-problèmes-de-volumes-et-données)
5. [Vérification des Logs](#-5-vérification-des-logs)
6. [Diagnostic de Performances](#-6-diagnostic-de-performances)
7. [Problèmes Réseau](#-7-problèmes-réseau)
8. [Erreurs Spécifiques par BDD](#-8-erreurs-spécifiques-par-bdd)
9. [Outils de Diagnostic](#-9-outils-de-diagnostic)

---

## 🔍 1. Méthodologie de Dépannage

### 1.1 Les 5 Questions à se Poser

Avant de paniquer, posez-vous ces questions dans l'ordre :

```
1️⃣ Le conteneur est-il démarré ?
   → docker ps

2️⃣ Y a-t-il des erreurs dans les logs ?
   → docker logs <conteneur>

3️⃣ Le service écoute-t-il sur le bon port ?
   → docker port <conteneur>

4️⃣ Puis-je me connecter depuis le conteneur ?
   → docker exec <conteneur> <commande_test>

5️⃣ Les ressources sont-elles suffisantes ?
   → docker stats
```

---

### 1.2 Workflow de Diagnostic

```
Problème détecté
    ↓
Vérifier l'état (docker ps)
    ↓
Lire les logs (docker logs)
    ↓
Identifier la cause
    ↓
Corriger
    ↓
Redémarrer (docker-compose restart)
    ↓
Vérifier (docker ps, docker logs)
    ↓
✅ Problème résolu
```

---

### 1.3 Commandes de Base du Diagnostic

```bash
# 1. État général
docker ps -a                    # Tous les conteneurs
docker-compose ps               # Conteneurs du projet

# 2. Logs
docker logs <conteneur>         # Tous les logs
docker logs -f <conteneur>      # Suivre en temps réel
docker logs --tail 50 <conteneur>  # Dernières lignes

# 3. Détails du conteneur
docker inspect <conteneur>      # Informations complètes

# 4. Processus en cours
docker top <conteneur>          # Processus dans le conteneur

# 5. Ressources
docker stats                    # Utilisation CPU/RAM

# 6. Réseau
docker network inspect <network>  # Détails réseau
docker port <conteneur>           # Ports exposés
```

---

## 🚫 2. Problèmes de Démarrage

### 2.1 Le Conteneur Ne Démarre Pas

#### Symptôme

```bash
docker ps
# Le conteneur n'apparaît pas

docker ps -a
# CONTAINER   STATUS
# mariadb     Exited (1) 2 seconds ago
```

---

#### Diagnostic

```bash
# Voir pourquoi il s'est arrêté
docker logs mariadb
```

---

#### Causes et Solutions Courantes

##### Cause 1 : Mot de Passe Manquant

**Message d'erreur :**
```
ERROR: You need to specify one of MYSQL_ROOT_PASSWORD,
MYSQL_ALLOW_EMPTY_PASSWORD or MYSQL_RANDOM_ROOT_PASSWORD
```

**Solution :**
```yaml
services:
  mariadb:
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}  # ✅ Ajouter
```

Vérifier que le fichier `.env` existe et contient :
```bash
MYSQL_ROOT_PASSWORD=votre_mot_de_passe
```

---

##### Cause 2 : Port Déjà Utilisé

**Message d'erreur :**
```
Error starting userland proxy: listen tcp 0.0.0.0:3306:
bind: address already in use
```

**Signification :** Un autre programme utilise déjà le port 3306.

**Solutions :**

**Option 1 : Changer le port Docker**
```yaml
services:
  mariadb:
    ports:
      - "3307:3306"  # Utiliser le port 3307 à la place
```

**Option 2 : Arrêter le service qui occupe le port**
```bash
# Linux/macOS : Identifier le processus
sudo lsof -i :3306

# Windows : Identifier le processus
netstat -ano | findstr :3306

# Arrêter le service (exemple MySQL natif)
sudo systemctl stop mysql
```

---

##### Cause 3 : Volume Corrompu

**Message d'erreur :**
```
InnoDB: Operating system error number 13 in a file operation
InnoDB: Database page corruption on disk or a failed file read
```

**Solution :**
```bash
# 1. Arrêter le conteneur
docker-compose down

# 2. Supprimer le volume corrompu (⚠️ PERTE DE DONNÉES)
docker volume rm nom_du_volume

# 3. Restaurer depuis un backup
./restore-volume.sh nom_du_volume backup.tar.gz

# 4. Ou repartir de zéro
docker-compose up -d
```

---

##### Cause 4 : Syntaxe Invalide dans docker-compose.yml

**Message d'erreur :**
```
ERROR: yaml.scanner.ScannerError: while scanning for the next token
found character '\t' that cannot start any token
```

**Solution :**
```yaml
# ❌ MAUVAIS (tabulations)
services:
	mariadb:
		image: mariadb

# ✅ BON (espaces)
services:
  mariadb:
    image: mariadb
```

**Astuce :** Validez votre fichier YAML :
```bash
docker-compose config
# Si erreur de syntaxe, elle sera affichée
```

---

### 2.2 Le Conteneur Redémarre en Boucle

#### Symptôme

```bash
docker ps
# STATUS: Restarting (1) 5 seconds ago
```

---

#### Diagnostic

```bash
# Voir les logs (souvent répétitifs)
docker logs --tail 100 mariadb
```

---

#### Causes Courantes

##### Cause 1 : Erreur de Configuration

**Exemple :** Fichier `my.cnf` avec syntaxe invalide

**Solution :**
```bash
# Vérifier le fichier de config
cat config/my.cnf

# Désactiver temporairement
# Commenter le bind mount dans docker-compose.yml
# - ./config/my.cnf:/etc/mysql/conf.d/custom.cnf

# Redémarrer
docker-compose up -d
```

---

##### Cause 2 : Ressources Insuffisantes

**Message d'erreur (dans les logs) :**
```
Cannot allocate memory
```

**Solution :**
```yaml
# Réduire les ressources demandées ou augmenter celles disponibles
services:
  mariadb:
    deploy:
      resources:
        limits:
          memory: 512M  # Au lieu de 2G
```

Ou augmenter les ressources dans Docker Desktop (Settings → Resources).

---

### 2.3 Le Conteneur S'arrête Immédiatement

#### Symptôme

```bash
docker-compose up -d
# Démarre puis s'arrête immédiatement

docker ps -a
# STATUS: Exited (0) 1 second ago
```

---

#### Causes

##### Cause 1 : Commande d'Entrée Incorrecte

**Pour les images personnalisées :**
```dockerfile
# ❌ Commande qui se termine immédiatement
CMD echo "Hello"

# ✅ Commande qui reste active
CMD ["mysqld"]
```

---

##### Cause 2 : Conteneur de Test/Debug

Certains conteneurs sont conçus pour exécuter une commande puis se terminer :
```bash
# C'est normal pour :
docker run --rm alpine echo "test"
```

---

## 🔌 3. Problèmes de Connexion

### 3.1 Impossible de Se Connecter à la Base de Données

#### Depuis Votre Machine Hôte

**Symptôme :**
```bash
mysql -h localhost -u root -p
# ERROR 2003: Can't connect to MySQL server on 'localhost' (10061)
```

---

**Checklist de Diagnostic :**

```bash
# 1. Le conteneur est-il démarré ?
docker ps | grep mariadb

# 2. Le port est-il exposé ?
docker port mariadb_container
# Résultat attendu : 3306/tcp -> 0.0.0.0:3306

# 3. Le service écoute-t-il ?
docker exec mariadb_container netstat -tuln | grep 3306
```

---

**Solutions Courantes :**

##### Solution 1 : Port Non Exposé

```yaml
services:
  mariadb:
    # ❌ Oublié
    # ports:
    #   - "3306:3306"

    # ✅ Ajouter
    ports:
      - "3306:3306"
```

---

##### Solution 2 : Mauvais Port

```bash
# Si vous avez changé le port mapping
# docker-compose.yml :
# ports:
#   - "3307:3306"

# Se connecter sur le bon port
mysql -h localhost -P 3307 -u root -p
```

---

##### Solution 3 : Pare-feu Bloqué

**Windows :**
```powershell
# Vérifier si le port est bloqué
Test-NetConnection -ComputerName localhost -Port 3306

# Créer une règle (en administrateur)
New-NetFirewallRule -DisplayName "MariaDB" -Direction Inbound -Protocol TCP -LocalPort 3306 -Action Allow
```

**Linux :**
```bash
# Vérifier UFW
sudo ufw status

# Autoriser le port
sudo ufw allow 3306/tcp
```

---

#### Depuis un Autre Conteneur

**Symptôme :**
```bash
docker exec app_container ping mariadb_container
# ping: unknown host mariadb_container
```

---

**Checklist :**

```bash
# 1. Sont-ils sur le même réseau ?
docker network inspect mon_reseau
# Vérifier que les deux conteneurs apparaissent

# 2. Résolution DNS fonctionne-t-elle ?
docker exec app_container nslookup mariadb_container
```

---

**Solutions :**

##### Solution 1 : Pas sur le Même Réseau

```yaml
services:
  mariadb:
    networks:
      - backend  # ✅ Même réseau

  app:
    networks:
      - backend  # ✅ Même réseau

networks:
  backend:
```

---

##### Solution 2 : Mauvais Nom d'Hôte

```yaml
# Utiliser le NOM DU SERVICE (pas le container_name)
services:
  mariadb:  # ← NOM À UTILISER
    container_name: mariadb_dev

  app:
    environment:
      DB_HOST: mariadb  # ✅ Nom du service
      # Pas : DB_HOST: mariadb_dev  # ❌
```

---

### 3.2 Erreur "Access Denied"

**Symptôme :**
```bash
mysql -h localhost -u app_user -p
# ERROR 1045 (28000): Access denied for user 'app_user'@'localhost'
```

---

**Causes et Solutions :**

##### Cause 1 : Mauvais Mot de Passe

```bash
# Vérifier le mot de passe dans .env
cat .env | grep MYSQL_PASSWORD

# Réinitialiser si nécessaire
docker exec -it mariadb mariadb -u root -p
```

```sql
ALTER USER 'app_user'@'%' IDENTIFIED BY 'nouveau_password';
FLUSH PRIVILEGES;
```

---

##### Cause 2 : Utilisateur N'existe Pas

```bash
# Vérifier les utilisateurs
docker exec -it mariadb mariadb -u root -p
```

```sql
SELECT User, Host FROM mysql.user;

-- Si l'utilisateur n'existe pas, le créer
CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON app_db.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

---

##### Cause 3 : Mauvais Hôte

```sql
-- Utilisateur créé pour 'localhost' uniquement
CREATE USER 'app'@'localhost' IDENTIFIED BY 'pass';

-- Mais connexion depuis le réseau Docker (non-localhost)
-- Solution : Créer pour '%' ou IP spécifique
CREATE USER 'app'@'%' IDENTIFIED BY 'pass';
```

---

### 3.3 Connexion Lente ou Timeout

**Symptôme :**
```bash
mysql -h localhost -u root -p
# Connexion très lente (30+ secondes)
# Ou : ERROR 2003: Can't connect (timeout)
```

---

**Solutions :**

##### Solution 1 : Désactiver la Résolution DNS

```ini
# config/my.cnf
[mysqld]
skip-name-resolve
```

---

##### Solution 2 : Augmenter le Timeout

```bash
# Connexion avec timeout plus long
mysql -h localhost -u root -p --connect-timeout=60
```

---

## 💾 4. Problèmes de Volumes et Données

### 4.1 Données Perdues Après Redémarrage

**Symptôme :**
```
Vous créez des données dans la BDD
Vous arrêtez le conteneur (docker-compose down)
Vous le redémarrez
➡️ Toutes les données ont disparu !
```

---

**Cause :** Pas de volume configuré

**Solution :**

```yaml
# ❌ Sans volume
services:
  mariadb:
    image: mariadb

# ✅ Avec volume
services:
  mariadb:
    image: mariadb
    volumes:
      - mariadb_data:/var/lib/mysql  # ✅ OBLIGATOIRE

volumes:
  mariadb_data:
```

---

### 4.2 "Permission Denied" sur les Volumes

**Symptôme :**
```bash
docker logs mariadb
# mysqld: Can't create/write to file '/var/lib/mysql/...'
# (Errcode: 13 - Permission denied)
```

---

**Solutions :**

##### Solution 1 : Permissions du Dossier (Bind Mount)

```bash
# Vérifier les permissions
ls -la ./data

# Donner les permissions (développement uniquement)
chmod 777 ./data

# Ou changer le propriétaire
sudo chown -R 999:999 ./data  # UID MySQL/MariaDB
```

---

##### Solution 2 : Utiliser un Volume Nommé

```yaml
services:
  mariadb:
    volumes:
      # ✅ Volume nommé (pas de problèmes de permissions)
      - mariadb_data:/var/lib/mysql

      # Au lieu de bind mount
      # - ./data:/var/lib/mysql  # ⚠️ Peut poser problème

volumes:
  mariadb_data:
```

---

### 4.3 Volume Plein / Espace Disque Insuffisant

**Symptôme :**
```bash
docker logs mariadb
# ERROR: No space left on device
```

---

**Diagnostic :**

```bash
# Vérifier l'utilisation disque de Docker
docker system df

# Voir les gros volumes
docker system df -v | sort -k 3 -h

# Vérifier l'espace disque du système
df -h
```

---

**Solutions :**

##### Solution 1 : Nettoyer Docker

```bash
# Supprimer volumes inutilisés
docker volume prune

# Supprimer images inutilisées
docker image prune -a

# Nettoyage complet
docker system prune -a --volumes
```

---

##### Solution 2 : Déplacer les Données Docker

**Linux :** Modifier `/etc/docker/daemon.json`
```json
{
  "data-root": "/mnt/nouveau_disque/docker"
}
```

**Docker Desktop :** Settings → Resources → Disk image location

---

## 📝 5. Vérification des Logs

### 5.1 Accéder aux Logs

**Commandes de Base :**

```bash
# Tous les logs
docker logs mariadb_container

# Suivre en temps réel
docker logs -f mariadb_container

# Dernières 50 lignes
docker logs --tail 50 mariadb_container

# Avec timestamps
docker logs -t mariadb_container

# Depuis une date
docker logs --since 2024-10-29T10:00:00 mariadb_container

# Depuis X minutes
docker logs --since 30m mariadb_container
```

---

**Avec Docker Compose :**

```bash
# Logs de tous les services
docker-compose logs

# Logs d'un service spécifique
docker-compose logs mariadb

# Suivre en temps réel
docker-compose logs -f

# Dernières lignes
docker-compose logs --tail=100 -f
```

---

### 5.2 Comprendre les Logs MariaDB/MySQL

#### Logs de Démarrage Normaux

```
[Note] mysqld: ready for connections.
Version: '10.11.6-MariaDB'  socket: '/run/mysqld/mysqld.sock'  port: 3306
```
✅ **Bon signe :** Le serveur est prêt

---

#### Logs d'Erreur Courants

##### Erreur 1 : Initialisation

```
[ERROR] --initialize specified but the data directory has files in it. Aborting.
```

**Cause :** Volume contient déjà des données incompatibles

**Solution :**
```bash
docker-compose down
docker volume rm mariadb_data  # ⚠️ Supprime les données
docker-compose up -d
```

---

##### Erreur 2 : Corruption de Table

```
[ERROR] InnoDB: Database page corruption on disk or a failed file read
```

**Solution :**
```bash
# Tenter une réparation
docker exec -it mariadb mysqlcheck --all-databases --repair -u root -p

# Ou restaurer depuis backup
```

---

##### Erreur 3 : Mémoire Insuffisante

```
[ERROR] InnoDB: Cannot allocate memory for the buffer pool
```

**Solution :**
```yaml
# Réduire innodb_buffer_pool_size
services:
  mariadb:
    environment:
      - MYSQL_INNODB_BUFFER_POOL_SIZE=128M
```

---

### 5.3 Logs PostgreSQL

#### Logs Normaux

```
LOG:  database system is ready to accept connections
```

---

#### Erreurs Courantes

```
FATAL:  password authentication failed for user "postgres"
```
➡️ Mauvais mot de passe

```
FATAL:  database "mydb" does not exist
```
➡️ Base de données non créée

---

### 5.4 Sauvegarder les Logs

```bash
# Sauvegarder dans un fichier
docker logs mariadb > logs_mariadb_$(date +%Y%m%d).txt

# Logs avec compose
docker-compose logs > logs_complets.txt

# Logs datés (garder historique)
docker logs mariadb >> logs_historique.txt
```

---

## ⚡ 6. Diagnostic de Performances

### 6.1 Conteneur Lent

#### Symptôme

```
Les requêtes prennent beaucoup de temps
L'application est lente
Le conteneur semble "laggy"
```

---

#### Diagnostic

```bash
# 1. Vérifier l'utilisation des ressources
docker stats mariadb

# Résultat :
# CONTAINER   CPU %   MEM USAGE / LIMIT     MEM %
# mariadb     95%     1.8GiB / 2GiB         90%
```

---

#### Causes et Solutions

##### Cause 1 : Mémoire Saturée

**Indicateur :**
```
MEM % proche de 100%
Swap élevé
```

**Solutions :**

**Option 1 : Augmenter la limite mémoire**
```yaml
services:
  mariadb:
    deploy:
      resources:
        limits:
          memory: 4G  # Au lieu de 2G
```

**Option 2 : Optimiser la configuration**
```ini
# config/my.cnf
[mysqld]
# Réduire les buffers
innodb_buffer_pool_size = 512M
sort_buffer_size = 1M
join_buffer_size = 1M
```

---

##### Cause 2 : CPU Saturé

**Indicateur :**
```
CPU % proche de 100%
```

**Solutions :**

**Option 1 : Identifier les requêtes lentes**
```sql
-- Activer le log des requêtes lentes
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- Voir les requêtes
SHOW PROCESSLIST;
```

**Option 2 : Augmenter les CPUs**
```yaml
services:
  mariadb:
    deploy:
      resources:
        limits:
          cpus: '4'  # Au lieu de 2
```

---

##### Cause 3 : Disque Lent (I/O)

**Diagnostic :**
```bash
# Voir les I/O disque
docker stats --no-stream --format "table {{.Name}}\t{{.BlockIO}}"
```

**Solutions :**
- Utiliser un SSD
- Optimiser les requêtes (ajouter des index)
- Augmenter innodb_buffer_pool_size (plus de cache en RAM)

---

### 6.2 Requêtes Lentes

#### Identifier les Requêtes Lentes

**Activer le log dans MariaDB :**
```sql
-- Se connecter
docker exec -it mariadb mariadb -u root -p

-- Activer
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow-query.log';
SET GLOBAL long_query_time = 2;  -- 2 secondes
```

---

**Voir les processus en cours :**
```sql
SHOW FULL PROCESSLIST;

-- Résultat :
-- Id  | User | Command | Time | State         | Info
-- 123 | root | Query   | 45   | Sending data  | SELECT * FROM huge_table...
```

---

**Tuer une requête bloquée :**
```sql
KILL 123;  -- ID de la requête
```

---

#### Optimiser les Requêtes

```sql
-- Analyser une requête
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Si "type: ALL" → pas d'index, création nécessaire
CREATE INDEX idx_email ON users(email);

-- Revérifier
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
-- Devrait maintenant montrer "type: ref"
```

---

### 6.3 Problèmes de Réseau (Latence)

#### Diagnostic

```bash
# Tester la latence entre conteneurs
docker exec app ping -c 5 mariadb

# Résultat :
# 5 packets transmitted, 5 received, 0% packet loss, time 4ms
# rtt min/avg/max = 0.1/0.2/0.3 ms
```

---

**Latence élevée (> 10ms) ?**

**Causes possibles :**
- Réseaux Docker mal configurés
- Conteneurs sur des hôtes différents (Swarm)
- Problèmes de virtualisation (Windows/Mac)

**Solution :**
```yaml
# Utiliser le réseau host (performances maximales)
services:
  app:
    network_mode: "host"
```

⚠️ **Attention :** Moins de sécurité, pas d'isolation

---

### 6.4 Outils de Monitoring

#### Monitoring en Temps Réel

```bash
# Stats continues
watch -n 1 'docker stats --no-stream'

# Avec Docker Compose
docker-compose top
```

---

#### Outils Graphiques

**Portainer (interface web) :**
```yaml
services:
  portainer:
    image: portainer/portainer-ce
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

Accès : http://localhost:9000

---

**cAdvisor (métriques détaillées) :**
```yaml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
```

Accès : http://localhost:8080

---

## 🌐 7. Problèmes Réseau

### 7.1 Conteneurs Ne Peuvent Pas Communiquer

**Diagnostic :**
```bash
# Test de connectivité
docker exec app ping mariadb

# Si échec :
# 1. Vérifier les réseaux
docker network ls
docker network inspect mon_reseau

# 2. Vérifier la résolution DNS
docker exec app nslookup mariadb
```

---

**Solutions :**

##### Connecter au Même Réseau

```bash
# Via ligne de commande
docker network connect mon_reseau app
docker network connect mon_reseau mariadb

# Via docker-compose.yml
services:
  app:
    networks:
      - backend
  mariadb:
    networks:
      - backend

networks:
  backend:
```

---

### 7.2 Conflit d'IP

**Message d'erreur :**
```
Error response from daemon: Address already in use
```

**Solution :**
```bash
# Voir qui utilise l'IP
docker network inspect mon_reseau

# Arrêter le conteneur en conflit
docker stop conteneur_conflictuel

# Ou changer l'IP dans docker-compose.yml
services:
  mariadb:
    networks:
      backend:
        ipv4_address: 172.20.0.11  # Au lieu de .10
```

---

### 7.3 Réseau Non Trouvé

**Message d'erreur :**
```
network mon_reseau declared as external, but could not be found
```

**Solution :**
```bash
# Créer le réseau
docker network create --subnet=172.20.0.0/16 mon_reseau

# Puis lancer
docker-compose up -d
```

---

## 🗄️ 8. Erreurs Spécifiques par BDD

### 8.1 MariaDB / MySQL

#### Erreur : "Table doesn't exist"

```sql
ERROR 1146: Table 'mydb.users' doesn't exist
```

**Solutions :**
```sql
-- Vérifier que la base est sélectionnée
USE mydb;

-- Lister les tables
SHOW TABLES;

-- Si la table manque, la créer
CREATE TABLE users (...);

-- Ou restaurer depuis backup
```

---

#### Erreur : "Too many connections"

```
ERROR 1040: Too many connections
```

**Solutions :**

**Temporaire :**
```sql
SET GLOBAL max_connections = 500;
```

**Permanent :**
```ini
# config/my.cnf
[mysqld]
max_connections = 500
```

---

### 8.2 PostgreSQL

#### Erreur : "role does not exist"

```
FATAL: role "myuser" does not exist
```

**Solution :**
```bash
docker exec -it postgres psql -U postgres
```

```sql
CREATE ROLE myuser WITH LOGIN PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```

---

#### Erreur : "too many clients"

```
FATAL: sorry, too many clients already
```

**Solution :**
```bash
# Dans docker-compose.yml
services:
  postgres:
    command: postgres -c max_connections=200
```

---

### 8.3 MongoDB

#### Erreur : "Authentication failed"

```
MongoError: Authentication failed
```

**Solution :**
```bash
docker exec -it mongo mongo
```

```javascript
use admin
db.createUser({
  user: "myuser",
  pwd: "password",
  roles: ["readWrite"]
})
```

---

### 8.4 Redis

#### Erreur : "NOAUTH Authentication required"

```
(error) NOAUTH Authentication required
```

**Solution :**
```bash
# Se connecter avec le mot de passe
docker exec -it redis redis-cli -a password

# Ou dans l'application
redis://password@redis:6379
```

---

## 🛠️ 9. Outils de Diagnostic

### 9.1 Outils en Ligne de Commande

#### netcat (nc) - Test de Port

```bash
# Installer dans un conteneur
docker exec -it app sh -c "apk add netcat-openbsd"

# Tester un port
docker exec app nc -zv mariadb 3306

# Résultat si OK :
# mariadb (172.20.0.10:3306) open
```

---

#### curl - Test HTTP

```bash
# Tester une API
docker exec app curl -I http://api:8080/health

# Résultat attendu :
# HTTP/1.1 200 OK
```

---

#### dig / nslookup - Test DNS

```bash
# Résoudre un nom
docker exec app nslookup mariadb

# Avec dig (plus détaillé)
docker exec app dig mariadb
```

---

### 9.2 Conteneur de Debug

**Créer un conteneur avec tous les outils :**

```bash
# Lancer un conteneur debug sur le réseau
docker run -it --rm \
  --network mon_reseau \
  nicolaka/netshoot

# Outils disponibles :
# - ping, curl, wget
# - nslookup, dig
# - netcat, telnet
# - traceroute, mtr
# - tcpdump, netstat
```

---

### 9.3 Scripts de Diagnostic Automatisés

**Créer un fichier `diagnose.sh` :**

```bash
#!/bin/bash

echo "=== Diagnostic Docker ==="
echo ""

echo "📦 Conteneurs:"
docker ps -a

echo ""
echo "🌐 Réseaux:"
docker network ls

echo ""
echo "💾 Volumes:"
docker volume ls

echo ""
echo "📊 Utilisation disque:"
docker system df

echo ""
echo "⚡ Ressources:"
docker stats --no-stream

echo ""
echo "📝 Logs récents (MariaDB):"
docker logs --tail 20 mariadb 2>&1

echo ""
echo "=== Fin du diagnostic ==="
```

**Utilisation :**
```bash
chmod +x diagnose.sh
./diagnose.sh > diagnostic_$(date +%Y%m%d).txt
```

---

## 📊 Tableaux de Référence

### Codes d'Exit Courants

| Code | Signification | Action |
|------|---------------|--------|
| 0 | Sortie normale | ✅ OK |
| 1 | Erreur générale | Voir les logs |
| 125 | Erreur Docker | Commande invalide |
| 126 | Commande non exécutable | Problème de permissions |
| 127 | Commande introuvable | Chemin incorrect |
| 137 | Tué (SIGKILL) | OOM (Out of Memory) |
| 139 | Segmentation fault | Bug logiciel |
| 143 | Arrêt normal (SIGTERM) | ✅ OK |

---

### Messages d'Erreur Courants

| Erreur | Cause | Solution Rapide |
|--------|-------|-----------------|
| `bind: address already in use` | Port occupé | Changer le port |
| `network not found` | Réseau manquant | Créer le réseau |
| `permission denied` | Permissions volumes | `chmod 777` (dev) |
| `no space left` | Disque plein | `docker system prune` |
| `Cannot allocate memory` | RAM insuffisante | Augmenter limites |
| `Connection refused` | Service non démarré | Vérifier logs |
| `Access denied` | Mauvais credentials | Vérifier .env |

---

### Commandes de Récupération d'Urgence

```bash
# Arrêt brutal
docker-compose kill

# Redémarrage forcé
docker-compose up -d --force-recreate

# Réinitialisation complète (⚠️ perte de données)
docker-compose down -v
docker system prune -a --volumes -f
docker-compose up -d

# Backup d'urgence
docker exec mariadb mysqldump -u root -p --all-databases > emergency_backup.sql
```

---

## 💡 Conseils de Dépannage

### Règles d'Or

1. **Toujours commencer par les logs**
   ```bash
   docker logs <conteneur>
   ```

2. **Vérifier l'état avant tout**
   ```bash
   docker ps -a
   ```

3. **Sauvegarder avant d'agir**
   ```bash
   ./backup-volume.sh mon_volume
   ```

4. **Tester une chose à la fois**
   ```
   Changement 1 → Test → OK/KO
   Changement 2 → Test → OK/KO
   ```

5. **Documenter le problème et la solution**
   ```
   Problème : ...
   Cause : ...
   Solution : ...
   ```

---

### Workflow en Cas de Panique

```
1. 🛑 STOP : Ne pas paniquer
    ↓
2. 📸 Capturer l'état actuel
    docker ps -a > etat.txt
    docker logs conteneur > logs.txt
    ↓
3. 💾 Sauvegarder si critique
    ./backup-volume.sh
    ↓
4. 🔍 Diagnostiquer méthodiquement
    (Suivre la méthodologie de cette annexe)
    ↓
5. 🔧 Appliquer UNE solution
    ↓
6. ✅ Vérifier le résultat
    ↓
7. 📝 Documenter
```

---

## 🆘 Quand Demander de l'Aide

### Avant de Poster sur un Forum

**Informations à Fournir :**

1. **Contexte**
   - Système d'exploitation
   - Version Docker (`docker --version`)
   - Version Docker Compose

2. **Configuration**
   - Fichier `docker-compose.yml`
   - Fichiers de config (my.cnf, etc.)

3. **Symptôme**
   - Description précise du problème
   - Quand ça a commencé
   - Ce qui a changé

4. **Logs**
   ```bash
   docker logs conteneur > logs.txt
   ```

5. **Tentatives de résolution**
   - Ce que vous avez déjà essayé
   - Résultats obtenus

---

### Ressources d'Aide

- 📖 [Docker Forums](https://forums.docker.com/)
- 💬 [Stack Overflow - Docker](https://stackoverflow.com/questions/tagged/docker)
- 💬 [Reddit - r/docker](https://www.reddit.com/r/docker/)
- 📖 [Docker Documentation](https://docs.docker.com/)
- 📖 Documentation spécifique de chaque BDD

---

## ✅ Checklist de Dépannage

### Checklist Générale

- [ ] Le conteneur est-il démarré ? (`docker ps`)
- [ ] Y a-t-il des erreurs dans les logs ? (`docker logs`)
- [ ] Les ports sont-ils correctement mappés ? (`docker port`)
- [ ] Les volumes sont-ils montés ? (`docker inspect`)
- [ ] Les réseaux sont-ils configurés ? (`docker network ls`)
- [ ] Les ressources sont-elles suffisantes ? (`docker stats`)
- [ ] Les fichiers de config sont-ils valides ?
- [ ] Les mots de passe sont-ils corrects ?
- [ ] Le pare-feu ne bloque-t-il pas ?
- [ ] Y a-t-il suffisamment d'espace disque ?

---

### Checklist Avant de Redémarrer en Production

- [ ] Sauvegardes récentes effectuées
- [ ] Tests en environnement de dev
- [ ] Fenêtre de maintenance planifiée
- [ ] Équipe avertie
- [ ] Rollback plan préparé
- [ ] Monitoring en place
- [ ] Documentation à jour

---

## 🎓 Conclusion

Le dépannage est une compétence qui s'acquiert avec la pratique. Plus vous rencontrerez de problèmes, plus vous deviendrez efficace pour les résoudre.

**Points clés à retenir :**

1. **Méthodologie > Intuition** : Suivez une approche systématique
2. **Logs = Vérité** : Ils contiennent presque toujours la réponse
3. **Un problème à la fois** : Ne changez pas 10 choses simultanément
4. **Documentez** : Pour vous et les autres
5. **Sauvegardez** : Avant toute action destructrice

---

## 🚀 Pour Aller Plus Loin

### Annexes Connexes

- **[Annexe A - Commandes](A-reference-commandes.md)** - Toutes les commandes de diagnostic
- **[Annexe B - Réseaux](B-gestion-reseaux.md)** - Dépannage réseau
- **[Annexe C - Volumes](C-gestion-volumes.md)** - Problèmes de volumes
- **[Annexe D - Sécurité](D-securite-bonnes-pratiques.md)** - Sécuriser après dépannage

### Outils Avancés

- [Docker Bench Security](https://github.com/docker/docker-bench-security) - Audit de sécurité
- [Dive](https://github.com/wagoodman/dive) - Explorer les couches d'images
- [Lazydocker](https://github.com/jesseduffield/lazydocker) - Interface TUI pour Docker

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Le meilleur débogueur est une bonne nuit de sommeil... mais les logs aident aussi ! 🐛💤*
