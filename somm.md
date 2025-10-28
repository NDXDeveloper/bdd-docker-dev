# Table des Matières - Bases de Données avec Docker

## 📋 Introduction
- À propos de ce repository
- Prérequis et installation
- Concepts Docker de base
- Bonnes pratiques

---

## 🗄️ Bases de Données Relationnelles

### 1. MariaDB
- 1.1 Configuration basique avec docker-compose
- 1.2 Configuration avec fichier my.cnf
- 1.3 Configuration avec IP fixe
- 1.4 Gestion des utilisateurs et permissions
- 1.5 Accès depuis le réseau local

### 2. PostgreSQL
- 2.1 Configuration basique avec docker-compose
- 2.2 Configuration avancée avec postgresql.conf
- 2.3 Configuration avec IP fixe
- 2.4 Gestion des utilisateurs et bases de données
- 2.5 Configuration avec pgAdmin

### 3. Microsoft SQL Server
- 3.1 Configuration basique avec docker-compose
- 3.2 Configuration avec IP fixe
- 3.3 Gestion des utilisateurs et logins
- 3.4 Restauration de backup

### 4. SQLite
- 4.1 Utilisation de SQLite en conteneur
- 4.2 SQLite avec volume persistant
- 4.3 Outils graphiques (SQLite Browser)

---

## 🔥 Bases de Données NoSQL

### 5. MongoDB
- 5.1 Configuration basique avec docker-compose
- 5.2 Configuration avec authentification
- 5.3 Configuration avec IP fixe
- 5.4 MongoDB avec Mongo Express
- 5.5 Replica Set simple

### 6. Redis
- 6.1 Configuration basique avec docker-compose
- 6.2 Configuration avec mot de passe
- 6.3 Configuration avec redis.conf
- 6.4 Configuration avec IP fixe
- 6.5 Redis avec Redis Commander

### 7. Cassandra
- 7.1 Configuration basique d'un nœud unique
- 7.2 Configuration avec IP fixe
- 7.3 Cluster Cassandra simple (3 nœuds)
- 7.4 Gestion des keyspaces et tables

### 8. Neo4j
- 8.1 Configuration basique avec docker-compose
- 8.2 Configuration avec IP fixe
- 8.3 Configuration avancée avec plugins
- 8.4 Import de données CSV

### 9. InfluxDB
- 9.1 Configuration basique v2.x avec docker-compose
- 9.2 Configuration avec IP fixe
- 9.3 InfluxDB avec Grafana
- 9.4 Configuration avec Telegraf

### 10. DynamoDB Local
- 10.1 Configuration basique avec docker-compose
- 10.2 Configuration avec IP fixe
- 10.3 DynamoDB avec Admin UI
- 10.4 Utilisation avec AWS CLI

### 11. Elasticsearch
- 11.1 Configuration basique d'un nœud unique
- 11.2 Configuration avec IP fixe
- 11.3 Elasticsearch avec Kibana
- 11.4 Cluster Elasticsearch simple (3 nœuds)

---

## 📚 Annexes

### A. Référence des Commandes
- Commandes Docker essentielles
- Commandes Docker Compose
- Commandes de débogage

### B. Gestion des Réseaux Docker
- Création de réseaux personnalisés
- Attribution d'IP fixes
- Communication entre conteneurs

### C. Gestion des Volumes
- Volumes nommés vs bind mounts
- Sauvegarde et restauration
- Nettoyage des volumes

### D. Sécurité et Bonnes Pratiques
- Gestion des mots de passe
- Fichiers .env
- Isolation réseau
- Mises à jour de sécurité

### E. Dépannage
- Problèmes courants et solutions
- Vérification des logs
- Diagnostic de performances

### F. Outils de Gestion
- Clients GUI pour chaque BDD
- Outils de monitoring
- Outils de backup

### G. Comparaison des BDD
- Quand utiliser quelle base de données
- Tableau comparatif
- Cas d'usage typiques

---

## 🚀 Cas Pratiques

### Stack LAMP
- Apache + MariaDB + PHP

### Stack MEAN
- MongoDB + Express + Angular + Node

### Stack ELK
- Elasticsearch + Logstash + Kibana

### Environnement de Dev Complet
- Multi-BDD pour tests

### Migration de Données
- Entre différents SGBD

---

*Dernière mise à jour : Octobre 2025*
