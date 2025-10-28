# 🐳 Bases de Données avec Docker pour Développeurs

![License](https://img.shields.io/badge/License-CC%20BY%204.0-blue.svg)
![Docker](https://img.shields.io/badge/Docker-20.10%2B-blue.svg)
![Databases](https://img.shields.io/badge/Bases-12%20BDD-green.svg)
![Language](https://img.shields.io/badge/Langue-Français-blue.svg)

**Guide complet pour déployer et configurer rapidement vos bases de données de développement avec Docker.**

![Docker Logo](https://www.docker.com/wp-content/uploads/2022/03/Moby-logo.png)

---

## 📖 Table des matières

- [À propos](#-à-propos)
- [Bases de données couvertes](#-bases-de-données-couvertes)
- [Installation rapide](#-installation-rapide)
- [Utilisation](#-utilisation)
- [Structure](#-structure-du-projet)
- [Licence](#-licence)
- [Contact](#-contact)

---

## 📋 À propos

**Le problème :** Installer et configurer une base de données pour le développement prend du temps, pollue votre système et peut créer des conflits de versions.

**La solution :** Docker ! Ce guide vous fournit des configurations prêtes à l'emploi pour démarrer n'importe quelle base de données en quelques secondes, avec des environnements isolés et reproductibles.

**✨ Points clés :**
- 🗄️ **12 bases de données** (SQL et NoSQL)
- ⚡ **Installation en < 5 minutes** par BDD
- 🔧 **Multiples configurations** (basique, IP fixe, avec GUI)
- 🔐 **Gestion des utilisateurs** et permissions
- 🌐 **Accès réseau local** configuré
- 📚 **7 annexes de référence** complètes
- 🚀 **5 cas pratiques** (stacks complètes)
- 🇫🇷 **En français** et gratuit (CC BY 4.0)

**Idéal pour :** Développeurs, étudiants, formateurs, tests d'applications

---

## 🗄️ Bases de données couvertes

### SQL (Relationnelles)
| Base | Version | Fiches | Use Case Principal |
|------|---------|--------|-------------------|
| 🐬 **MariaDB** | 10.11+ | 5 | Alternative MySQL open source |
| 🐘 **PostgreSQL** | 15+ | 5 | BDD relationnelle avancée |
| 🔷 **MS SQL Server** | 2022 | 4 | Environnements .NET |
| 📦 **SQLite** | 3.x | 3 | BDD embarquée légère |

### NoSQL
| Base | Type | Fiches | Use Case Principal |
|------|------|--------|-------------------|
| 🍃 **MongoDB** | Document | 5 | Applications web modernes |
| 🔴 **Redis** | Clé-Valeur | 5 | Cache et temps réel |
| 🔶 **Cassandra** | Colonnes | 4 | Big Data distribué |
| 🌐 **Neo4j** | Graphe | 4 | Réseaux sociaux, graphes |
| 📊 **InfluxDB** | Séries temporelles | 4 | Métriques et IoT |
| ⚡ **DynamoDB** | Document | 4 | Compatible AWS |
| 🔍 **Elasticsearch** | Recherche | 4 | Moteur de recherche |

**Total : 48 fiches de configuration** couvrant tous les cas d'usage !

---

## 🚀 Installation rapide

### Prérequis

```bash
# Vérifier Docker
docker --version
# Minimum requis : Docker 20.10+

# Vérifier Docker Compose
docker-compose --version
# Minimum requis : Docker Compose 2.0+
```

**Installer Docker :**
- 🪟 Windows : [Docker Desktop](https://www.docker.com/products/docker-desktop)
- 🍎 macOS : [Docker Desktop](https://www.docker.com/products/docker-desktop)
- 🐧 Linux : `curl -fsSL https://get.docker.com | sh`

### Exemple : Démarrer PostgreSQL en 30 secondes

```bash
# 1. Créer le fichier docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: postgres_dev
    environment:
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
EOF

# 2. Démarrer
docker-compose up -d

# 3. Connecter
docker exec -it postgres_dev psql -U postgres

# 🎉 C'est prêt !
```

**Se connecter depuis un client :**
- **Hôte :** `localhost` ou `127.0.0.1`
- **Port :** `5432`
- **Utilisateur :** `postgres`
- **Mot de passe :** `dev_password`

---

## 🎯 Utilisation

### Consultation du guide complet

📚 **[SOMMAIRE.md](SOMMAIRE.md)** - Table des matières détaillée

### Démarrage selon votre besoin

| Vous voulez... | Consultez... |
|----------------|--------------|
| 🆕 Démarrer une BDD rapidement | Les fiches `01-config-basique-*` |
| 🔧 Configuration avancée | Les fiches `02-config-avec-*` et suivantes |
| 🌐 IP fixe pour conteneur | Les fiches `*-config-ip-fixe` |
| 👥 Gérer les utilisateurs | Les fiches `*-gestion-utilisateurs` |
| 🖥️ Interface graphique | Les fiches `*-avec-[GUI]` |
| 📚 Référence commandes | [Annexe A](annexes/A-reference-commandes.md) |
| 🚀 Stack complète | [Cas pratiques](cas-pratiques/) |

### Workflow recommandé

```bash
# 1. Choisir votre BDD
cd 02-postgresql  # Par exemple

# 2. Lire le README de la BDD
cat README.md

# 3. Suivre une fiche
cat 01-config-basique-docker-compose.md

# 4. Copier le docker-compose.yml
# 5. Personnaliser (mots de passe, ports...)
# 6. Lancer : docker-compose up -d
```

---

## 📁 Structure du projet

```
bdd-docker-dev/
├── README.md                    # Ce fichier
├── SOMMAIRE.md                  # Table des matières détaillée
├── LICENSE                      # Licence CC BY 4.0
│
├── 00-introduction/             # Concepts de base
├── 01-mariadb/                  # 5 fiches MariaDB
├── 02-postgresql/               # 5 fiches PostgreSQL
├── 03-mssql/                    # 4 fiches MS SQL Server
├── 04-sqlite/                   # 3 fiches SQLite
├── 05-mongodb/                  # 5 fiches MongoDB
├── 06-redis/                    # 5 fiches Redis
├── 07-cassandra/                # 4 fiches Cassandra
├── 08-neo4j/                    # 4 fiches Neo4j
├── 09-influxdb/                 # 4 fiches InfluxDB
├── 10-dynamodb/                 # 4 fiches DynamoDB
├── 11-elasticsearch/            # 4 fiches Elasticsearch
│
├── annexes/                     # 7 annexes de référence
│   ├── A-reference-commandes.md
│   ├── B-gestion-reseaux.md
│   ├── C-gestion-volumes.md
│   ├── D-securite-bonnes-pratiques.md
│   ├── E-depannage.md
│   ├── F-outils-gestion.md
│   └── G-comparaison-bdd.md
│
└── cas-pratiques/               # 5 stacks complètes
    ├── 01-stack-lamp.md         # Apache + MariaDB + PHP
    ├── 02-stack-mean.md         # MongoDB + Express + Node
    ├── 03-stack-elk.md          # Elasticsearch + Logstash + Kibana
    ├── 04-env-dev-complet.md    # Multi-BDD
    └── 05-migration-donnees.md  # Migration entre BDD
```

---

## 💡 Cas d'usage typiques

### 🎓 Étudiant / Apprenant
```bash
# Tester différentes BDD pour un cours
docker-compose up -d  # MariaDB pour SQL
docker-compose up -d  # MongoDB pour NoSQL
# Suppression propre après : docker-compose down -v
```

### 👨‍💻 Développeur Full-Stack
```bash
# Stack complète MEAN
# MongoDB + Express + Angular + Node
# Voir : cas-pratiques/02-stack-mean.md
```

### 🧪 Tests d'Application
```bash
# Créer un environnement de test isolé
# Plusieurs versions de PostgreSQL en parallèle
# Sur des ports différents (5432, 5433, 5434...)
```

### 🏢 Équipe de Dev
```bash
# Partager les docker-compose.yml via Git
# Toute l'équipe a le même environnement
# Reproductibilité garantie
```

---

## ⚡ Commandes essentielles

```bash
# Démarrer une BDD
docker-compose up -d

# Voir les logs
docker-compose logs -f

# Se connecter au conteneur
docker exec -it <nom_conteneur> bash

# Arrêter (sans supprimer les données)
docker-compose stop

# Redémarrer
docker-compose start

# Supprimer (ATTENTION : supprime les données)
docker-compose down -v

# Vérifier l'état
docker-compose ps

# Voir les réseaux
docker network ls

# Voir les volumes
docker volume ls
```

💡 **Astuce :** Consultez l'[Annexe A](annexes/A-reference-commandes.md) pour la liste complète !

---

## 🎨 Fonctionnalités par BDD

| Fonctionnalité | MariaDB | PostgreSQL | MongoDB | Redis | Neo4j | Autres |
|----------------|---------|------------|---------|-------|-------|--------|
| ✅ Config basique | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 🔧 Config avancée | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 🌐 IP fixe | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 👥 Gestion users | ✅ | ✅ | ✅ | ✅ | ✅ | Selon BDD |
| 🖥️ Interface GUI | phpMyAdmin | pgAdmin | Mongo Express | Redis Commander | Browser | Selon BDD |
| 🔄 Réplication | ❌ | ❌ | ✅ | ❌ | ❌ | Selon BDD |
| 📦 Cluster | ❌ | ❌ | ❌ | ❌ | ❌ | Cassandra, Elastic |

---

## ❓ FAQ rapide

**Q : Docker ralentit-il les performances de la BDD ?**
R : Impact minimal (< 5%) pour le développement. Production : utiliser des BDD natives.

**Q : Puis-je utiliser ces configs en production ?**
R : Non, elles sont optimisées pour le développement. Production = configurations durcies.

**Q : Comment sauvegarder mes données ?**
R : Les volumes Docker persistent les données. Voir [Annexe C](annexes/C-gestion-volumes.md).

**Q : Plusieurs versions de la même BDD ?**
R : Oui ! Utilisez des ports et noms de conteneurs différents.

**Q : Comment gérer les mots de passe ?**
R : Utilisez des fichiers `.env`. Voir [Annexe D](annexes/D-securite-bonnes-pratiques.md).

**Q : Problème de connexion depuis un client ?**
R : Vérifiez ports, mots de passe, permissions. Voir [Annexe E](annexes/E-depannage.md).

---

## 🛡️ Sécurité

**⚠️ IMPORTANT - Ce guide est pour le DÉVELOPPEMENT uniquement**

🚫 **NE JAMAIS utiliser ces configurations en production sans :**
- Changer TOUS les mots de passe par défaut
- Configurer un pare-feu
- Activer le chiffrement SSL/TLS
- Restreindre les accès réseau
- Mettre en place des backups
- Suivre les bonnes pratiques de sécurité

📖 Consultez l'[Annexe D - Sécurité](annexes/D-securite-bonnes-pratiques.md) pour les détails.

---

## 🗺️ Roadmap

- ✅ 12 bases de données couvertes
- ✅ Configurations multiples par BDD
- ✅ 7 annexes de référence
- ✅ 5 cas pratiques
- 🔜 Vidéos tutoriels
- 🔜 Scripts d'automatisation
- 🔜 Templates de projets

---

## 📝 Licence

Ce projet est sous licence **CC BY 4.0** (Creative Commons Attribution 4.0 International).

✅ Libre d'utiliser, modifier, partager (même commercialement) avec attribution.

**Attribution :**
```
Bases de Données avec Docker par Nicolas DEOUX
https://github.com/NDXDeveloper/bdd-docker-dev
Licence CC BY 4.0
```

---

## 👨‍💻 Contact

**Nicolas DEOUX**
- 📧 [NDXDev@gmail.com](mailto:NDXDev@gmail.com)
- 🐙 [GitHub @NDXDeveloper](https://github.com/NDXDeveloper)

---

## 🙏 Remerciements

Merci à la communauté Docker, aux mainteneurs des images officielles, et à tous les développeurs qui partagent leurs connaissances ! 🎉

**Ressources complémentaires :**
[Docker Docs](https://docs.docker.com/) • [Docker Hub](https://hub.docker.com/) • [Awesome Docker](https://github.com/veggiemonk/awesome-docker)

---

<div align="center">

**🐳 Bon développement avec Docker ! 🚀**

[![Star on GitHub](https://img.shields.io/github/stars/NDXDeveloper/bdd-docker-dev?style=social)](https://github.com/NDXDeveloper/bdd-docker-dev)
[![Follow](https://img.shields.io/github/followers/NDXDeveloper?style=social)](https://github.com/NDXDeveloper)

**[⬆ Retour en haut](#-bases-de-données-avec-docker-pour-développeurs)**

*Dernière mise à jour : Octobre 2025*

</div>
