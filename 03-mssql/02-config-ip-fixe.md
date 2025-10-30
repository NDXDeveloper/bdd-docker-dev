# 3.2 Configuration avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Par défaut, Docker attribue des adresses IP dynamiques aux conteneurs. Cela signifie que l'adresse IP de votre conteneur MS SQL Server peut changer à chaque redémarrage, ce qui peut poser problème si :

- Vous avez des applications qui se connectent directement via l'IP
- Vous voulez configurer des règles de pare-feu spécifiques
- Vous avez besoin d'une configuration réseau stable et prévisible
- Vous travaillez avec plusieurs conteneurs qui doivent communiquer entre eux

Dans ce tutoriel, nous allons apprendre à **assigner une adresse IP fixe** à votre conteneur MS SQL Server.

---

## 🎯 Ce que vous allez obtenir

À la fin de ce tutoriel, vous aurez :
- ✅ Un réseau Docker personnalisé avec une plage d'adresses définie
- ✅ Un conteneur MS SQL Server avec une IP fixe (exemple : `172.20.0.10`)
- ✅ La possibilité d'accéder à SQL Server via cette IP depuis d'autres conteneurs
- ✅ Un environnement réseau stable et prévisible

---

## 📋 Prérequis

Avant de commencer, assurez-vous d'avoir :
- Docker et Docker Compose installés
- Les connaissances de base du [tutoriel 3.1 - Configuration basique](01-config-basique-docker-compose.md)
- Un terminal ouvert

> 💡 **Note** : Si vous avez déjà un conteneur MS SQL en cours d'exécution, arrêtez-le avec `docker-compose down` avant de continuer.

---

## 🧠 Comprendre les réseaux Docker

### Réseau par défaut (bridge)

Quand vous créez un conteneur sans spécifier de réseau, Docker utilise le réseau **bridge** par défaut :
- Adresses IP attribuées automatiquement (ex: 172.17.0.2, 172.17.0.3...)
- Ces adresses peuvent changer à chaque redémarrage
- Difficile de prévoir quelle IP sera attribuée

### Réseau personnalisé

En créant notre propre réseau, nous pouvons :
- Définir la plage d'adresses IP (subnet)
- Assigner des IP fixes à nos conteneurs
- Isoler nos conteneurs du réseau par défaut
- Faciliter la communication entre conteneurs

---

## 🛠️ Étape 1 : Créer un réseau Docker personnalisé

Nous allons créer un réseau nommé `mssql_net` avec une plage d'adresses définie.

### A. Créer le réseau

Ouvrez un terminal et exécutez :

```bash
docker network create --subnet=172.20.0.0/16 mssql_net
```

**Détails de la commande :**
- `docker network create` : Crée un nouveau réseau Docker
- `--subnet=172.20.0.0/16` : Définit la plage d'adresses IP
  - `172.20.0.0` : Adresse de base du réseau
  - `/16` : Masque de sous-réseau (permet 65534 adresses utilisables)
  - Plage disponible : de `172.20.0.1` à `172.20.255.254`
- `mssql_net` : Nom du réseau

**Sortie attendue :**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```
(Un identifiant unique du réseau)

### B. Vérifier que le réseau a été créé

```bash
docker network ls
```

**Sortie attendue :**
```
NETWORK ID     NAME        DRIVER    SCOPE
...
a1b2c3d4e5f6   mssql_net   bridge    local
```

### C. Inspecter le réseau (optionnel)

Pour voir les détails du réseau :

```bash
docker network inspect mssql_net
```

Vous verrez notamment :
```json
"Subnet": "172.20.0.0/16",
"Gateway": "172.20.0.1"
```

---

## 📁 Étape 2 : Créer le dossier du projet

Si ce n'est pas déjà fait, créons un dossier pour ce projet :

```bash
# Créer un dossier
mkdir mssql-ip-fixe

# Se placer dedans
cd mssql-ip-fixe
```

---

## 📝 Étape 3 : Créer le fichier docker-compose.yml

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  mssql:
    # Image officielle MS SQL Server 2022
    image: mcr.microsoft.com/mssql/server:2022-latest

    # Nom du conteneur
    container_name: mssql_with_ip

    # Redémarrage automatique
    restart: unless-stopped

    # Variables d'environnement
    environment:
      # Accepter le contrat de licence
      ACCEPT_EULA: "Y"

      # Édition Developer (gratuite pour développement)
      MSSQL_PID: "Developer"

      # Mot de passe administrateur
      # ⚠️ CHANGEZ CE MOT DE PASSE !
      SA_PASSWORD: "VotreMotDePasse123!"

    # Exposition du port
    ports:
      - "1433:1433"

    # Volume pour la persistance des données
    volumes:
      - mssql_data:/var/opt/mssql

    # Configuration réseau avec IP fixe
    networks:
      mssql_net:
        ipv4_address: 172.20.0.10

# Déclaration du volume
volumes:
  mssql_data:

# Déclaration du réseau externe (créé précédemment)
networks:
  mssql_net:
    external: true
```

