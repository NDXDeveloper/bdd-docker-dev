# Prérequis et Installation

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Avant de pouvoir déployer des bases de données avec Docker, vous devez installer et configurer quelques outils sur votre machine. Cette fiche vous guide pas à pas, quel que soit votre système d'exploitation.

**Ce dont vous aurez besoin :**
- Un ordinateur (Windows, macOS ou Linux)
- Une connexion Internet
- Environ 30 minutes pour l'installation complète
- Des droits administrateur sur votre machine

---

## 🖥️ Configuration Matérielle Minimale

Avant de commencer, vérifiez que votre ordinateur répond aux exigences minimales :

| Composant | Minimum | Recommandé |
|-----------|---------|------------|
| **Processeur** | 64 bits (x86_64 ou ARM64) | Multi-cœurs |
| **RAM** | 4 GB | 8 GB ou plus |
| **Disque** | 10 GB libres | 50 GB libres (SSD de préférence) |
| **Système** | Windows 10/11, macOS 10.15+, Linux moderne | Dernières versions |

### Pourquoi ces exigences ?

- **64 bits obligatoire** : Docker nécessite un processeur 64 bits pour fonctionner.
- **RAM** : Chaque conteneur consomme de la mémoire. Avec 4 GB, vous pourrez faire tourner 1-2 bases de données en même temps. Avec 8 GB ou plus, vous serez beaucoup plus confortable.
- **Disque** : Les images Docker et les données de vos bases prennent de la place. Un SSD accélère considérablement les performances.

---

## 🐳 Étape 1 : Installation de Docker

Docker est le moteur qui fait tourner vos conteneurs. L'installation diffère selon votre système d'exploitation.

### 🪟 Windows

**Option recommandée : Docker Desktop**

1. **Vérifier la virtualisation**
   - Appuyez sur `Ctrl + Shift + Esc` pour ouvrir le Gestionnaire des tâches
   - Allez dans l'onglet "Performances"
   - Vérifiez que "Virtualisation" est activée
   - Si elle est désactivée, vous devrez l'activer dans le BIOS (consultez la documentation de votre fabricant)

