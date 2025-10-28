# Bonnes Pratiques Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Utiliser Docker, c'est bien. L'utiliser correctement, c'est mieux ! Cette fiche vous présente les bonnes pratiques essentielles pour travailler avec Docker de manière professionnelle, sécurisée et efficace.

**Ce que vous allez apprendre :**
- Comment organiser vos projets Docker
- Les règles d'or de la sécurité
- La gestion optimale des ressources
- Les erreurs courantes à éviter
- Les astuces pour gagner du temps

**Public visé :** Développeurs débutants et intermédiaires travaillant en environnement de développement.

---

## 🗂️ Organisation des Projets

### Structure de Dossiers Recommandée

Une bonne organisation facilite la maintenance et le partage de vos projets.

#### Structure Simple (1 Base de Données)

```
mon-projet/
├── docker-compose.yml
├── .env
├── .gitignore
├── data/                    # Données (ignoré par Git)
├── config/                  # Fichiers de configuration
│   └── my.cnf
└── README.md
```

#### Structure Complète (Plusieurs Services)

```
mon-projet/
├── docker-compose.yml
├── .env
├── .gitignore
├── README.md
│
├── mariadb/
│   ├── config/
│   │   └── my.cnf
│   └── init/               # Scripts d'initialisation
│       └── 01-create-users.sql
│
├── postgres/
│   ├── config/
│   │   └── postgresql.conf
│   └── init/
│       └── 01-init-db.sql
│
└── data/                   # Données (ignoré par Git)
    ├── mariadb/
    └── postgres/
```

### Règles d'Or

| Règle | Explication | Exemple |
|-------|-------------|---------|
| **Un dossier = Un projet** | Ne mélangez pas plusieurs projets | `projet-blog/`, `projet-shop/` |
| **Données hors Git** | Ne versionnez jamais les données | Ajoutez `data/` au `.gitignore` |
| **Configuration versionnée** | Les fichiers de config vont dans Git | `config/my.cnf` dans Git |
| **README systématique** | Documentez comment lancer le projet | Instructions claires |

---

## 🔐 Sécurité : Les Fondamentaux

### 1. Gestion des Mots de Passe

#### ❌ À NE JAMAIS FAIRE

```yaml
# MAUVAIS : Mot de passe en clair dans docker-compose.yml
services:
  mariadb:
    image: mariadb
    environment:
      MARIADB_ROOT_PASSWORD: motdepasse123  # ❌ DANGEREUX
```

**Problèmes :**
- Visible dans Git (historique complet)
- Partagé avec toute l'équipe
- Risque si le repo devient public

#### ✅ BONNE PRATIQUE : Fichier .env

**1. Créez un fichier `.env` :**
```bash
# .env (ne JAMAIS commiter dans Git)
MARIADB_ROOT_PASSWORD=un_mot_de_passe_tres_securise_123!
MARIADB_USER=mon_utilisateur
MARIADB_PASSWORD=autre_mot_de_passe_securise_456!
```

**2. Utilisez-le dans `docker-compose.yml` :**
```yaml
services:
  mariadb:
    image: mariadb
    environment:
      MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD}
      MARIADB_USER: ${MARIADB_USER}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD}
```

**3. Ajoutez `.env` au `.gitignore` :**
```gitignore
# .gitignore
.env
.env.local
*.env
data/
```

**4. Créez un fichier `.env.example` (versionné) :**
```bash
# .env.example (à commiter dans Git)
# Copiez ce fichier en .env et remplissez vos valeurs

MARIADB_ROOT_PASSWORD=changez_moi
MARIADB_USER=votre_utilisateur
MARIADB_PASSWORD=changez_moi_aussi
```

**Avantages :**
- ✅ Chaque développeur a ses propres mots de passe
- ✅ Pas de risque de fuite dans Git
- ✅ Facilité de changement
- ✅ Documentation claire (.env.example)

### 2. Règles de Mots de Passe Forts

#### Pour le Développement

**Minimum acceptable :**
- Au moins 12 caractères
- Mélange de lettres, chiffres et symboles
- Pas de mots du dictionnaire
- Différent pour chaque base de données

**Exemples corrects :**
```bash
MARIADB_ROOT_PASSWORD=Dev2024!MariaDB#Secure
POSTGRES_PASSWORD=P0stgr3s_D3v!2024
MONGODB_ROOT_PASSWORD=M0ng0DB@D3v#2024!
```

