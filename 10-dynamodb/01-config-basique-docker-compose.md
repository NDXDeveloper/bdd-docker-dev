# 10.1 - Configuration basique de DynamoDB Local avec docker-compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**DynamoDB** est une base de données NoSQL créée par Amazon Web Services (AWS). Elle est utilisée pour stocker des données sous forme de **documents** (comme MongoDB) avec une structure clé-valeur très performante.

**DynamoDB Local** est une version de DynamoDB que vous pouvez faire tourner sur votre ordinateur (au lieu du cloud AWS). C'est parfait pour :
- 🧪 **Développer et tester** des applications sans frais AWS
- 📚 **Apprendre** à utiliser DynamoDB gratuitement
- 🚀 **Prototyper** rapidement sans configuration cloud

**Ce que vous allez apprendre :**
- Démarrer DynamoDB Local avec Docker en moins de 5 minutes
- Comprendre la configuration de base
- Vérifier que tout fonctionne correctement
- Arrêter et supprimer proprement votre base de données

---

## 🎯 Prérequis

Avant de commencer, assurez-vous d'avoir :

✅ **Docker** installé sur votre machine ([voir guide installation](../00-introduction/01-prerequis.md))
✅ **Docker Compose** (inclus avec Docker Desktop)
✅ Un terminal / invite de commandes
✅ 5 minutes de votre temps ⏱️

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🚀 Étape 1 : Création du fichier de configuration

### A. Créer le dossier du projet

Ouvrez votre terminal et créez un nouveau dossier pour ce projet :

```bash
# Créer et entrer dans le dossier
mkdir dynamodb-local
cd dynamodb-local
```

### B. Créer le fichier docker-compose.yml

Créez un fichier nommé `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  dynamodb:
    # Image officielle DynamoDB Local d'Amazon
    image: amazon/dynamodb-local:latest

    # Nom du conteneur (pour le retrouver facilement)
    container_name: dynamodb_local

    # Redémarrage automatique si le conteneur s'arrête
    restart: unless-stopped

    # Ports exposés
    ports:
      # Port hôte : Port conteneur
      # DynamoDB Local écoute sur le port 8000
      - "8000:8000"

    # Options de démarrage de DynamoDB
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"

    # Volume pour la persistance des données
    volumes:
      # Stocke les données dans un volume Docker nommé
      - dynamodb_data:/data

# Déclaration du volume
volumes:
  dynamodb_data:
    driver: local
```

---

## 📝 Comprendre la configuration

Décortiquons chaque partie du fichier :

### 🔍 Section `services`

```yaml
services:
  dynamodb:
    image: amazon/dynamodb-local:latest
```
- **image** : L'image Docker officielle fournie par Amazon
- **latest** : Toujours la dernière version disponible

### 🏷️ Nom du conteneur