---

## 🔍 Comprendre la configuration réseau

### Section networks du service

```yaml
networks:
  mssql_net:
    ipv4_address: 172.20.0.10
```

- `mssql_net` : Le nom du réseau que nous avons créé
- `ipv4_address: 172.20.0.10` : L'adresse IP fixe assignée au conteneur
- Cette IP doit être dans la plage du subnet (172.20.0.0/16)
- Évitez d'utiliser `.0` ou `.1` (réservées pour le réseau et la gateway)

### Déclaration du réseau externe

```yaml
networks:
  mssql_net:
    external: true
```

- `external: true` : Indique que le réseau existe déjà (nous l'avons créé à l'Étape 1)
- Docker ne tentera pas de créer ce réseau
- Si le réseau n'existe pas, vous aurez une erreur

> ⚠️ **Important** : Le réseau doit être créé AVANT de lancer `docker-compose up`.

---

## ▶️ Étape 4 : Démarrer le conteneur

Dans votre terminal, depuis le dossier `mssql-ip-fixe`, exécutez :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
[+] Running 2/2
 ✔ Volume "mssql-ip-fixe_mssql_data"  Created
 ✔ Container mssql_with_ip            Started
```

---

## ✅ Étape 5 : Vérifier l'IP fixe

### A. Vérifier l'adresse IP du conteneur

```bash
docker inspect mssql_with_ip | grep "IPAddress"
```

**Sortie attendue :**
```
"SecondaryIPAddresses": null,
"IPAddress": "",
"IPAddress": "172.20.0.10",
```

Vous devriez voir `"IPAddress": "172.20.0.10"` confirmant l'IP fixe.

### B. Méthode alternative avec docker network

```bash
docker network inspect mssql_net
```

Cherchez la section `Containers`, vous verrez :
```json
"Containers": {
    "abc123...": {
        "Name": "mssql_with_ip",
        "IPv4Address": "172.20.0.10/16",
        ...
    }
}
```

### C. Vérifier la connectivité

```bash
docker exec mssql_with_ip hostname -I
```

Cela affichera : `172.20.0.10`

---

## 🔌 Étape 6 : Se connecter via l'IP fixe

### A. Depuis la ligne de commande

**1. Se connecter au conteneur :**

```bash
docker exec -it mssql_with_ip /opt/mssql-tools/bin/sqlcmd \
  -S 172.20.0.10 \
  -U sa \
  -P "VotreMotDePasse123!"
```

> 💡 Notez qu'on utilise `-S 172.20.0.10` au lieu de `localhost`.

**2. Tester une requête :**

```sql
SELECT @@SERVERNAME AS 'Nom du serveur', @@VERSION AS 'Version';
GO
```

### B. Depuis un client SQL (Azure Data Studio, SSMS, DBeaver)

**Paramètres de connexion :**

| Paramètre | Valeur |
|-----------|--------|
| **Serveur** | `172.20.0.10` OU `localhost` |
| **Port** | `1433` |
| **Authentification** | SQL Server Authentication |
| **Utilisateur** | `sa` |
| **Mot de passe** | `VotreMotDePasse123!` |

> 💡 **Note** : `localhost` fonctionne toujours grâce au mapping de ports (`-p 1433:1433`). L'IP fixe est surtout utile pour la communication entre conteneurs.

---

## 🌐 Cas d'usage : Communication entre conteneurs

L'intérêt principal d'une IP fixe est de faciliter la communication entre plusieurs conteneurs Docker.

### Exemple : Connecter une application web à SQL Server

**docker-compose.yml complet avec une application :**

```yaml
version: '3.8'