#### Pour la Production

**Exigences strictes :**
- Au moins 24 caractères
- Généré aléatoirement
- Stocké dans un gestionnaire de secrets (Vault, AWS Secrets Manager...)
- Rotation régulière

**Outil de génération :**
```bash
# Linux/macOS : Générer un mot de passe aléatoire
openssl rand -base64 32

# Ou avec pwgen (à installer)
pwgen -s 32 1
```

### 3. Exposition des Ports

#### ❌ DANGEREUX : Exposition sur toutes les interfaces

```yaml
services:
  mariadb:
    ports:
      - "3306:3306"  # ❌ Accessible depuis n'importe où sur le réseau
```

**Problème :** Votre base de données est accessible depuis n'importe quel ordinateur du réseau local (ou pire, d'Internet si mal configuré).

#### ✅ SÉCURISÉ : Exposition locale uniquement

```yaml
services:
  mariadb:
    ports:
      - "127.0.0.1:3306:3306"  # ✅ Accessible uniquement depuis votre PC
```

**Avantages :**
- ✅ Inaccessible depuis le réseau
- ✅ Protection contre les attaques
- ✅ Toujours accessible pour vos outils locaux

#### 📊 Cas Particulier : Besoin d'accès réseau

Si vous devez vraiment accéder depuis un autre ordinateur (ex: tester depuis un smartphone) :

```yaml
services:
  mariadb:
    ports:
      - "3306:3306"  # Temporairement
    environment:
      # Créez un utilisateur dédié avec permissions limitées
      MARIADB_USER: test_user
      MARIADB_PASSWORD: ${TEST_PASSWORD}
```

**Et configurez votre pare-feu !** (voir [Annexe D - Sécurité](/annexes/D-securite-bonnes-pratiques.md))

### 4. Utilisateurs et Permissions

#### ❌ MAUVAISE PRATIQUE

```sql
-- Donner tous les droits à un utilisateur accessible depuis partout
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' IDENTIFIED BY '123456';
```

**Problèmes :**
- Mot de passe faible
- Accès depuis n'importe où (`%`)
- Tous les privilèges (dangereux)

#### ✅ BONNE PRATIQUE

```sql
-- Utilisateur par application avec privilèges limités
CREATE USER 'blog_app'@'172.18.%' IDENTIFIED BY 'un_mot_de_passe_fort';
GRANT SELECT, INSERT, UPDATE, DELETE ON blog_db.* TO 'blog_app'@'172.18.%';

-- Pas de DROP, GRANT, ou autres privilèges dangereux
```

**Règles :**
- ✅ Un utilisateur par application
- ✅ Privilèges minimums nécessaires (principe du moindre privilège)
- ✅ Restriction d'IP si possible
- ✅ Mot de passe fort et unique

---

## 🏷️ Nommage et Versions

### 1. Noms de Conteneurs

#### ❌ NOMS GÉNÉRIQUES

```yaml
services:
  db:  # Nom trop générique
    image: mariadb
```

**Problème :** Si vous avez plusieurs projets, les conteneurs se confondent.

#### ✅ NOMS DESCRIPTIFS

```yaml
services:
  db:
    image: mariadb
    container_name: blog_mariadb_dev  # ✅ Clair et unique
```

**Format recommandé :**
```
{projet}_{service}_{environnement}

Exemples:
- shop_postgres_dev
- blog_redis_dev
- api_mongodb_dev
```

### 2. Tags d'Images

#### ❌ DANGEREUX : Tag `latest`

```yaml
services:
  db:
    image: mariadb:latest  # ❌ Version imprévisible
```

**Problèmes :**
- La version peut changer à tout moment
- Comportements différents entre développeurs
- Bugs impossibles à reproduire
- Incompatibilités lors de mises à jour

#### ✅ SÉCURISÉ : Versions spécifiques

```yaml
services:
  db:
    image: mariadb:10.11  # ✅ Version stable et prévisible
```

**Règles :**
- ✅ Toujours spécifier une version majeure.mineure
- ✅ Consulter Docker Hub pour les versions disponibles
- ✅ Documenter pourquoi cette version (compatibilité, fonctionnalités...)

