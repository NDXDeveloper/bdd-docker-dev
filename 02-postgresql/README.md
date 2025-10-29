🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

# 🐘 PostgreSQL avec Docker

## 📋 Table des matières

- [Vue d'ensemble](#-vue-densemble)
- [Pourquoi PostgreSQL ?](#-pourquoi-postgresql-)
- [Versions disponibles](#-versions-disponibles)
- [Guides disponibles](#-guides-disponibles)
- [Architecture recommandée](#-architecture-recommandée)
- [Prérequis](#-prérequis)
- [Points clés à connaître](#-points-clés-à-connaître)
- [Cas d'usage typiques](#-cas-dusage-typiques)
- [Ressources complémentaires](#-ressources-complémentaires)

---

## 🎯 Vue d'ensemble

**PostgreSQL** (souvent appelé "Postgres") est un système de gestion de base de données relationnelle objet (SGBDRO) open source, réputé pour sa **robustesse**, sa **conformité aux standards SQL** et ses **fonctionnalités avancées**.

### Caractéristiques principales

| Caractéristique | Description |
|-----------------|-------------|
| 🔓 **Open Source** | Licence PostgreSQL (très permissive) |
| 📊 **Type** | Base de données relationnelle SQL |
| 🧩 **Extensibilité** | Support des extensions (PostGIS, pg_trgm...) |
| 🔒 **ACID** | Transactions complètement ACID |
| 🌐 **JSON** | Support natif JSON et JSONB (NoSQL-like) |
| 🔍 **Recherche** | Full-text search intégré |
| 🎯 **Standards** | Forte conformité SQL (SQL:2016) |
| 📈 **Performance** | Optimisé pour les charges complexes |

---

## 💡 Pourquoi PostgreSQL ?

### Points forts

✅ **Fiabilité légendaire**
- Stabilité éprouvée depuis 30+ ans
- Intégrité des données garantie (ACID)
- Réplication et haute disponibilité natives

✅ **Fonctionnalités avancées**
- Types de données riches (tableaux, JSON, XML, géométrie...)
- Requêtes complexes optimisées (CTE, window functions, LATERAL...)
- Procédures stockées (PL/pgSQL, Python, Perl...)
- Triggers et contraintes sophistiqués

✅ **Extensibilité**
- Création de types de données personnalisés
- Extensions puissantes (PostGIS pour géolocalisation, TimescaleDB pour séries temporelles...)
- Support de multiples langages procéduraux

✅ **Écosystème riche**
- Outils d'administration (pgAdmin, DBeaver, DataGrip)
- Excellent support communautaire
- Documentation de qualité

### Quand choisir PostgreSQL ?

| ✅ Utilisez PostgreSQL si... | ❌ Évitez PostgreSQL si... |
|------------------------------|----------------------------|
| Applications web complexes | Besoin de simplicité extrême (→ SQLite) |
| Intégrité des données critique | Besoin de schéma ultra-flexible (→ MongoDB) |
| Requêtes analytiques avancées | Cache haute performance (→ Redis) |
| Support JSON/NoSQL hybride | Applications Microsoft .NET (→ MS SQL) |
| Géolocalisation (+ PostGIS) | Très petits projets embarqués |
| Conformité SQL stricte requise | - |

---

## 📦 Versions disponibles

### Images Docker officielles

L'image officielle PostgreSQL est disponible sur [Docker Hub](https://hub.docker.com/_/postgres).

| Version | Tag Docker | Statut | Recommandation |
|---------|------------|--------|----------------|
| **16.x** | `postgres:16` | Stable actuel | ✅ **Production 2024+** |
| **15.x** | `postgres:15` | Stable LTS | ✅ Production |
| **14.x** | `postgres:14` | Maintenance | ⚠️ Fin de vie 2026 |
| **13.x** | `postgres:13` | Maintenance | ⚠️ Fin de vie 2025 |
| Alpine | `postgres:16-alpine` | Léger | 🪶 Image réduite |
| Latest | `postgres:latest` | Dernière stable | 🎯 Développement |

**💡 Recommandation :** Utilisez `postgres:15` ou `postgres:16` pour la production, `postgres:latest` pour le développement.

### Nouveautés principales

**PostgreSQL 16 (2023)**
- Amélioration des performances (parallélisation)
- Réplication logique améliorée
- Optimisations des requêtes

**PostgreSQL 15 (2022)**
- Compression TOAST améliorée
- Support SQL/JSON étendu
- Performance des tris optimisée

---

## 📚 Guides disponibles

Ce dossier contient 5 guides couvrant tous les aspects de PostgreSQL avec Docker :

| Guide | Niveau | Temps | Description |
|-------|--------|-------|-------------|
| **2.1** [Config basique](01-config-basique-docker-compose.md) | 🟢 Débutant | 5 min | Installation rapide |
| **2.2** [Config avancée](02-config-avec-postgresqlconf.md) | 🟡 Intermédiaire | 15 min | Fichier `postgresql.conf` |
| **2.3** [IP fixe](03-config-ip-fixe.md) | 🟡 Intermédiaire | 10 min | Réseau Docker personnalisé |
| **2.4** [Gestion utilisateurs](04-gestion-utilisateurs-bdd.md) | 🟡 Intermédiaire | 20 min | Roles, permissions, bases |
| **2.5** [Avec pgAdmin](05-config-avec-pgadmin.md) | 🟢 Débutant | 10 min | Interface graphique |

### Parcours recommandés

**🎓 Débutant complet**
```
Guide 2.1 (Config basique) → Guide 2.5 (pgAdmin)
```

**👨‍💻 Développeur**
```
Guide 2.1 → Guide 2.4 (Utilisateurs) → Guide 2.2 (Config avancée)
```

**🔧 Administrateur**
```
Guide 2.2 (Config avancée) → Guide 2.3 (IP fixe) → Guide 2.4 (Utilisateurs)
```

---

## 🏗️ Architecture recommandée

### Pour le développement

```
┌─────────────────────────────────────────┐
│           Votre Machine (Hôte)          │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │   Docker Container: postgres_dev  │ │
│  │                                   │ │
│  │   PostgreSQL 15                   │ │
│  │   Port: 5432 → 5432              │ │
│  │   Volume: ./data → /var/lib/...  │ │
│  │   Réseau: bridge (défaut)        │ │
│  └───────────────────────────────────┘ │
│              ↕                          │
│  ┌───────────────────────────────────┐ │
│  │   Client SQL (pgAdmin, DBeaver)  │ │
│  │   localhost:5432                  │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Avec IP fixe (multi-conteneurs)

```
┌──────────────────────────────────────────────┐
│        Réseau Docker: postgres_net           │
│            (172.20.0.0/16)                   │
│                                              │
│  ┌────────────────────┐  ┌────────────────┐ │
│  │   postgres_db      │  │   app_backend  │ │
│  │   172.20.0.10:5432│←─│   172.20.0.20  │ │
│  └────────────────────┘  └────────────────┘ │
│            ↕                                 │
│  ┌────────────────────┐                     │
│  │     pgadmin        │                     │
│  │   172.20.0.30      │                     │
│  └────────────────────┘                     │
└──────────────────────────────────────────────┘
         ↕
   Hôte (5432, 8080)
```

---

## ✅ Prérequis

### Logiciels requis

- ✅ **Docker** 20.10+ installé
- ✅ **Docker Compose** 2.0+ installé
- ✅ **4 Go RAM** minimum disponible
- ✅ **10 Go d'espace disque** minimum

### Vérification rapide

```bash
# Vérifier Docker
docker --version
# Attendu: Docker version 20.10.x ou supérieur

# Vérifier Docker Compose
docker-compose --version
# Attendu: Docker Compose version 2.x.x ou supérieur

# Tester Docker
docker run hello-world
# Doit afficher: "Hello from Docker!"
```

### Connaissances recommandées

| Niveau | Connaissances |
|--------|---------------|
| 🟢 **Débutant** | Commandes terminal de base |
| 🟡 **Intermédiaire** | Bases SQL (SELECT, INSERT, UPDATE) |
| 🔴 **Avancé** | Administration PostgreSQL, réseaux Docker |

---

## 🔑 Points clés à connaître

### 1. Ports par défaut

| Service | Port | Usage |
|---------|------|-------|
| PostgreSQL | `5432` | Connexions client |
| pgAdmin | `80` ou `8080` | Interface web |

### 2. Utilisateurs et authentification

PostgreSQL utilise un système de **roles** (utilisateurs et groupes) :

- **`postgres`** : Super-utilisateur par défaut (équivalent `root`)
- **Authentification** : Par mot de passe (md5, scram-sha-256)
- **Fichier `pg_hba.conf`** : Contrôle les méthodes d'authentification

### 3. Bases de données initiales

Par défaut, PostgreSQL crée 3 bases :

| Base | Usage |
|------|-------|
| `postgres` | Base par défaut, administration |
| `template0` | Template de référence (non modifiable) |
| `template1` | Template personnalisable pour nouvelles bases |

### 4. Stockage et volumes

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

**Chemin important :** `/var/lib/postgresql/data`
- Contient TOUTES les données
- **Critique** : À sauvegarder régulièrement

### 5. Variables d'environnement essentielles

| Variable | Description | Exemple |
|----------|-------------|---------|
| `POSTGRES_PASSWORD` | Mot de passe root (**OBLIGATOIRE**) | `mon_mdp_secure` |
| `POSTGRES_USER` | Nom utilisateur (défaut: `postgres`) | `admin` |
| `POSTGRES_DB` | Nom de la base initiale | `myapp` |
| `POSTGRES_INITDB_ARGS` | Args d'initialisation | `--encoding=UTF8` |

---

## 🎯 Cas d'usage typiques

### 1. Application web (Django, Rails, Node.js)

**Caractéristiques :**
- Base principale de l'application
- Schéma relationnel complexe
- Intégrité référentielle importante

**Configuration :** Guide 2.1 (basique) + Guide 2.4 (utilisateurs)

### 2. Analytique et reporting

**Caractéristiques :**
- Requêtes complexes (JOINs, agrégations)
- Grandes volumétries
- Window functions et CTE

**Configuration :** Guide 2.2 (config avancée pour performance)

### 3. Application géolocalisée

**Caractéristiques :**
- Extension PostGIS
- Requêtes spatiales
- Données cartographiques

**Configuration :** Guide 2.2 (avec installation PostGIS)

### 4. API REST / GraphQL

**Caractéristiques :**
- Backend moderne (NestJS, FastAPI...)
- ORM (Prisma, TypeORM, SQLAlchemy)
- Migrations automatisées

**Configuration :** Guide 2.3 (IP fixe) + Guide 2.4

### 5. Microservices

**Caractéristiques :**
- Plusieurs services → plusieurs bases
- Isolation des données
- Réseau Docker dédié

**Configuration :** Guide 2.3 (IP fixe avec réseau personnalisé)

---

## 📊 Comparaison rapide

### PostgreSQL vs autres BDD

| Critère | PostgreSQL | MySQL/MariaDB | MS SQL | MongoDB |
|---------|------------|---------------|---------|---------|
| **Conformité SQL** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ (NoSQL) |
| **Performance lecture** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Performance écriture** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Fonctionnalités** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Facilité** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **JSON/NoSQL** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Extensions** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Communauté** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

**🎯 PostgreSQL excelle pour :**
- Applications nécessitant SQL avancé
- Intégrité des données critique
- Données hybrides (SQL + JSON)
- Requêtes analytiques complexes

---

## 🔗 Ressources complémentaires

### Documentation officielle

- 📖 [Documentation PostgreSQL](https://www.postgresql.org/docs/)
- 🐳 [Image Docker officielle](https://hub.docker.com/_/postgres)
- 📚 [Wiki PostgreSQL](https://wiki.postgresql.org/)

### Outils recommandés

| Outil | Type | Usage |
|-------|------|-------|
| **pgAdmin** | GUI | Administration complète |
| **DBeaver** | GUI | Client universel |
| **DataGrip** | GUI | IDE JetBrains (payant) |
| **psql** | CLI | Client en ligne de commande |
| **pg_dump** | CLI | Sauvegarde |
| **pgBadger** | CLI | Analyse de logs |

### Commandes rapides

```bash
# Se connecter au conteneur
docker exec -it postgres_dev bash

# Client psql
docker exec -it postgres_dev psql -U postgres

# Voir les bases
docker exec -it postgres_dev psql -U postgres -c "\l"

# Backup
docker exec postgres_dev pg_dump -U postgres mydb > backup.sql

# Restore
docker exec -i postgres_dev psql -U postgres mydb < backup.sql
```

---

## 🚀 Démarrage rapide

Prêt à commencer ? Suivez le guide selon votre besoin :

1. **Premier contact avec PostgreSQL ?**
   → [Guide 2.1 - Configuration basique](01-config-basique-docker-compose.md)

2. **Besoin d'une interface graphique ?**
   → [Guide 2.5 - PostgreSQL avec pgAdmin](05-config-avec-pgadmin.md)

3. **Projet avec plusieurs conteneurs ?**
   → [Guide 2.3 - Configuration IP fixe](03-config-ip-fixe.md)

4. **Configuration pour la production ?**
   → [Guide 2.2 - Configuration avancée](02-config-avec-postgresqlconf.md)

---

## ⚠️ Notes importantes

### Sécurité

🔒 **Les configurations de ce guide sont pour le DÉVELOPPEMENT uniquement**

Pour la production :
- Changez TOUS les mots de passe par défaut
- Utilisez des certificats SSL/TLS
- Configurez `pg_hba.conf` de manière restrictive
- Activez les audits de connexion
- Mettez en place des sauvegardes automatiques

### Performance

💡 **Ajustements recommandés pour le développement :**
- `shared_buffers` : 25% de la RAM disponible
- `work_mem` : 10-50 MB
- `maintenance_work_mem` : 256 MB

→ Détails dans le [Guide 2.2](02-config-avec-postgresqlconf.md)

### Compatibilité

✅ **Testé avec :**
- Docker Desktop (Windows, macOS)
- Docker Engine (Linux)
- Docker Compose v2

---

**🐘 Prêt à explorer PostgreSQL avec Docker !**

**[▶️ Commencer avec le Guide 2.1](01-config-basique-docker-compose.md)**

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
