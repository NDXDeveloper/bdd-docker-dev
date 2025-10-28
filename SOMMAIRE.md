# Table des Matières - Bases de Données avec Docker

## 📋 Introduction
- [À propos de ce repository](README.md)
- [Prérequis et installation](00-introduction/01-prerequis.md)
- [Concepts Docker de base](00-introduction/02-concepts-docker.md)
- [Bonnes pratiques](00-introduction/03-bonnes-pratiques.md)

---

## 🗄️ Bases de Données Relationnelles

### 1. [MariaDB](01-mariadb/README.md)
- 1.1 [Configuration basique avec docker-compose](01-mariadb/01-config-basique-docker-compose.md)
- 1.2 [Configuration avec fichier my.cnf](01-mariadb/02-config-avec-mycnf.md)
- 1.3 [Configuration avec IP fixe](01-mariadb/03-config-ip-fixe.md)
- 1.4 [Gestion des utilisateurs et permissions](01-mariadb/04-gestion-utilisateurs.md)
- 1.5 [Accès depuis le réseau local](01-mariadb/05-acces-reseau-local.md)

### 2. [PostgreSQL](02-postgresql/README.md)
- 2.1 [Configuration basique avec docker-compose](02-postgresql/01-config-basique-docker-compose.md)
- 2.2 [Configuration avancée avec postgresql.conf](02-postgresql/02-config-avec-postgresqlconf.md)
- 2.3 [Configuration avec IP fixe](02-postgresql/03-config-ip-fixe.md)
- 2.4 [Gestion des utilisateurs et bases de données](02-postgresql/04-gestion-utilisateurs-bdd.md)
- 2.5 [Configuration avec pgAdmin](02-postgresql/05-config-avec-pgadmin.md)

### 3. [Microsoft SQL Server](03-mssql/README.md)
- 3.1 [Configuration basique avec docker-compose](03-mssql/01-config-basique-docker-compose.md)
- 3.2 [Configuration avec IP fixe](03-mssql/02-config-ip-fixe.md)
- 3.3 [Gestion des utilisateurs et logins](03-mssql/03-gestion-utilisateurs-logins.md)
- 3.4 [Restauration de backup](03-mssql/04-restauration-backup.md)

### 4. [SQLite](04-sqlite/README.md)
- 4.1 [Utilisation de SQLite en conteneur](04-sqlite/01-sqlite-conteneur.md)
- 4.2 [SQLite avec volume persistant](04-sqlite/02-sqlite-volume-persistant.md)
- 4.3 [Outils graphiques (SQLite Browser)](04-sqlite/03-outils-graphiques.md)

---

## 🔥 Bases de Données NoSQL

### 5. [MongoDB](05-mongodb/README.md)
- 5.1 [Configuration basique avec docker-compose](05-mongodb/01-config-basique-docker-compose.md)
- 5.2 [Configuration avec authentification](05-mongodb/02-config-avec-authentification.md)
- 5.3 [Configuration avec IP fixe](05-mongodb/03-config-ip-fixe.md)
- 5.4 [MongoDB avec Mongo Express](05-mongodb/04-mongodb-mongo-express.md)
- 5.5 [Replica Set simple](05-mongodb/05-replica-set-simple.md)

### 6. [Redis](06-redis/README.md)
- 6.1 [Configuration basique avec docker-compose](06-redis/01-config-basique-docker-compose.md)
- 6.2 [Configuration avec mot de passe](06-redis/02-config-avec-password.md)
- 6.3 [Configuration avec redis.conf](06-redis/03-config-avec-redisconf.md)
- 6.4 [Configuration avec IP fixe](06-redis/04-config-ip-fixe.md)
- 6.5 [Redis avec Redis Commander](06-redis/05-redis-commander.md)