**Exemples de bonnes versions :**
```yaml
services:
  mariadb:
    image: mariadb:10.11        # Version stable

  postgres:
    image: postgres:15          # Version majeure

  redis:
    image: redis:7.2-alpine     # Version + variante légère

  mongodb:
    image: mongo:7.0            # Version stable
```

### 3. Noms de Volumes

#### ❌ NOMS GÉNÉRIQUES

```yaml
volumes:
  data:  # Trop vague
```

#### ✅ NOMS DESCRIPTIFS

```yaml
volumes:
  blog_mariadb_data:     # ✅ On sait ce que c'est
  blog_postgres_data:    # ✅ Pas de confusion
  shop_mongodb_data:     # ✅ Organisé par projet
```

---

## 💾 Gestion des Données

### 1. Persistance : Toujours des Volumes

#### ❌ ERREUR CRITIQUE

```yaml
services:
  mariadb:
    image: mariadb
    # ❌ Pas de volume = perte de données à chaque suppression
```

#### ✅ OBLIGATOIRE

```yaml
services:
  mariadb:
    image: mariadb
    volumes:
      - mariadb_data:/var/lib/mysql  # ✅ Données persistantes

volumes:
  mariadb_data:
```

**Règle absolue :** TOUTE base de données DOIT avoir un volume pour les données.

### 2. Bind Mount vs Volume Nommé

**Utilisez les bind mounts UNIQUEMENT pour :**
- Fichiers de configuration
- Scripts d'initialisation
- Développement (code source)

**Utilisez les volumes nommés pour :**
- Données de bases de données (TOUJOURS)
- Performances optimales
- Portabilité

**Exemple correct :**
```yaml
services:
  mariadb:
    image: mariadb
    volumes:
      # Bind mount pour config (éditable facilement)
      - ./config/my.cnf:/etc/mysql/conf.d/custom.cnf:ro  # :ro = read-only

      # Volume nommé pour données (performant)
      - mariadb_data:/var/lib/mysql

volumes:
  mariadb_data:
```

### 3. Sauvegardes

#### Stratégie Minimale

**Pour le développement :**
```bash
# Sauvegarde manuelle avant action risquée
docker exec blog_mariadb_dev mysqldump -u root -p --all-databases > backup_$(date +%Y%m%d).sql

# Restauration
docker exec -i blog_mariadb_dev mysql -u root -p < backup_20241029.sql
```

**Pour la production :**
- ✅ Sauvegardes automatiques quotidiennes
- ✅ Stockage hors du serveur
- ✅ Tests de restauration réguliers
- ✅ Rétention de plusieurs versions

### 4. Scripts d'Initialisation

Utilisez les dossiers d'init pour créer automatiquement vos bases/utilisateurs.

```yaml
services:
  mariadb:
    image: mariadb
    volumes:
      - ./init:/docker-entrypoint-initdb.d:ro
      - mariadb_data:/var/lib/mysql
```

**Fichier `init/01-init.sql` :**
```sql
-- Exécuté automatiquement au premier démarrage
CREATE DATABASE IF NOT EXISTS blog_db;
CREATE USER 'blog_user'@'%' IDENTIFIED BY 'mot_de_passe';
GRANT ALL PRIVILEGES ON blog_db.* TO 'blog_user'@'%';
FLUSH PRIVILEGES;
```

**Avantages :**
- ✅ Configuration automatique
- ✅ Reproductible
- ✅ Versionnable (Git)
- ✅ Documentation vivante

---

## 🚀 Performance et Ressources

### 1. Limiter les Ressources

Évitez qu'un conteneur monopolise toute la machine.

```yaml
services:
  mariadb:
    image: mariadb
    deploy:
      resources:
        limits:
          cpus: '2.0'      # Max 2 CPU
          memory: 2G       # Max 2 GB RAM
        reservations:
          cpus: '0.5'      # Min 0.5 CPU garanti
          memory: 512M     # Min 512 MB garanti
```

**Recommandations par type de BDD :**

| Base de Données | RAM Min | RAM Recommandée | CPU |
|-----------------|---------|-----------------|-----|
| MariaDB/MySQL | 512M | 1-2G | 1-2 |
| PostgreSQL | 512M | 1-2G | 1-2 |
| MongoDB | 1G | 2-4G | 2 |
| Redis | 256M | 512M-1G | 1 |
| Elasticsearch | 2G | 4-8G | 2-4 |