services:
  # Service MS SQL Server
  mssql:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: mssql_with_ip
    restart: unless-stopped
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Developer"
      SA_PASSWORD: "VotreMotDePasse123!"
    ports:
      - "1433:1433"
    volumes:
      - mssql_data:/var/opt/mssql
    networks:
      mssql_net:
        ipv4_address: 172.20.0.10

  # Application exemple (Node.js)
  app:
    image: node:18
    container_name: mon_application
    working_dir: /app
    command: tail -f /dev/null  # Garde le conteneur actif
    networks:
      mssql_net:
        ipv4_address: 172.20.0.20
    environment:
      # L'application peut utiliser l'IP fixe pour se connecter
      DB_HOST: "172.20.0.10"
      DB_PORT: "1433"
      DB_USER: "sa"
      DB_PASSWORD: "VotreMotDePasse123!"

volumes:
  mssql_data:

networks:
  mssql_net:
    external: true
```

**Avantage :** L'application sait toujours que la base de données est sur `172.20.0.10`, même après des redémarrages.

---

## 🎨 Personnaliser la configuration réseau

### Utiliser une plage d'IP différente

Vous pouvez choisir n'importe quelle plage d'IP privée :

```bash
# Plage 10.x.x.x
docker network create --subnet=10.5.0.0/16 mssql_net

# Plage 192.168.x.x
docker network create --subnet=192.168.100.0/24 mssql_net
```

**Plages privées recommandées :**
- `10.0.0.0` à `10.255.255.255` (10.0.0.0/8)
- `172.16.0.0` à `172.31.255.255` (172.16.0.0/12)
- `192.168.0.0` à `192.168.255.255` (192.168.0.0/16)

### Adapter l'IP du conteneur

Si vous changez la plage du réseau, adaptez l'IP du conteneur :

```yaml
networks:
  mssql_net:
    ipv4_address: 10.5.0.10  # Doit correspondre au subnet
```

---

## 🛠️ Commandes de gestion

### Arrêter le conteneur

```bash
docker-compose stop
```

### Redémarrer le conteneur

```bash
docker-compose start
```

**Vérifiez que l'IP est conservée :**
```bash
docker inspect mssql_with_ip | grep "IPAddress"
```

Résultat : L'IP sera toujours `172.20.0.10` ! 🎉

### Voir les logs

```bash
docker-compose logs -f
```

---

## 🗑️ Suppression complète

### Étape 1 : Arrêter et supprimer le conteneur

```bash
docker-compose down
```

### Étape 2 : Supprimer le volume (⚠️ perte de données)

```bash
docker volume rm mssql-ip-fixe_mssql_data
```

Ou avec l'option `-v` directement :
```bash
docker-compose down -v
```

### Étape 3 : Supprimer le réseau

```bash
docker network rm mssql_net
```

**Vérifier la suppression :**
```bash
docker network ls
```

Le réseau `mssql_net` ne doit plus apparaître.

---

## 🔍 Dépannage

### Problème : Erreur "network mssql_net not found"

**Cause :** Le réseau n'a pas été créé avant `docker-compose up`.

**Solution :**
```bash
docker network create --subnet=172.20.0.0/16 mssql_net
docker-compose up -d
```

### Problème : Erreur "address already in use"

**Cause :** Un autre conteneur utilise déjà l'IP `172.20.0.10`.

**Solution 1 - Trouver le conteneur :**
```bash
docker network inspect mssql_net
```

**Solution 2 - Changer l'IP dans docker-compose.yml :**
```yaml
ipv4_address: 172.20.0.11  # Nouvelle IP
```

### Problème : Impossible de se connecter via l'IP fixe

**Vérifier que le conteneur est sur le bon réseau :**
```bash
docker inspect mssql_with_ip | grep "NetworkMode"
```

**Vérifier l'IP assignée :**
```bash
docker inspect mssql_with_ip | grep "IPAddress"
```

**Tester la connectivité réseau :**
```bash
docker exec mssql_with_ip ping -c 3 172.20.0.1  # Gateway
```

### Problème : Le réseau ne se supprime pas

**Cause :** Des conteneurs utilisent encore ce réseau.

**Solution - Lister les conteneurs sur ce réseau :**
```bash
docker network inspect mssql_net
```

**Arrêter tous les conteneurs concernés :**
```bash
docker stop $(docker ps -q --filter network=mssql_net)
```

**Puis supprimer le réseau :**
```bash
docker network rm mssql_net
```

---

## 📊 Schéma récapitulatif

```
┌─────────────────────────────────────────────────────────┐
│  Réseau Docker : mssql_net (172.20.0.0/16)              │
│                                                         │
│  ┌──────────────────────────────────────────┐           │
│  │  Gateway : 172.20.0.1                    │           │
│  └──────────────────────────────────────────┘           │
│                                                         │
│  ┌──────────────────────────────────────────┐           │
│  │  Conteneur : mssql_with_ip               │           │
│  │  IP fixe : 172.20.0.10                   │           │
│  │  Port : 1433                             │           │
│  │  Volume : mssql_data                     │           │
│  └──────────────────────────────────────────┘           │
│                                                         │
│  ┌──────────────────────────────────────────┐           │
│  │  Conteneur : mon_application (optionnel) │           │
│  │  IP fixe : 172.20.0.20                   │           │
│  └──────────────────────────────────────────┘           │
│                                                         │
└─────────────────────────────────────────────────────────┘
          │
          │ Port mapping (-p 1433:1433)
          │