### 7. [Cassandra](07-cassandra/README.md)
- 7.1 [Configuration basique d'un nœud unique](07-cassandra/01-config-noeud-unique.md)
- 7.2 [Configuration avec IP fixe](07-cassandra/02-config-ip-fixe.md)
- 7.3 [Cluster Cassandra simple (3 nœuds)](07-cassandra/03-cluster-simple.md)
- 7.4 [Gestion des keyspaces et tables](07-cassandra/04-gestion-keyspaces.md)

### 8. [Neo4j](08-neo4j/README.md)
- 8.1 [Configuration basique avec docker-compose](08-neo4j/01-config-basique-docker-compose.md)
- 8.2 [Configuration avec IP fixe](08-neo4j/02-config-ip-fixe.md)
- 8.3 [Configuration avancée avec plugins](08-neo4j/03-config-avec-plugins.md)
- 8.4 [Import de données CSV](08-neo4j/04-import-donnees-csv.md)

### 9. [InfluxDB](09-influxdb/README.md)
- 9.1 [Configuration basique v2.x avec docker-compose](09-influxdb/01-config-basique-v2.md)
- 9.2 [Configuration avec IP fixe](09-influxdb/02-config-ip-fixe.md)
- 9.3 [InfluxDB avec Grafana](09-influxdb/03-influxdb-grafana.md)
- 9.4 [Configuration avec Telegraf](09-influxdb/04-config-telegraf.md)

### 10. [DynamoDB Local](10-dynamodb/README.md)
- 10.1 [Configuration basique avec docker-compose](10-dynamodb/01-config-basique-docker-compose.md)
- 10.2 [Configuration avec IP fixe](10-dynamodb/02-config-ip-fixe.md)
- 10.3 [DynamoDB avec Admin UI](10-dynamodb/03-dynamodb-admin-ui.md)
- 10.4 [Utilisation avec AWS CLI](10-dynamodb/04-utilisation-aws-cli.md)

### 11. [Elasticsearch](11-elasticsearch/README.md)
- 11.1 [Configuration basique d'un nœud unique](11-elasticsearch/01-config-noeud-unique.md)
- 11.2 [Configuration avec IP fixe](11-elasticsearch/02-config-ip-fixe.md)
- 11.3 [Elasticsearch avec Kibana](11-elasticsearch/03-elasticsearch-kibana.md)
- 11.4 [Cluster Elasticsearch simple (3 nœuds)](11-elasticsearch/04-cluster-simple.md)

---

## 📚 Annexes

### A. [Référence des Commandes](annexes/A-reference-commandes.md)
- Commandes Docker essentielles
- Commandes Docker Compose
- Commandes de débogage

### B. [Gestion des Réseaux Docker](annexes/B-gestion-reseaux.md)
- Création de réseaux personnalisés
- Attribution d'IP fixes
- Communication entre conteneurs

### C. [Gestion des Volumes](annexes/C-gestion-volumes.md)
- Volumes nommés vs bind mounts
- Sauvegarde et restauration
- Nettoyage des volumes

### D. [Sécurité et Bonnes Pratiques](annexes/D-securite-bonnes-pratiques.md)
- Gestion des mots de passe
- Fichiers .env
- Isolation réseau
- Mises à jour de sécurité

### E. [Dépannage](annexes/E-depannage.md)
- Problèmes courants et solutions
- Vérification des logs
- Diagnostic de performances

### F. [Outils de Gestion](annexes/F-outils-gestion.md)
- Clients GUI pour chaque BDD
- Outils de monitoring
- Outils de backup

### G. [Comparaison des BDD](annexes/G-comparaison-bdd.md)
- Quand utiliser quelle base de données
- Tableau comparatif
- Cas d'usage typiques

---

## 🚀 Cas Pratiques

### [Stack LAMP](cas-pratiques/01-stack-lamp.md)
- Apache + MariaDB + PHP

### [Stack MEAN](cas-pratiques/02-stack-mean.md)
- MongoDB + Express + Angular + Node

### [Stack ELK](cas-pratiques/03-stack-elk.md)
- Elasticsearch + Logstash + Kibana

### [Environnement de Dev Complet](cas-pratiques/04-env-dev-complet.md)
- Multi-BDD pour tests

### [Migration de Données](cas-pratiques/05-migration-donnees.md)
- Entre différents SGBD

---

*Dernière mise à jour : Octobre 2025*