### 2. Health Checks

Vérifiez automatiquement que vos conteneurs sont en bonne santé.

```yaml
services:
  mariadb:
    image: mariadb
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s      # Vérification toutes les 10s
      timeout: 5s        # Timeout après 5s
      retries: 3         # 3 tentatives avant "unhealthy"
      start_period: 30s  # Délai avant première vérification
```

**Avantages :**
- ✅ Détection automatique des problèmes
- ✅ Redémarrage automatique si nécessaire
- ✅ Visible dans `docker ps` (état de santé)

### 3. Réseaux Dédiés

Isolez vos différents projets.

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb
    networks:
      - backend  # ✅ Réseau dédié

  redis:
    image: redis
    networks:
      - backend

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

**Avantages :**
- ✅ Isolation entre projets
- ✅ Pas de conflit de noms
- ✅ Sécurité accrue

---

## 🧹 Maintenance et Nettoyage

### 1. Routine de Nettoyage

Docker accumule des images, conteneurs et volumes inutilisés. Nettoyez régulièrement !

```bash
# Supprimer les conteneurs arrêtés
docker container prune -f

# Supprimer les images non utilisées
docker image prune -a -f

# Supprimer les volumes non utilisés (ATTENTION : perte de données)
docker volume prune -f

# Supprimer les réseaux non utilisés
docker network prune -f

# TOUT nettoyer d'un coup (DANGER : perte de données)
docker system prune -a --volumes -f
```

**⚠️ ATTENTION avec `volume prune` :** Vous perdrez TOUTES les données des volumes non attachés à un conteneur actif !

**Nettoyage sécurisé :**
```bash
# 1. Lister les volumes
docker volume ls

# 2. Identifier ceux à supprimer
docker volume inspect nom_du_volume

# 3. Supprimer individuellement
docker volume rm nom_du_volume
```

### 2. Logs : Éviter le Débordement

Les logs Docker peuvent remplir votre disque !

```yaml
services:
  mariadb:
    image: mariadb
    logging:
      driver: "json-file"
      options:
        max-size: "10m"      # Max 10 MB par fichier de log
        max-file: "3"        # Garder 3 fichiers max
```

**Résultat :** Max 30 MB de logs par conteneur (10 MB × 3 fichiers).

### 3. Mises à Jour

Gardez vos images à jour (mais contrôlez les versions !).

```bash
# Mettre à jour une image spécifique
docker pull mariadb:10.11

# Recréer les conteneurs avec la nouvelle image
docker-compose up -d --force-recreate

# Vérifier les versions
docker images | grep mariadb
```

**Fréquence recommandée :**
- Développement : Tous les 2-3 mois
- Production : Après tests approfondis uniquement

---

## 📝 Documentation et .gitignore

### 1. README.md Complet

Chaque projet doit avoir un README clair.

**Template recommandé :**
```markdown
# Nom du Projet

## Prérequis
- Docker 20.10+
- Docker Compose 2.0+

## Installation

1. Cloner le repo
2. Copier `.env.example` vers `.env`
3. Modifier les mots de passe dans `.env`
4. Lancer : `docker-compose up -d`

## Connexion

- **MariaDB** : localhost:3306
  - User: voir `.env`
  - Password: voir `.env`

## Arrêt

```bash
docker-compose down
```

## Suppression complète (ATTENTION : perte de données)

```bash
docker-compose down -v
```
```

### 2. .gitignore Complet

**Fichier `.gitignore` essentiel :**
```gitignore
# Données Docker (ne JAMAIS commiter)
data/
*/data/
**/data/

# Fichiers de configuration avec secrets
.env
.env.local
.env.*.local
**/.env

# Logs
*.log
logs/
*/logs/
**/logs/

# Backups de bases de données
*.sql
*.dump
*.backup
*.bak

# Certificats et clés
*.key
*.pem
*.crt
*.p12
*.pfx
certs/

# Fichiers système
.DS_Store
Thumbs.db
*.swp
*.swo

# Éditeurs
.vscode/
.idea/
*.iml

# Docker overrides locaux
docker-compose.override.yml
docker-compose.local.yml
```

---

## ⚡ Astuces et Optimisations