2. **Télécharger Docker Desktop**
   - Rendez-vous sur [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   - Cliquez sur "Download for Windows"
   - Choisissez la version correspondant à votre processeur (Intel/AMD ou ARM)

3. **Installer Docker Desktop**
   - Double-cliquez sur le fichier téléchargé (`Docker Desktop Installer.exe`)
   - Laissez les options par défaut cochées :
     - ✅ Use WSL 2 instead of Hyper-V (recommandé pour Windows 10/11)
     - ✅ Add shortcut to desktop
   - Cliquez sur "Ok" et patientez pendant l'installation
   - **Redémarrez votre ordinateur** lorsque demandé

4. **Vérifier l'installation**
   - Ouvrez l'invite de commandes (tapez `cmd` dans le menu Démarrer)
   - Tapez la commande suivante :
     ```bash
     docker --version
     ```
   - Vous devriez voir quelque chose comme : `Docker version 24.0.7, build afdd53b`

**Remarque sur WSL 2 :**
Docker Desktop utilise WSL 2 (Windows Subsystem for Linux 2) pour de meilleures performances. Si l'installation vous demande d'installer ou de mettre à jour WSL 2, acceptez. C'est normal et nécessaire.

---

### 🍎 macOS

**Option recommandée : Docker Desktop**

1. **Vérifier la compatibilité**
   - Cliquez sur le logo Apple () > "À propos de ce Mac"
   - Notez si vous avez un Mac Intel ou Apple Silicon (M1/M2/M3)
   - Vérifiez que vous êtes sur macOS 10.15 (Catalina) ou plus récent

2. **Télécharger Docker Desktop**
   - Rendez-vous sur [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   - Cliquez sur "Download for Mac"
   - Choisissez la bonne version :
     - **Mac with Intel chip** pour les Mac Intel
     - **Mac with Apple chip** pour les Mac M1/M2/M3

3. **Installer Docker Desktop**
   - Ouvrez le fichier `.dmg` téléchargé
   - Glissez-déposez l'icône Docker dans le dossier Applications
   - Ouvrez le dossier Applications et double-cliquez sur Docker
   - Acceptez les autorisations système si demandées
   - Docker Desktop va démarrer (une icône de baleine apparaît dans la barre de menu)

4. **Vérifier l'installation**
   - Ouvrez le Terminal (Applications > Utilitaires > Terminal)
   - Tapez la commande suivante :
     ```bash
     docker --version
     ```
   - Vous devriez voir quelque chose comme : `Docker version 24.0.7, build afdd53b`

---

### 🐧 Linux

Sur Linux, vous avez deux options : Docker Desktop (avec interface graphique) ou Docker Engine (ligne de commande uniquement). Nous recommandons Docker Engine pour Linux, plus léger et natif.

**Option recommandée : Docker Engine**

#### Installation automatique (script officiel)

C'est la méthode la plus simple pour la plupart des distributions Linux :

```bash
# Télécharger et exécuter le script d'installation officiel
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Ajouter votre utilisateur au groupe docker (pour ne pas avoir à utiliser sudo)
sudo usermod -aG docker $USER

# Appliquer les changements (ou redémarrez votre session)
newgrp docker
```

#### Installation manuelle (Ubuntu/Debian)

Si vous préférez installer manuellement ou si le script ne fonctionne pas :

```bash
# 1. Mettre à jour les paquets
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

# 2. Ajouter la clé GPG officielle de Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 3. Ajouter le dépôt Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Installer Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 5. Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER
newgrp docker
```

#### Vérifier l'installation

```bash
docker --version
```

Vous devriez voir : `Docker version 24.0.7, build afdd53b` (ou une version similaire)

---

## 🔧 Étape 2 : Installation de Docker Compose

Docker Compose permet de gérer plusieurs conteneurs facilement via des fichiers de configuration. C'est **essentiel** pour ce guide.

### Windows et macOS

**Bonne nouvelle !** Docker Compose est **déjà inclus** dans Docker Desktop. Vous n'avez rien à faire de plus.

Vérifiez simplement qu'il est bien installé :

```bash
docker-compose --version
```

Vous devriez voir : `Docker Compose version v2.23.0` (ou similaire)

### Linux

Si vous avez utilisé le script d'installation automatique ou installé le paquet `docker-compose-plugin`, Docker Compose est déjà installé.

Vérifiez avec :

```bash
docker compose version
```

**Remarque :** Sur Linux, la commande peut être `docker compose` (avec un espace) au lieu de `docker-compose` (avec un tiret) selon la version. Les deux fonctionnent, mais nous utiliserons `docker-compose` dans ce guide pour la compatibilité.

Si Docker Compose n'est pas installé, installez-le avec :

```bash
# Ubuntu/Debian
sudo apt-get install docker-compose-plugin

# Ou téléchargement direct (toutes distributions)
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---

## ✅ Étape 3 : Vérification de l'Installation

Maintenant que Docker et Docker Compose sont installés, testons que tout fonctionne correctement.

### Test 1 : Commandes de base

Ouvrez un terminal (ou invite de commandes sur Windows) et tapez :

```bash
# Vérifier la version de Docker
docker --version

# Vérifier la version de Docker Compose
docker-compose --version

# Afficher des informations détaillées sur Docker
docker info
```

### Test 2 : Premier conteneur

Testons Docker avec un conteneur de démonstration :

```bash
docker run hello-world
```

**Ce qui devrait se passer :**
1. Docker télécharge l'image `hello-world` (très petite, quelques KB)
2. Docker lance un conteneur à partir de cette image
3. Le conteneur affiche un message de bienvenue
4. Le conteneur s'arrête automatiquement

**Message attendu :**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

Si vous voyez ce message, **félicitations !** Docker fonctionne correctement.

### Test 3 : Nettoyage

Nettoyons ce conteneur de test (bonne habitude à prendre) :

```bash
# Voir les conteneurs (même arrêtés)
docker ps -a

# Supprimer le conteneur hello-world
docker rm <ID_ou_NOM_du_conteneur>

# Supprimer l'image hello-world
docker rmi hello-world
```

---

## 🛠️ Étape 4 : Configuration Optionnelle (Recommandée)

Ces configurations ne sont pas obligatoires, mais amélioreront votre expérience.

### Augmenter les ressources allouées à Docker (Windows/macOS)

Par défaut, Docker Desktop limite les ressources utilisées. Vous pouvez les augmenter :

1. Ouvrez Docker Desktop
2. Cliquez sur l'icône ⚙️ (Paramètres / Settings)
3. Allez dans "Resources"
4. Ajustez :
   - **CPUs** : Au moins 2, idéalement 4
   - **Memory** : Au moins 4 GB, idéalement 8 GB
   - **Swap** : 1-2 GB
   - **Disk image size** : Au moins 30 GB
5. Cliquez sur "Apply & Restart"

### Activer Docker au démarrage

- **Windows/macOS** : Dans Docker Desktop > Settings, cochez "Start Docker Desktop when you log in"
- **Linux** :
  ```bash
  sudo systemctl enable docker
  ```

### Configurer un proxy (si nécessaire)

Si vous êtes derrière un proxy d'entreprise, consultez la documentation officielle : [Configure Docker to use a proxy](https://docs.docker.com/network/proxy/)

---

## 📦 Étape 5 : Outils Complémentaires (Optionnels)

Ces outils ne sont pas obligatoires, mais très pratiques.

### Éditeur de texte/code

Pour éditer les fichiers `docker-compose.yml` et de configuration, vous aurez besoin d'un éditeur. Recommandations :

- **VS Code** (gratuit, multi-plateforme) : [https://code.visualstudio.com/](https://code.visualstudio.com/)
  - Installer l'extension "Docker" pour la coloration syntaxique
- **Sublime Text** (gratuit) : [https://www.sublimetext.com/](https://www.sublimetext.com/)
- **Notepad++** (Windows uniquement) : [https://notepad-plus-plus.org/](https://notepad-plus-plus.org/)

**Évitez** le Bloc-notes Windows classique qui peut causer des problèmes de formatage.

### Clients pour Bases de Données

Pour vous connecter à vos bases de données, vous aurez besoin de clients GUI :

| Base de Données | Client Recommandé | Lien |
|-----------------|-------------------|------|
| **MariaDB / MySQL** | HeidiSQL (Windows), DBeaver (tous) | [heidisql.com](https://www.heidisql.com/) |
| **PostgreSQL** | pgAdmin, DBeaver | [pgadmin.org](https://www.pgadmin.org/) |
| **MongoDB** | MongoDB Compass | [mongodb.com/compass](https://www.mongodb.com/products/compass) |
| **Redis** | RedisInsight | [redis.com/redis-enterprise/redis-insight](https://redis.com/redis-enterprise/redis-insight/) |
| **Universel** | DBeaver (gratuit, supporte presque tout) | [dbeaver.io](https://dbeaver.io/) |

Nous recommandons **DBeaver Community** comme client universel gratuit pour débuter.

---

## 🎯 Vérification Finale : Checklist

Avant de passer aux fiches de configuration, assurez-vous que :

- [ ] Docker est installé (`docker --version` fonctionne)
- [ ] Docker Compose est installé (`docker-compose --version` fonctionne)
- [ ] Le test `docker run hello-world` a réussi
- [ ] Docker Desktop démarre automatiquement (Windows/macOS) ou le service est actif (Linux)
- [ ] Vous avez un éditeur de texte approprié
- [ ] Vous avez téléchargé au moins un client de BDD (ou DBeaver)

Si tous ces points sont validés, vous êtes prêt à déployer votre première base de données ! 🎉

---

## ❓ Problèmes Courants

### "docker: command not found"

**Cause :** Docker n'est pas dans le PATH ou pas démarré.

**Solutions :**
- Windows/macOS : Assurez-vous que Docker Desktop est lancé (icône dans la barre des tâches/menu)
- Linux : Vérifiez que le service tourne : `sudo systemctl status docker`
- Redémarrez votre terminal après l'installation

### "permission denied while trying to connect to the Docker daemon"

**Cause :** Votre utilisateur n'a pas les droits pour utiliser Docker.

**Solution (Linux uniquement) :**
```bash
sudo usermod -aG docker $USER
newgrp docker
# Ou déconnectez-vous et reconnectez-vous
```

### "WSL 2 installation is incomplete" (Windows)

**Cause :** WSL 2 n'est pas à jour ou pas installé.

**Solution :**
1. Ouvrez PowerShell en administrateur
2. Tapez : `wsl --update`
3. Redémarrez Docker Desktop

### Docker Desktop ne démarre pas (macOS)

**Cause :** Problèmes de permissions ou macOS trop ancien.

**Solutions :**
1. Vérifiez que vous êtes sur macOS 10.15 ou plus récent
2. Donnez les autorisations complètes à Docker dans "Préférences Système > Sécurité et confidentialité"
3. Réinstallez Docker Desktop si nécessaire

### Performances lentes (Windows/macOS)

**Cause :** Ressources insuffisantes allouées à Docker.

**Solution :** Augmentez la RAM et les CPU dans Docker Desktop > Settings > Resources (voir Étape 4)

---

## 🔗 Ressources Officielles

- **Documentation Docker** : [https://docs.docker.com/](https://docs.docker.com/)
- **Docker Hub** (images officielles) : [https://hub.docker.com/](https://hub.docker.com/)
- **Docker Compose documentation** : [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
- **Forum communautaire** : [https://forums.docker.com/](https://forums.docker.com/)

---

## 🚀 Prochaines Étapes

Maintenant que Docker est installé et configuré, vous pouvez :

1. **Comprendre les concepts** → [Concepts Docker de base](02-concepts-docker.md)
2. **Apprendre les bonnes pratiques** → [Bonnes pratiques](03-bonnes-pratiques.md)
3. **Déployer votre première base de données** → Choisissez dans le [Sommaire](/SOMMAIRE.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
