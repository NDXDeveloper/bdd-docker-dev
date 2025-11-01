# Annexe D - Sécurité et Bonnes Pratiques

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

La sécurité est **essentielle**, même en développement. Un projet mal sécurisé peut exposer vos données, faciliter les attaques, ou causer des problèmes en production. Cette annexe vous guide pour sécuriser vos conteneurs Docker dès le départ.

**Ce que vous allez apprendre :**
- 🔐 Gérer les mots de passe de manière sécurisée
- 📄 Utiliser correctement les fichiers `.env`
- 🛡️ Isoler vos conteneurs sur le réseau
- 🔄 Maintenir vos images à jour
- ✅ Suivre les bonnes pratiques de sécurité

**⚠️ IMPORTANT :** Ce guide se concentre sur le **développement**. La production nécessite des mesures supplémentaires (certificats SSL, pare-feu avancés, monitoring, etc.).

**Niveau :** 🟡 Intermédiaire (expliqué pour débutants)

**Durée de lecture :** 40 minutes

---

## 📑 Table des Matières

1. [Gestion des Mots de Passe](#-1-gestion-des-mots-de-passe)
2. [Fichiers .env et Secrets](#-2-fichiers-env-et-secrets)
3. [Isolation Réseau](#-3-isolation-réseau)
4. [Mises à Jour de Sécurité](#-4-mises-à-jour-de-sécurité)
5. [Sécurité des Images](#-5-sécurité-des-images)
6. [Principe du Moindre Privilège](#-6-principe-du-moindre-privilège)
7. [Checklist de Sécurité](#-7-checklist-de-sécurité)

---

## 🔐 1. Gestion des Mots de Passe

### 1.1 Les Dangers des Mauvaises Pratiques

#### ❌ Scénario Catastrophe

```yaml
# docker-compose.yml commité dans Git
version: '3.8'
services:
  mariadb:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: admin123  # ❌ MOT DE PASSE EN CLAIR !
```

**Ce qui peut se passer :**

```
Vous : *commit et push sur GitHub*
    ↓
GitHub : *repo public par erreur*
    ↓
Bot malveillant : *scan automatique des repos*
    ↓
Attaquant : "Super, un mot de passe !"
    ↓
Votre serveur : 💥 Compromis
```

**Même en repo privé, c'est risqué :**
- 👥 Toute l'équipe voit le mot de passe
- 📜 Historique Git conserve TOUT (même après suppression)
- 🔓 Si le repo devient public plus tard
- 💼 Collègue qui quitte l'entreprise

---

### 1.2 Règles d'Or des Mots de Passe

#### Règle 1 : Jamais en Clair dans les Fichiers

**❌ MAUVAIS :**
```yaml
environment:
  MYSQL_ROOT_PASSWORD: admin123
  DB_PASSWORD: password
```

**✅ BON :**
```yaml
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
  DB_PASSWORD: ${DB_PASSWORD}
```

---

#### Règle 2 : Mots de Passe Forts

**❌ Mots de passe FAIBLES :**
```
admin
password123
root
123456
qwerty
admin123
```

**✅ Mots de passe FORTS :**
```
Xk9$mP2#qL8@vN5&wR7!tY4
J8!pLm#2wQ9$xR5@kN3&vT7
dF$8hG#3kL@9mP!5nQ&2rS
```

**Caractéristiques d'un bon mot de passe :**
- ✅ Au moins 16 caractères
- ✅ Mélange : majuscules, minuscules, chiffres, symboles
- ✅ Pas de mots du dictionnaire
- ✅ Pas d'informations personnelles
- ✅ Unique pour chaque service

---

#### Règle 3 : Génération Automatique

**Utilisez des outils pour générer des mots de passe :**

```bash
# Linux/macOS : Générer un mot de passe aléatoire
openssl rand -base64 32

# Résultat : Xk9$mP2#qL8@vN5&wR7!tY4uI3oP6xZ1

# Ou avec pwgen (à installer)
pwgen -s 32 1

# Windows PowerShell
$bytes = New-Object byte[] 32
[Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes)
[Convert]::ToBase64String($bytes)
```

**Sites web (utilisez avec prudence) :**
- [1Password Password Generator](https://1password.com/password-generator/)
- [LastPass Password Generator](https://www.lastpass.com/features/password-generator)

---

### 1.3 Où NE PAS Stocker les Mots de Passe

| ❌ Ne JAMAIS stocker | Raison |
|---------------------|---------|
| Dans `docker-compose.yml` | Versionné dans Git |
| Dans `Dockerfile` | Image partagée = mot de passe partagé |
| Dans le code source | Accessible à tous les devs |
| Dans les logs | Lisibles par n'importe qui |
| En commentaire | "Juste pour moi" = fuite |
| Dans un fichier texte commité | Git garde l'historique |

---

### 1.4 Où Stocker les Mots de Passe

#### Pour le Développement

**✅ Fichier `.env` (détaillé dans la section suivante)**

#### Pour la Production

| Solution | Quand l'utiliser | Exemples |
|----------|------------------|----------|
| **Gestionnaire de secrets** | Production sérieuse | AWS Secrets Manager, Azure Key Vault, HashiCorp Vault |
| **Variables d'environnement** | Serveurs contrôlés | Définies au niveau système |
| **Fichiers chiffrés** | Déploiement automatisé | Ansible Vault, SOPS |
| **Docker Secrets** | Docker Swarm | `docker secret create` |

---

## 📄 2. Fichiers .env et Secrets

### 2.1 Qu'est-ce qu'un Fichier .env ?

Un fichier `.env` contient des **variables d'environnement** qui sont injectées dans vos conteneurs. C'est le moyen standard de séparer la configuration (mots de passe, URLs) du code.

**Analogie :**
```
docker-compose.yml = Recette de cuisine (les instructions)
.env = Liste de courses (les ingrédients secrets)
```

---

### 2.2 Mise en Place

#### Étape 1 : Créer le fichier .env

**Dans le même dossier que `docker-compose.yml` :**

```bash
# Fichier : .env

# Base de données
MYSQL_ROOT_PASSWORD=Xk9$mP2#qL8@vN5&wR7!tY4
MYSQL_DATABASE=app_db
MYSQL_USER=app_user
MYSQL_PASSWORD=J8!pLm#2wQ9$xR5@kN3&vT7

# Application
APP_SECRET_KEY=dF$8hG#3kL@9mP!5nQ&2rS
API_KEY=lM#7pQ$2xR@9nK&5tV!3yW

# Redis
REDIS_PASSWORD=kN$5tV!3yW&8zX@2mP#7qL
```

---

#### Étape 2 : Utiliser dans docker-compose.yml

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    environment:
      # Syntaxe : ${NOM_VARIABLE}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mariadb_data:/var/lib/mysql

  app:
    image: mon_app
    environment:
      DB_HOST: mariadb
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
      DB_NAME: ${MYSQL_DATABASE}
      APP_SECRET: ${APP_SECRET_KEY}

volumes:
  mariadb_data:
```

---

#### Étape 3 : Créer .env.example (versionné)

```bash
# Fichier : .env.example
# À commiter dans Git comme template

# Base de données
MYSQL_ROOT_PASSWORD=changez_moi_avec_un_mot_de_passe_fort
MYSQL_DATABASE=app_db
MYSQL_USER=app_user
MYSQL_PASSWORD=changez_moi_aussi

# Application
APP_SECRET_KEY=votre_secret_key
API_KEY=votre_api_key

# Redis
REDIS_PASSWORD=changez_redis_password
```

---

#### Étape 4 : Configurer .gitignore

```bash
# Fichier : .gitignore

# Fichiers de secrets (NE JAMAIS COMMITER)
.env
.env.local
.env.*.local
*.env

# Données Docker
data/
*/data/

# Logs
*.log
logs/
```

---

### 2.3 Workflow d'Équipe

**Pour un nouveau développeur :**

```bash
# 1. Clone le repo
git clone https://github.com/equipe/projet.git
cd projet

# 2. Copie le template
cp .env.example .env

# 3. Édite avec ses propres mots de passe
nano .env

# 4. Lance le projet
docker-compose up -d

# ✅ Chaque dev a ses propres mots de passe
```

---

### 2.4 Bonnes Pratiques .env

#### ✅ À FAIRE

```bash
# Nommer clairement les variables
DB_ROOT_PASSWORD=...          # ✅ Clair
MYSQL_ROOT_PASSWORD=...       # ✅ Explicite

# Grouper par service
# === Base de données ===
DB_HOST=mariadb
DB_PASSWORD=...

# === Redis ===
REDIS_HOST=redis
REDIS_PASSWORD=...

# Commenter si nécessaire
# Mot de passe root MariaDB (minimum 16 caractères)
MYSQL_ROOT_PASSWORD=...

# Utiliser des valeurs par défaut sûres
DB_PORT=3306
REDIS_PORT=6379
```

---

#### ❌ À ÉVITER

```bash
# ❌ Mots de passe faibles
PASSWORD=123456

# ❌ Noms vagues
PASS=secret
PWD=admin

# ❌ Valeurs en production dans le template
# .env.example
MYSQL_ROOT_PASSWORD=vrai_password_prod  # ❌ DANGER !

# ❌ Informations sensibles en commentaire
# Admin password is: admin123  # ❌ Visible dans Git
```

---

### 2.5 Vérifier que .env n'est pas dans Git

```bash
# Vérifier si .env est ignoré
git status

# Si .env apparaît, c'est MAUVAIS !
# Il faut l'ajouter à .gitignore

# Si déjà commité par erreur :
git rm --cached .env
git commit -m "Remove .env from Git"

# PUIS ajouter à .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .env to .gitignore"
```

**⚠️ IMPORTANT :** Même après suppression, le fichier reste dans l'historique Git. Pour un nettoyage complet :

```bash
# Utiliser git filter-branch (avancé, demande à un senior)
# Ou utiliser BFG Repo-Cleaner
```

---

### 2.6 Alternatives aux .env

#### Docker Secrets (Docker Swarm)

```bash
# Créer un secret
echo "mon_password" | docker secret create db_password -

# Utiliser dans docker-compose.yml
services:
  mariadb:
    image: mariadb:10.11
    secrets:
      - db_password
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
```

---

#### Gestionnaires de Mots de Passe

**Pour l'équipe :**
- [1Password](https://1password.com/) (payant)
- [Bitwarden](https://bitwarden.com/) (gratuit/payant)
- [LastPass](https://www.lastpass.com/) (gratuit/payant)

**Pour l'entreprise :**
- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault
- Google Secret Manager

---

## 🛡️ 3. Isolation Réseau

### 3.1 Pourquoi Isoler ?

**Sans isolation (comportement par défaut) :**

```
┌──────────────────────────────────────────────┐
│  Réseau Docker par défaut                    │
│                                              │
│  🌐 Internet ←→ All Containers               │
│                                              │
│  Tous les conteneurs peuvent :               │
│  - Se parler entre eux                       │
│  - Accéder à Internet                        │
│  - Être accessibles depuis l'extérieur       │
└──────────────────────────────────────────────┘
```

**Problèmes :**
- 🔓 Une base de données exposée publiquement
- 🔓 Des services internes accessibles de partout
- 🔓 Pas de séparation entre frontend et backend

---

**Avec isolation :**

```
┌──────────────────────────────────────────────────────┐
│  Internet                                            │
│     ↓                                                │
│  ┌─────────────────────┐                             │
│  │  Réseau Frontend    │                             │
│  │  (public)           │                             │
│  │                     │                             │
│  │  🌐 Nginx           │                             │
│  │  🌐 App Web         │                             │
│  └─────────────────────┘                             │
│            ↓                                         │
│  ┌─────────────────────┐                             │
│  │  Réseau Backend     │                             │
│  │  (privé)            │                             │
│  │                     │                             │
│  │  🗄️ MariaDB         │ ← Inaccessible de Internet  │
│  │  📦 Redis           │ ← Inaccessible de Internet  │
│  └─────────────────────┘                             │
└──────────────────────────────────────────────────────┘
```

**Avantages :**
- ✅ Base de données inaccessible depuis Internet
- ✅ Séparation claire des responsabilités
- ✅ Réduction de la surface d'attaque
- ✅ Conformité aux bonnes pratiques

---

### 3.2 Configuration de Base

**Sans isolation (❌ non recommandé) :**

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    ports:
      - "3306:3306"  # ❌ Exposé sur Internet !
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

  app:
    image: mon_app
    ports:
      - "80:80"
```

---

**Avec isolation (✅ recommandé) :**

```yaml
version: '3.8'

services:
  # Base de données (réseau backend uniquement)
  mariadb:
    image: mariadb:10.11
    # ✅ PAS de ports exposés publiquement
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    networks:
      - backend

  # Application (entre frontend et backend)
  app:
    image: mon_app
    environment:
      DB_HOST: mariadb  # Accessible via nom de service
    networks:
      - frontend
      - backend
    # Port exposé uniquement si nécessaire
    # ports:
    #   - "127.0.0.1:8080:8080"  # Uniquement en local

  # Proxy web (réseau frontend uniquement)
  nginx:
    image: nginx
    ports:
      - "80:80"  # Seul point d'entrée public
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

---

### 3.3 Niveaux d'Isolation

#### Niveau 1 : Bind sur localhost uniquement

```yaml
services:
  mariadb:
    image: mariadb:10.11
    ports:
      # ✅ Accessible uniquement depuis votre PC
      - "127.0.0.1:3306:3306"
      # Au lieu de
      # - "3306:3306"  # ❌ Accessible depuis le réseau
```

**Résultat :**
```
✅ Vous pouvez vous connecter : localhost:3306
❌ Quelqu'un sur le réseau ne peut pas : 192.168.1.50:3306
```

---

#### Niveau 2 : Pas de ports exposés

```yaml
services:
  mariadb:
    image: mariadb:10.11
    # ✅ Aucun port exposé
    # Accessible uniquement par les autres conteneurs
    networks:
      - backend
```

**Résultat :**
```
✅ Conteneur "app" peut se connecter : mariadb:3306
❌ Votre PC ne peut pas se connecter directement
❌ Quelqu'un sur le réseau ne peut pas se connecter
```

---

#### Niveau 3 : Réseaux séparés

```yaml
networks:
  frontend:    # Applications publiques
  backend:     # Bases de données, caches
  admin:       # Outils d'administration
```

---

### 3.4 Exemple Complet : Application 3-Tiers

```yaml
version: '3.8'

services:
  # === Frontend (public) ===
  nginx:
    image: nginx
    ports:
      - "80:80"
      - "443:443"
    networks:
      - frontend
    depends_on:
      - api

  # === API (bridge entre frontend et backend) ===
  api:
    image: mon_api
    environment:
      DB_HOST: mariadb
      REDIS_HOST: redis
    networks:
      - frontend   # Accessible par Nginx
      - backend    # Accède à MariaDB et Redis
    # Pas de ports exposés

  # === Base de données (backend privé) ===
  mariadb:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    networks:
      - backend  # Uniquement backend
    volumes:
      - mariadb_data:/var/lib/mysql
    # Pas de ports exposés

  # === Cache (backend privé) ===
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    networks:
      - backend  # Uniquement backend
    # Pas de ports exposés

  # === Outil d'admin (réseau admin) ===
  phpmyadmin:
    image: phpmyadmin
    environment:
      PMA_HOST: mariadb
    networks:
      - admin
      - backend
    ports:
      - "127.0.0.1:8080:80"  # Accessible uniquement en local

volumes:
  mariadb_data:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
  admin:
    driver: bridge
```

**Architecture résultante :**

```
Internet
   ↓
┌──────────────────┐
│  Nginx (80/443)  │  ← Seul point d'entrée public
└──────────────────┘
   ↓ (frontend)
┌──────────────────┐
│       API        │
└──────────────────┘
   ↓ (backend)
┌──────────────────┐  ┌──────────────────┐
│     MariaDB      │  │      Redis       │
└──────────────────┘  └──────────────────┘
        ↑ (backend)
┌──────────────────┐
│   phpMyAdmin     │ ← Accessible uniquement localhost:8080
└──────────────────┘
```

---

### 3.5 Bonnes Pratiques d'Isolation

#### ✅ Règles à Suivre

1. **N'exposez que ce qui est nécessaire**
```yaml
# ❌ MAUVAIS
ports:
  - "3306:3306"
  - "6379:6379"
  - "27017:27017"

# ✅ BON
# Aucun port exposé pour les BDD
# Communication via réseau Docker interne
```

2. **Bind sur localhost pour les outils de dev**
```yaml
# ✅ phpMyAdmin accessible uniquement en local
phpmyadmin:
  ports:
    - "127.0.0.1:8080:80"
```

3. **Utiliser plusieurs réseaux**
```yaml
# ✅ Séparer frontend et backend
networks:
  - frontend
  - backend
```

4. **Limiter les connexions sortantes (avancé)**
```yaml
# Empêcher un conteneur d'accéder à Internet
services:
  app:
    networks:
      backend:
        internal: true  # Pas d'accès Internet
```

---

## 🔄 4. Mises à Jour de Sécurité

### 4.1 Pourquoi Mettre à Jour ?

**Les failles de sécurité sont découvertes régulièrement :**

```
Janvier 2024 : Faille dans MariaDB 10.10
    ↓
Correctif dans MariaDB 10.10.7
    ↓
Vous utilisez toujours 10.10.2 ❌
    ↓
Vulnérable à l'attaque
```

**Exemples réels de failles :**
- CVE-2023-XXXX : Injection SQL dans MariaDB < 10.11.3
- CVE-2023-YYYY : Déni de service Redis < 7.0.12
- CVE-2023-ZZZZ : Exécution de code PostgreSQL < 15.4

---

### 4.2 Stratégie de Mise à Jour

#### Développement

```bash
# Toutes les 2-4 semaines
docker-compose pull
docker-compose up -d --force-recreate
```

---

#### Production (plus prudent)

```
1. Surveiller les annonces de sécurité
    ↓
2. Tester dans un environnement de staging
    ↓
3. Planifier une fenêtre de maintenance
    ↓
4. Sauvegarder les données
    ↓
5. Mettre à jour
    ↓
6. Vérifier le bon fonctionnement
```

---

### 4.3 Versions des Images

**❌ DANGEREUX : Tag `latest`**

```yaml
services:
  mariadb:
    image: mariadb:latest  # ❌ Version imprévisible
```

**Problèmes :**
- 🔀 Version change à chaque pull
- 🐛 Nouveaux bugs possibles
- 💥 Breaking changes
- 🔄 Comportement différent entre devs

---

**✅ RECOMMANDÉ : Version spécifique**

```yaml
services:
  mariadb:
    image: mariadb:10.11  # ✅ Version stable
```

**Avantages :**
- ✅ Prévisible
- ✅ Reproductible
- ✅ Contrôle sur les mises à jour
- ✅ Tests possibles avant déploiement

---

**🎯 IDÉAL : Version exacte (production)**

```yaml
services:
  mariadb:
    image: mariadb:10.11.6  # ✅ Version exacte
```

---

### 4.4 Processus de Mise à Jour

#### Étape 1 : Vérifier les versions actuelles

```bash
# Voir les versions de vos images
docker-compose images

# Résultat :
# Container       Repository         Tag       Image Id       Size
# my_mariadb      mariadb            10.11     abc123def      450MB
```

---

#### Étape 2 : Vérifier les nouvelles versions

```bash
# Voir les tags disponibles sur Docker Hub
# https://hub.docker.com/_/mariadb/tags

# Ou via commande
docker search mariadb --limit 5
```

---

#### Étape 3 : Sauvegarder les données

```bash
# Avant toute mise à jour, SAUVEGARDEZ !
./backup-volume.sh mariadb_data

# Ou backup SQL
docker exec mariadb mysqldump \
  -u root -p'${MYSQL_ROOT_PASSWORD}' \
  --all-databases > backup_avant_maj_$(date +%Y%m%d).sql
```

---

#### Étape 4 : Mettre à jour docker-compose.yml

```yaml
services:
  mariadb:
    image: mariadb:10.11.6  # Nouvelle version
```

---

#### Étape 5 : Télécharger et recréer

```bash
# Télécharger la nouvelle image
docker-compose pull

# Recréer les conteneurs avec la nouvelle image
docker-compose up -d --force-recreate

# Vérifier les logs
docker-compose logs -f
```

---

#### Étape 6 : Vérifier le bon fonctionnement

```bash
# Se connecter et vérifier
docker exec -it mariadb mariadb -u root -p

# Vérifier la version
SELECT VERSION();

# Tester quelques requêtes
SHOW DATABASES;
```

---

### 4.5 Automatisation des Mises à Jour

#### Surveillance des Nouvelles Versions

**Outils recommandés :**
- [Watchtower](https://containrrr.dev/watchtower/) : Mise à jour automatique
- [Diun](https://crazymax.dev/diun/) : Notifications de nouvelles versions
- [Dependabot](https://github.com/dependabot) : Pour les repos GitHub

**Exemple avec Watchtower :**

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  # Service Watchtower (vérifie et met à jour)
  watchtower:
    image: containrrr/watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 86400  # Vérifier toutes les 24h
    environment:
      - WATCHTOWER_CLEANUP=true  # Supprimer anciennes images
```

**⚠️ ATTENTION :** Automatisation complète = risqué en production !

---

### 4.6 Checklist de Sécurité des Versions

- [ ] Pas de tag `latest` en production
- [ ] Versions spécifiques documentées
- [ ] Processus de mise à jour défini
- [ ] Sauvegardes avant chaque mise à jour
- [ ] Tests après mise à jour
- [ ] Surveillance des annonces de sécurité
- [ ] Mise à jour au moins trimestrielle

---

## 🖼️ 5. Sécurité des Images

### 5.1 Choisir des Images Officielles

**✅ Images OFFICIELLES (vérifiées) :**

```yaml
services:
  # ✅ Image officielle Docker (badge vérifié)
  mariadb:
    image: mariadb:10.11

  # ✅ Image officielle
  postgres:
    image: postgres:15

  # ✅ Image officielle
  redis:
    image: redis:7-alpine
```

**❌ Images NON OFFICIELLES (risquées) :**

```yaml
services:
  # ❌ Image de source inconnue
  mariadb:
    image: randomuser/mariadb-custom

  # ❌ Image sans vérification
  database:
    image: untrusted-source/database:latest
```

---

### 5.2 Vérifier les Images

#### Sur Docker Hub

1. Rechercher l'image : https://hub.docker.com
2. Vérifier le badge "Official Image" ou "Verified Publisher"
3. Lire la description et la documentation
4. Vérifier le nombre de téléchargements (plus = mieux)
5. Voir la date de dernière mise à jour (récent = maintenu)

---

#### Avec la Commande `docker inspect`

```bash
# Voir les détails d'une image
docker inspect mariadb:10.11

# Vérifier le créateur
docker inspect mariadb:10.11 --format='{{.Author}}'

# Voir les couches (layers)
docker history mariadb:10.11
```

---

### 5.3 Scanner les Vulnérabilités

#### Avec Docker Desktop (intégré)

```bash
# Scanner une image
docker scan mariadb:10.11

# Résultat :
# Testing mariadb:10.11...
#
# ✓ No vulnerabilities found
#
# Tested 145 dependencies for known issues, no vulnerable paths found.
```

---

#### Avec Trivy (outil externe)

```bash
# Installer Trivy
# Linux
sudo apt-get install trivy

# macOS
brew install trivy

# Scanner une image
trivy image mariadb:10.11

# Résultat :
# Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```

---

### 5.4 Images Alpine (Légères et Sécurisées)

**Images Alpine = Version minimale de Linux**

```yaml
services:
  # ✅ Image Alpine (plus petite, moins de vulnérabilités)
  redis:
    image: redis:7-alpine  # ~30MB

  # vs

  # ⚠️ Image standard (plus grosse)
  redis:
    image: redis:7         # ~130MB
```

**Avantages Alpine :**
- ✅ Plus petite (téléchargement rapide)
- ✅ Moins de packages = moins de failles
- ✅ Surface d'attaque réduite

**Inconvénients :**
- ⚠️ Certains outils manquants
- ⚠️ Debugging parfois plus complexe

---

### 5.5 Ne Pas Exécuter en Root

**❌ DANGEREUX : Exécution en root (par défaut)**

```dockerfile
# Par défaut, beaucoup d'images s'exécutent en root
# Si compromis → attaquant a tous les droits
```

**✅ SÉCURISÉ : Utilisateur non-privilégié**

```yaml
services:
  app:
    image: node:18
    # Exécuter en tant qu'utilisateur non-root
    user: "1000:1000"
```

**Ou dans le Dockerfile :**

```dockerfile
FROM node:18

# Créer un utilisateur non-root
RUN useradd -m -u 1000 appuser

# Changer le propriétaire des fichiers
COPY --chown=appuser:appuser . /app

# Basculer vers cet utilisateur
USER appuser

CMD ["node", "server.js"]
```

---

## 👤 6. Principe du Moindre Privilège

### 6.1 Qu'est-ce que le Principe du Moindre Privilège ?

**Définition :** Donner uniquement les permissions nécessaires, pas plus.

**Analogie :**
```
Employé de magasin :
❌ Donner les clés du coffre-fort
✅ Donner les clés du magasin uniquement

Utilisateur base de données :
❌ ALL PRIVILEGES
✅ SELECT, INSERT, UPDATE, DELETE uniquement
```

---

### 6.2 Permissions SQL

**❌ MAUVAIS : Tous les privilèges**

```sql
-- ❌ Utilisateur avec TOUS les droits
GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'%';
```

**Problème :** Si l'application est compromise, l'attaquant peut :
- Supprimer toutes les bases
- Créer des utilisateurs
- Modifier les permissions
- Accéder à tout

---

**✅ BON : Privilèges minimums**

```sql
-- ✅ Uniquement ce qui est nécessaire
CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';

-- Seulement CRUD sur une base spécifique
GRANT SELECT, INSERT, UPDATE, DELETE
ON app_database.*
TO 'app_user'@'%';

FLUSH PRIVILEGES;
```

**Résultat :** L'application fonctionne, mais en cas de compromission :
- ❌ Impossible de supprimer d'autres bases
- ❌ Impossible de créer des utilisateurs
- ❌ Limité à `app_database` uniquement

---

### 6.3 Types d'Utilisateurs Recommandés

```sql
-- 1. ROOT (admin total)
-- À utiliser UNIQUEMENT pour l'administration
-- Ne JAMAIS donner à une application

-- 2. Admin de base (gestion d'une BDD)
CREATE USER 'admin_blog'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON blog_db.* TO 'admin_blog'@'%';

-- 3. Application (CRUD uniquement)
CREATE USER 'app_blog'@'%' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON blog_db.* TO 'app_blog'@'%';

-- 4. Lecture seule (analytics, reporting)
CREATE USER 'read_blog'@'%' IDENTIFIED BY 'password';
GRANT SELECT ON blog_db.* TO 'read_blog'@'%';

-- 5. Backup (sauvegardes)
CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'backup_user'@'localhost';
```

---

### 6.4 Restriction par Hôte

**❌ DANGEREUX : Accès depuis partout**

```sql
CREATE USER 'app'@'%' IDENTIFIED BY 'password';
-- '%' = n'importe quelle IP
```

**✅ SÉCURISÉ : Accès limité**

```sql
-- Seulement depuis le réseau Docker (172.x)
CREATE USER 'app'@'172.%.%.%' IDENTIFIED BY 'password';

-- Ou depuis une IP spécifique
CREATE USER 'app'@'172.20.0.50' IDENTIFIED BY 'password';

-- Ou depuis localhost uniquement
CREATE USER 'app'@'localhost' IDENTIFIED BY 'password';
```

---

### 6.5 Séparation des Comptes

**❌ MAUVAIS : Un seul compte pour tout**

```yaml
environment:
  DB_USER: root
  DB_PASSWORD: ${ROOT_PASSWORD}
```

**✅ BON : Comptes séparés par usage**

```yaml
services:
  app:
    environment:
      DB_USER: app_user           # Utilisateur application
      DB_PASSWORD: ${APP_PASSWORD}

  backup:
    environment:
      DB_USER: backup_user        # Utilisateur backup
      DB_PASSWORD: ${BACKUP_PASSWORD}

  admin:
    environment:
      DB_USER: admin_user         # Utilisateur admin
      DB_PASSWORD: ${ADMIN_PASSWORD}
```

---

## ✅ 7. Checklist de Sécurité

### 7.1 Checklist Avant Déploiement

#### Mots de Passe et Secrets

- [ ] Aucun mot de passe en clair dans les fichiers versionnés
- [ ] Fichier `.env` créé et configuré
- [ ] `.env` ajouté au `.gitignore`
- [ ] `.env.example` créé avec des valeurs fictives
- [ ] Mots de passe forts (16+ caractères)
- [ ] Mots de passe uniques par service
- [ ] Aucun mot de passe par défaut (admin, root, password)

---

#### Réseau et Isolation

- [ ] Bases de données sur réseau backend privé
- [ ] Ports sensibles non exposés publiquement
- [ ] Ports de dev bindés sur `127.0.0.1` uniquement
- [ ] Séparation frontend/backend en place
- [ ] Aucun service inutile exposé

---

#### Images et Versions

- [ ] Images officielles uniquement
- [ ] Versions spécifiques (pas `latest`)
- [ ] Images scannées pour vulnérabilités
- [ ] Images Alpine utilisées quand possible
- [ ] Versions récentes (< 6 mois)

---

#### Permissions et Utilisateurs

- [ ] Utilisateurs SQL avec privilèges minimums
- [ ] Pas d'utilisation de `root` par les applications
- [ ] Restriction par hôte (pas `@'%'` en production)
- [ ] Comptes séparés par usage
- [ ] Conteneurs exécutés en non-root

---

#### Données et Backups

- [ ] Volumes pour toutes les données critiques
- [ ] Stratégie de backup définie
- [ ] Tests de restauration effectués
- [ ] Données sensibles chiffrées
- [ ] Logs ne contiennent pas de secrets

---

#### Configuration Générale

- [ ] Logs limités en taille
- [ ] Health checks configurés
- [ ] Restart policy appropriée
- [ ] Resource limits définis
- [ ] Documentation à jour

---

### 7.2 Checklist Maintenance Continue

#### Hebdomadaire

- [ ] Vérifier les logs d'erreurs
- [ ] Surveiller l'espace disque
- [ ] Nettoyer les volumes inutilisés

---

#### Mensuelle

- [ ] Vérifier les mises à jour de sécurité
- [ ] Tester les backups
- [ ] Auditer les utilisateurs SQL
- [ ] Vérifier les permissions

---

#### Trimestrielle

- [ ] Mettre à jour les images Docker
- [ ] Revoir la configuration de sécurité
- [ ] Tester la restauration complète
- [ ] Former l'équipe aux bonnes pratiques

---

### 7.3 Niveaux de Sécurité

#### 🟢 Niveau Débutant (Minimum Viable)

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11  # ✅ Version spécifique
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}  # ✅ Variable
    volumes:
      - mariadb_data:/var/lib/mysql  # ✅ Persistance

volumes:
  mariadb_data:
```

**+ Fichiers :**
- `.env` (mots de passe)
- `.gitignore` (avec .env dedans)
- `.env.example` (template)

---

#### 🟡 Niveau Intermédiaire (Recommandé)

**Ajoute :**
- Isolation réseau (frontend/backend)
- Ports bindés sur localhost
- Utilisateurs SQL avec privilèges limités
- Health checks
- Resource limits

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    networks:
      - backend  # ✅ Isolation
    volumes:
      - mariadb_data:/var/lib/mysql
    healthcheck:  # ✅ Surveillance
      test: ["CMD", "mysqladmin", "ping"]
      interval: 10s
    deploy:
      resources:  # ✅ Limites
        limits:
          memory: 2G

  app:
    image: mon_app
    networks:
      - frontend
      - backend
    ports:
      - "127.0.0.1:8080:8080"  # ✅ Local uniquement

networks:
  frontend:
  backend:

volumes:
  mariadb_data:
```

---

#### 🔴 Niveau Production (Complet)

**Ajoute :**
- Secrets manager (Vault, AWS Secrets)
- SSL/TLS
- Monitoring et alertes
- Backups automatisés
- Rotation des secrets
- Scanning automatique des vulnérabilités
- Logs centralisés
- Firewalls et WAF

---

## 📊 Tableaux de Référence

### Gravité des Vulnérabilités

| Niveau | Score CVSS | Impact | Action |
|--------|------------|--------|--------|
| **CRITIQUE** | 9.0-10.0 | Exécution de code à distance | Patcher IMMÉDIATEMENT |
| **ÉLEVÉ** | 7.0-8.9 | Accès non autorisé | Patcher sous 7 jours |
| **MOYEN** | 4.0-6.9 | Divulgation d'informations | Patcher sous 30 jours |
| **FAIBLE** | 0.1-3.9 | Impact limité | Patcher lors de la prochaine maj |

---

### Ports Standard et Sécurité

| Service | Port | Exposition Recommandée |
|---------|------|------------------------|
| MariaDB/MySQL | 3306 | ❌ Jamais exposer publiquement |
| PostgreSQL | 5432 | ❌ Jamais exposer publiquement |
| MongoDB | 27017 | ❌ Jamais exposer publiquement |
| Redis | 6379 | ❌ Jamais exposer publiquement |
| HTTP | 80 | ⚠️ Via proxy (Nginx) uniquement |
| HTTPS | 443 | ✅ OK si certificat valide |
| SSH | 22 | ⚠️ Avec authentification par clé |

---

## 💡 Conseils Finaux

### La Sécurité est un Processus Continu

```
Développement → Déploiement → Maintenance → Amélioration
      ↑                                            ↓
      └────────────────────────────────────────────┘
                    (Cycle continu)
```

### Règles d'Or

1. **Toujours partir du principe que vous SEREZ attaqué**
2. **La sécurité parfaite n'existe pas, mais l'amélioration est continue**
3. **Mieux vaut prévenir que guérir : investissez du temps maintenant**
4. **Documentez vos pratiques de sécurité**
5. **Formez votre équipe régulièrement**

---

### Ressources Complémentaires

- 📖 [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- 📖 [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- 📖 [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- 🔍 [Snyk Container Security](https://snyk.io/learn/container-security/)

---

## 🚨 En Cas de Compromission

**Si vous pensez être compromis :**

1. **Isoler** : Arrêter les conteneurs immédiatement
2. **Analyser** : Examiner les logs
3. **Identifier** : Trouver le point d'entrée
4. **Corriger** : Patcher la faille
5. **Restaurer** : Depuis un backup propre
6. **Renforcer** : Améliorer la sécurité
7. **Monitorer** : Surveiller activement

**Commandes d'urgence :**

```bash
# Arrêter tout immédiatement
docker-compose down

# Voir les logs récents
docker-compose logs --tail 1000 > incident_logs.txt

# Sauvegarder les volumes (potentiellement compromis)
./backup-volume.sh mariadb_data_compromis

# Vérifier les connexions réseau
docker network inspect <network_name>

# Scanner les images
trivy image <image_name>
```

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*La sécurité n'est pas une option, c'est une nécessité. 🔒*