```yaml
container_name: dynamodb_local
```
- Donne un nom fixe au conteneur (plus facile à gérer qu'un ID aléatoire)

### 🔌 Ports

```yaml
ports:
  - "8000:8000"
```
- **8000** (gauche) : Port sur votre PC
- **8000** (droite) : Port dans le conteneur
- Vous accéderez à DynamoDB via `http://localhost:8000`

### ⚙️ Commande de démarrage

```yaml
command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"
```

Options importantes :
- **-jar DynamoDBLocal.jar** : Lance DynamoDB Local
- **-sharedDb** : Utilise une base de données partagée (recommandé pour le développement)
- **-dbPath /data** : Stocke les données dans le dossier `/data` du conteneur

### 💾 Volumes

```yaml
volumes:
  - dynamodb_data:/data
```
- Crée un volume Docker nommé `dynamodb_data`
- **Persistance** : Vos données survivent même si vous supprimez le conteneur
- Mappé sur `/data` dans le conteneur (là où DynamoDB stocke ses fichiers)

---

## ▶️ Étape 2 : Démarrer DynamoDB Local

Dans le dossier contenant votre `docker-compose.yml`, lancez :

```bash
docker-compose up -d
```

**Explication de la commande :**
- `up` : Démarre les services définis dans le fichier
- `-d` : Mode "détaché" (tourne en arrière-plan)

**Sortie attendue :**
```
Creating network "dynamodb-local_default" with the default driver
Creating volume "dynamodb-local_dynamodb_data" with local driver
Creating dynamodb_local ... done
```

✅ **C'est tout !** Votre base de données DynamoDB Local tourne maintenant.

---

## ✅ Étape 3 : Vérifier que tout fonctionne

### 🔎 Vérifier le statut du conteneur

```bash
docker-compose ps
```

**Sortie attendue :**
```
      Name                    Command              State           Ports
-----------------------------------------------------------------------------------
dynamodb_local   java -jar DynamoDBLocal.jar ...  Up      0.0.0.0:8000->8000/tcp
```

Le statut doit être **"Up"** (en cours d'exécution).

### 📋 Voir les logs

```bash
docker-compose logs dynamodb
```

ou en temps réel :

```bash
docker-compose logs -f dynamodb
```

**Sortie attendue :**
```
Initializing DynamoDB Local with the following configuration:
Port:   8000
InMemory:       false
DbPath: /data
SharedDb:       true
```

Appuyez sur `Ctrl+C` pour quitter le mode logs en temps réel.

### 🌐 Test via navigateur

Ouvrez votre navigateur et allez sur :

```
http://localhost:8000/shell
```

Vous devriez voir l'interface **DynamoDB JavaScript Shell** s'afficher.

> ⚠️ **Note :** Cette interface est basique. Pour une vraie utilisation, vous utiliserez soit le AWS CLI, soit un client graphique (voir fiches suivantes).

---

## 🔗 Se connecter à DynamoDB Local

DynamoDB Local est maintenant accessible depuis :

| Paramètre | Valeur |
|-----------|--------|
| **Endpoint (URL)** | `http://localhost:8000` |
| **Région AWS** | `us-east-1` (ou n'importe laquelle, c'est local) |
| **Access Key ID** | `fakeAccessKeyId` (n'importe quelle valeur) |
| **Secret Access Key** | `fakeSecretAccessKey` (n'importe quelle valeur) |

**Important :** DynamoDB Local **n'a pas d'authentification réelle**. Les clés AWS sont juste requises par les clients, mais elles ne sont pas vérifiées.

---

## 🛠️ Commandes utiles

### Arrêter DynamoDB (sans supprimer les données)

```bash
docker-compose stop
```

### Redémarrer DynamoDB

```bash
docker-compose start
```

### Voir les logs en direct

```bash
docker-compose logs -f dynamodb
```

### Accéder au conteneur en ligne de commande

```bash
docker exec -it dynamodb_local bash
```

(Tapez `exit` pour sortir)

---

## 🧹 Étape 4 : Suppression complète (Nettoyage)

Quand vous avez terminé et voulez tout supprimer :

### 1. Arrêter et supprimer le conteneur

```bash
docker-compose down
```

**Sortie :**
```
Stopping dynamodb_local ... done
Removing dynamodb_local ... done
Removing network dynamodb-local_default
```

⚠️ **Attention :** Cela ne supprime **PAS** les données (le volume existe encore).

### 2. Supprimer le volume de données

**ATTENTION : Cette action est IRRÉVERSIBLE et supprime toutes vos données !**

```bash
docker volume rm dynamodb-local_dynamodb_data
```

ou plus simple (supprime tous les volumes non utilisés) :

```bash
docker volume prune
```

### Vérification du nettoyage

```bash
# Vérifier qu'il n'y a plus de conteneur
docker ps -a | grep dynamodb

# Vérifier qu'il n'y a plus de volume
docker volume ls | grep dynamodb
```

Les deux commandes ne doivent rien retourner.

---

## 🎓 Concepts clés à retenir

### 📦 Qu'est-ce qu'un volume Docker ?

Un **volume** est un espace de stockage géré par Docker, indépendant du conteneur :
- ✅ Les données **persistent** même si vous supprimez le conteneur
- ✅ Peut être **partagé** entre plusieurs conteneurs
- ✅ **Performance** optimale par rapport aux bind mounts

### 🌐 Pourquoi le port 8000 ?

DynamoDB Local utilise par défaut le port **8000** (différent du DynamoDB AWS qui est sur HTTPS). C'est le port standard pour tous les projets de développement.

### 🔄 Mode sharedDb vs. mode par défaut

| Mode | Description | Usage |
|------|-------------|-------|
| **-sharedDb** | Une seule base pour tous les clients | ✅ Recommandé pour le dev |
| **Par défaut** | Une base par combinaison Access Key + Région | Simulation AWS plus réaliste |

---

## ❓ Problèmes courants

### Le port 8000 est déjà utilisé

**Symptôme :** Erreur `port is already allocated`

**Solution :** Changez le port dans le `docker-compose.yml` :

```yaml
ports:
  - "8001:8000"  # Utilisez 8001 sur votre PC au lieu de 8000
```

Puis accédez via `http://localhost:8001`.

### Le conteneur s'arrête immédiatement

**Diagnostic :**
```bash
docker-compose logs dynamodb
```

**Causes fréquentes :**
- Problème de syntaxe dans le `docker-compose.yml`
- Manque de mémoire (rare)

### Les données ne persistent pas

**Vérification :**
```bash
docker volume ls
```

Vous devez voir un volume `dynamodb-local_dynamodb_data`.

**Solution :** Assurez-vous que la section `volumes:` est bien présente dans votre fichier.

---

## 🎯 Prochaines étapes

Maintenant que vous avez DynamoDB Local qui tourne, vous pouvez :

1. **[Configuration avec IP fixe](02-config-ip-fixe.md)** - Pour des setups multi-conteneurs
2. **[DynamoDB avec Admin UI](03-dynamodb-admin-ui.md)** - Interface graphique pour gérer vos tables
3. **[Utilisation avec AWS CLI](04-utilisation-aws-cli.md)** - Créer des tables et insérer des données

---

## 💡 Astuces pro

### Alias pour gagner du temps

Ajoutez ces alias dans votre `.bashrc` ou `.zshrc` :

```bash
alias ddb-up='docker-compose up -d'
alias ddb-down='docker-compose down'
alias ddb-logs='docker-compose logs -f dynamodb'
alias ddb-shell='docker exec -it dynamodb_local bash'
```

### Fichier .gitignore

Si vous versionnez votre projet avec Git, créez un `.gitignore` :

```
# Ignorer les données locales
data/

# Ignorer les fichiers de config sensibles
.env
```

---

## 📚 Ressources complémentaires

- 📖 [Documentation officielle DynamoDB Local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html)
- 🐙 [Image Docker amazon/dynamodb-local](https://hub.docker.com/r/amazon/dynamodb-local)
- 🔧 [AWS CLI pour DynamoDB](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/)
- 📺 [Guide vidéo DynamoDB (AWS)](https://www.youtube.com/results?search_query=dynamodb+tutorial)

---

## ✅ Checklist de fin

Avant de passer à la fiche suivante, vérifiez que vous savez :

- [ ] Créer un fichier `docker-compose.yml` pour DynamoDB
- [ ] Démarrer et arrêter le conteneur
- [ ] Consulter les logs
- [ ] Accéder à DynamoDB via `http://localhost:8000`
- [ ] Supprimer proprement le conteneur et les données

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
➡️ Fiche suivante : [10.2 - Configuration avec IP fixe](02-config-ip-fixe.md)

---

*Dernière mise à jour : Novembre 2025*