┌─────────▼───────────────────────────────────────────────┐
│  Machine hôte (localhost:1433)                          │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 Bonnes pratiques

### 1. Documenter les IP utilisées

Créez un fichier `NETWORK.md` dans votre projet :

```markdown
# Configuration réseau

## Réseau : mssql_net
- Subnet : 172.20.0.0/16
- Gateway : 172.20.0.1

## IP assignées
- 172.20.0.10 : MS SQL Server (mssql_with_ip)
- 172.20.0.20 : Application Node.js (mon_application)
- 172.20.0.30 : Redis (à venir)
```

### 2. Utiliser des plages cohérentes

Si vous avez plusieurs projets, utilisez des plages différentes :
- Projet A : `172.20.0.0/16`
- Projet B : `172.21.0.0/16`
- Projet C : `172.22.0.0/16`

### 3. Éviter les conflits d'IP

Ne pas utiliser :
- `.0` : Adresse réseau (ex: 172.20.0.0)
- `.1` : Gateway (ex: 172.20.0.1)
- `.255` : Broadcast dans un /24 (ex: 172.20.0.255)

**Plage recommandée :** `.10` à `.250`

### 4. Nommer explicitement les réseaux

```yaml
networks:
  mssql_net:
    external: true
    name: mon_projet_mssql_network
```

---

## 🎯 Quand utiliser une IP fixe ?

### ✅ Situations où c'est utile

1. **Communication inter-conteneurs** : Plusieurs services Docker doivent se parler
2. **Configuration d'applications** : L'application a des fichiers de config avec des IP
3. **Règles de sécurité** : Pare-feu ou ACL basés sur des IP
4. **Environnement de développement stable** : Vous voulez toujours la même IP
5. **Debugging réseau** : Plus facile de diagnostiquer avec des IP prévisibles

### ❌ Situations où ce n'est PAS nécessaire

1. **Service unique** : Un seul conteneur SQL Server accessible via `localhost`
2. **Noms de services** : Docker Compose permet d'utiliser les noms de services
3. **Production cloud** : Les orchestrateurs (Kubernetes, Swarm) gèrent le réseau
4. **Complexité excessive** : Si ça complique votre setup sans raison

> 💡 **Astuce** : Dans Docker Compose, vous pouvez souvent utiliser le nom du service au lieu de l'IP :
> ```yaml
> DB_HOST: "mssql"  # Au lieu de "172.20.0.10"
> ```

---

## 📚 Aller plus loin

- **[Gestion des utilisateurs et logins](03-gestion-utilisateurs-logins.md)** : Créer des utilisateurs SQL Server avec des permissions spécifiques
- **[Restauration de backup](04-restauration-backup.md)** : Importer des bases de données existantes
- **[Annexe B - Gestion des réseaux](../annexes/B-gestion-reseaux.md)** : Approfondir les concepts de réseaux Docker

---

## 📝 Récapitulatif

Vous avez appris à :
- ✅ Créer un réseau Docker personnalisé avec une plage d'IP définie
- ✅ Assigner une adresse IP fixe à un conteneur MS SQL Server
- ✅ Configurer docker-compose.yml pour utiliser un réseau externe
- ✅ Vérifier et tester l'IP fixe
- ✅ Gérer la communication entre conteneurs
- ✅ Dépanner les problèmes réseau courants

---

## 🎯 Points clés à retenir

1. **Le réseau doit être créé AVANT** `docker-compose up`
2. Utilisez `external: true` dans la déclaration du réseau
3. L'IP fixe doit être dans la plage du subnet
4. Les IP `.0`, `.1` et parfois `.255` sont réservées
5. L'IP est conservée même après redémarrage du conteneur
6. `localhost` fonctionne toujours grâce au port mapping
7. Documentez vos IP pour éviter les conflits

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