### 1. Alias Pratiques

Ajoutez ces alias dans votre `~/.bashrc` ou `~/.zshrc` :

```bash
# Alias Docker utiles
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dcu='docker-compose up -d'
alias dcd='docker-compose down'
alias dcl='docker-compose logs -f'
alias dex='docker exec -it'

# Nettoyage
alias docker-clean='docker system prune -a -f'
alias docker-clean-all='docker system prune -a --volumes -f'
```

### 2. Variables d'Environnement Globales

Créez un fichier `~/.docker_env` :
```bash
# Timezone par défaut
export TZ=Europe/Paris

# Éditeur par défaut
export EDITOR=nano
```

Chargez-le dans votre `.bashrc` :
```bash
source ~/.docker_env
```

### 3. Commandes Rapides

```bash
# Entrer dans un conteneur MariaDB
docker exec -it nom_conteneur mariadb -u root -p

# Voir les logs en temps réel des 100 dernières lignes
docker-compose logs -f --tail=100

# Recréer un seul service
docker-compose up -d --force-recreate nom_service

# Copier un fichier depuis un conteneur
docker cp nom_conteneur:/chemin/fichier.sql ./backup.sql

# Voir la consommation de ressources
docker stats
```

---

## ❌ Erreurs Courantes à Éviter

### Top 10 des Erreurs

| Erreur | Conséquence | Solution |
|--------|-------------|----------|
| 1. Pas de volume pour les données | **Perte de données** | Toujours utiliser des volumes |
| 2. Mots de passe dans Git | **Fuite de sécurité** | Utiliser `.env` + `.gitignore` |
| 3. Tag `latest` en production | **Instabilité** | Versions spécifiques |
| 4. Ports sur `0.0.0.0` | **Risque sécurité** | Bind sur `127.0.0.1` |
| 5. Pas de health check | **Pannes silencieuses** | Ajouter des health checks |
| 6. Logs illimités | **Disque plein** | Limiter taille des logs |
| 7. `docker-compose down -v` par erreur | **Perte totale de données** | Double vérification avant `-v` |
| 8. Oublier `.gitignore` | **Commit de données** | Créer `.gitignore` immédiatement |
| 9. Pas de backup | **Récupération impossible** | Sauvegardes régulières |
| 10. Tous les privilèges à un user | **Risque sécurité** | Moindre privilège |

---

## 🎯 Checklist de Projet

Avant de considérer un projet Docker "propre", vérifiez :

### Fichiers Essentiels
- [ ] `docker-compose.yml` complet et commenté
- [ ] `.env.example` avec toutes les variables
- [ ] `.gitignore` incluant `.env` et `data/`
- [ ] `README.md` avec instructions claires
- [ ] Fichiers de config versionnés (`config/`)

### Configuration
- [ ] Versions spécifiques pour toutes les images
- [ ] Volumes pour toutes les données persistantes
- [ ] Noms de conteneurs descriptifs
- [ ] Ports bindés sur `127.0.0.1` (sauf besoin spécifique)
- [ ] Health checks configurés

### Sécurité
- [ ] Pas de mots de passe en clair
- [ ] Utilisateurs avec privilèges minimums
- [ ] Logs limités en taille
- [ ] Pas de données dans Git

### Documentation
- [ ] Instructions de démarrage claires
- [ ] Informations de connexion documentées
- [ ] Procédure d'arrêt expliquée
- [ ] Avertissements sur la suppression des volumes

---

## 🚀 Prochaines Étapes

Maintenant que vous connaissez les bonnes pratiques :

1. **Déployez votre première base de données** → Choisissez dans le [Sommaire](/SOMMAIRE.md)
2. **Consultez les annexes** :
   - [Annexe A - Référence des commandes](/annexes/A-reference-commandes.md)
   - [Annexe D - Sécurité approfondie](/annexes/D-securite-bonnes-pratiques.md)
   - [Annexe E - Dépannage](/annexes/E-depannage.md)

---

## 📚 Ressources Complémentaires

- **Docker Best Practices** : [https://docs.docker.com/develop/dev-best-practices/](https://docs.docker.com/develop/dev-best-practices/)
- **Docker Security** : [https://docs.docker.com/engine/security/](https://docs.docker.com/engine/security/)
- **Compose file reference** : [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
