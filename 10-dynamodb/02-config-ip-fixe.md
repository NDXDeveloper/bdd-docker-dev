# 10.2 - Configuration de DynamoDB Local avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans la fiche précédente, nous avons configuré DynamoDB Local de manière basique. Le conteneur obtient automatiquement une adresse IP attribuée par Docker.

**Pourquoi vouloir une IP fixe ?**

Imaginez que vous développez une application web avec plusieurs conteneurs Docker :
- 🗄️ Un conteneur pour DynamoDB Local
- 🌐 Un conteneur pour votre API (Node.js, Python, etc.)
- 🖥️ Un conteneur pour votre frontend (React, Angular, etc.)

**Le problème :** À chaque redémarrage, Docker peut changer l'adresse IP de vos conteneurs. Votre API ne sait plus où trouver la base de données !

**La solution :** Assigner une **adresse IP fixe** à votre conteneur DynamoDB. Comme ça, peu importe combien de fois vous redémarrez, l'IP reste toujours la même.

**Ce que vous allez apprendre :**
- Créer un réseau Docker personnalisé
- Attribuer une IP fixe à DynamoDB Local
- Faire communiquer plusieurs conteneurs entre eux
- Comprendre les réseaux Docker

---

## 🎯 Prérequis

✅ Avoir lu la [fiche 10.1 - Configuration basique](01-config-basique-docker-compose.md)
✅ Docker et Docker Compose installés
✅ Comprendre les bases de Docker
✅ 10 minutes de votre temps ⏱️

---

## 🌐 Comprendre les réseaux Docker

### Sans réseau personnalisé (configuration basique)

```
┌─────────────────────────────────────┐
│     Réseau par défaut (bridge)      │
│                                     │
│  ┌─────────────────────────────┐    │
│  │  DynamoDB                   │    │
│  │  IP: 172.17.0.2 (aléatoire) │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

**Problème :** L'IP `172.17.0.2` peut changer au prochain redémarrage.

### Avec réseau personnalisé (ce qu'on va faire)

```
┌───────────────────────────────────────┐
│   Réseau dynamodb_net (172.18.0.0/16) │
│                                       │
│  ┌──────────────────────────┐         │
│  │  DynamoDB                │         │
│  │  IP: 172.18.0.10 (FIXE!) │         │
│  └──────────────────────────┘         │
└───────────────────────────────────────┘
```

**Avantage :** L'IP `172.18.0.10` ne change **jamais** !

---

## 🚀 Étape 1 : Création du réseau Docker

Avant de créer notre fichier `docker-compose.yml`, nous devons créer un **réseau personnalisé** avec une plage d'adresses IP définie.

### Créer le réseau

Ouvrez votre terminal et exécutez cette commande **une seule fois** :

```bash
docker network create --subnet=172.18.0.0/16 dynamodb_net
```

**Explication de la commande :**
- `docker network create` : Crée un nouveau réseau Docker
- `--subnet=172.18.0.0/16` : Définit la plage d'adresses IP disponibles
  - `172.18.0.0/16` signifie : de `172.18.0.1` à `172.18.255.254`
- `dynamodb_net` : Nom de notre réseau personnalisé

**Sortie attendue :**
```
abc123def456...
```

C'est l'identifiant unique de votre réseau.

### Vérifier le réseau

```bash
docker network ls
```

**Sortie :**
```
NETWORK ID     NAME            DRIVER    SCOPE
abc123def456   dynamodb_net    bridge    local
...
```

Vous devez voir votre réseau `dynamodb_net` dans la liste.

### Inspecter le réseau

Pour voir les détails de votre réseau :

```bash
docker network inspect dynamodb_net
```

Vous verrez notamment :
```json
"Subnet": "172.18.0.0/16"
```

---

## 📝 Étape 2 : Configuration avec docker-compose

### A. Créer le dossier du projet

```bash
# Créer et entrer dans le dossier
mkdir dynamodb-ip-fixe
cd dynamodb-ip-fixe
```

### B. Créer le fichier docker-compose.yml

Créez un fichier `docker-compose.yml` avec ce contenu :

```yaml
version: '3.8'

services:
  dynamodb:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_ip_fixe
    restart: unless-stopped

    ports:
      - "8000:8000"

    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"

    volumes:
      - dynamodb_data:/data

    # Configuration réseau avec IP fixe
    networks:
      dynamodb_net:
        ipv4_address: 172.18.0.10

# Déclaration du volume
volumes:
  dynamodb_data:
    driver: local

# Déclaration du réseau (EXTERNE car créé manuellement)
networks:
  dynamodb_net:
    external: true
```

---

## 🔍 Comprendre les modifications

Comparons avec la configuration basique :

### 🆕 Nouvelle section : `networks` dans le service

```yaml
networks:
  dynamodb_net:
    ipv4_address: 172.18.0.10
```

**Explication :**
- `dynamodb_net` : Utilise le réseau que nous avons créé
- `ipv4_address: 172.18.0.10` : **Fixe** l'IP à `172.18.0.10`

**Pourquoi `.10` ?** Convention personnelle. Vous pourriez choisir :
- `172.18.0.5`
- `172.18.0.100`
- N'importe quelle IP dans la plage `172.18.0.1` à `172.18.255.254`

### 🆕 Déclaration du réseau global

```yaml
networks:
  dynamodb_net:
    external: true
```

**Explication :**
- `external: true` : Indique à Docker Compose que le réseau existe **déjà**
- Docker Compose ne tentera **pas** de créer ce réseau (il a déjà été créé manuellement)

---

## ▶️ Étape 3 : Démarrer DynamoDB avec IP fixe

Dans le dossier contenant votre `docker-compose.yml` :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating volume "dynamodb-ip-fixe_dynamodb_data" with local driver
Creating dynamodb_ip_fixe ... done
```

✅ DynamoDB Local démarre maintenant avec l'IP **172.18.0.10** !

---

## ✅ Étape 4 : Vérifier l'IP fixe

### Méthode 1 : Inspection du conteneur

```bash
docker inspect dynamodb_ip_fixe | grep "IPAddress"
```

**Sortie attendue :**
```
"IPAddress": "",
"IPAddress": "172.18.0.10",
```

Vous devez voir `172.18.0.10` !

### Méthode 2 : Inspection complète du réseau

```bash
docker network inspect dynamodb_net
```

Dans la section `"Containers"`, vous verrez :

```json
"Containers": {
    "abc123...": {
        "Name": "dynamodb_ip_fixe",
        "IPv4Address": "172.18.0.10/16",
        ...
    }
}
```

### Méthode 3 : Test de connectivité

```bash
# Depuis votre PC (via localhost)
curl http://localhost:8000/shell

# Depuis l'IP fixe
curl http://172.18.0.10:8000/shell
```

Les deux doivent fonctionner !

---

## 🔗 Étape 5 : Connexion depuis un autre conteneur

Maintenant que DynamoDB a une IP fixe, d'autres conteneurs peuvent s'y connecter facilement.

### Exemple : Conteneur Node.js qui se connecte à DynamoDB

Créez un fichier `docker-compose.yml` complet avec deux services :

```yaml
version: '3.8'

services:
  # Service DynamoDB
  dynamodb:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_ip_fixe
    restart: unless-stopped
    ports:
      - "8000:8000"
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"
    volumes:
      - dynamodb_data:/data
    networks:
      dynamodb_net:
        ipv4_address: 172.18.0.10

  # Service d'exemple : Application Node.js
  app:
    image: node:18-alpine
    container_name: app_nodejs
    restart: unless-stopped

    # Variables d'environnement pour la connexion
    environment:
      DYNAMODB_ENDPOINT: "http://172.18.0.10:8000"
      AWS_REGION: "us-east-1"
      AWS_ACCESS_KEY_ID: "fakeKey"
      AWS_SECRET_ACCESS_KEY: "fakeSecret"

    # Connecté au même réseau
    networks:
      - dynamodb_net

    # Commande qui maintient le conteneur actif
    command: tail -f /dev/null

volumes:
  dynamodb_data:

networks:
  dynamodb_net:
    external: true
```

**Points clés :**
- L'application utilise `DYNAMODB_ENDPOINT: "http://172.18.0.10:8000"`
- L'IP **172.18.0.10** est fixe et ne changera jamais
- Les deux conteneurs sont sur le même réseau `dynamodb_net`

### Tester la connexion

```bash
# Démarrer les deux services
docker-compose up -d

# Entrer dans le conteneur Node.js
docker exec -it app_nodejs sh

# Installer AWS SDK (dans le conteneur)
npm install -g aws-sdk

# Test de connexion (dans le conteneur)
node
> const AWS = require('aws-sdk');
> const dynamodb = new AWS.DynamoDB({
...   endpoint: 'http://172.18.0.10:8000',
...   region: 'us-east-1'
... });
> dynamodb.listTables({}, (err, data) => {
...   if (err) console.error(err);
...   else console.log('Connexion OK:', data);
... });
```

Si vous voyez `Connexion OK:`, tout fonctionne ! 🎉

---

## 🌍 Accès depuis votre machine hôte

Votre PC (la machine hôte) peut accéder à DynamoDB de **deux façons** :

### 1. Via localhost (recommandé)

```javascript
// Dans votre code sur votre PC
const dynamodb = new AWS.DynamoDB({
  endpoint: 'http://localhost:8000',
  region: 'us-east-1'
});
```

**Fonctionne car :** Le port 8000 du conteneur est mappé sur le port 8000 de votre PC.

### 2. Via l'IP fixe

```javascript
// Dans votre code sur votre PC
const dynamodb = new AWS.DynamoDB({
  endpoint: 'http://172.18.0.10:8000',
  region: 'us-east-1'
});
```

**Fonctionne car :** Votre PC peut accéder aux réseaux Docker bridge.

**⚠️ Attention :** La méthode via IP fixe peut ne pas fonctionner sur tous les systèmes (notamment Docker Desktop sur macOS/Windows avec certaines versions).

**Recommandation :** Utilisez `localhost` depuis votre PC, et l'IP fixe uniquement pour la communication entre conteneurs.

---

## 🛠️ Commandes utiles

### Voir les conteneurs sur le réseau

```bash
docker network inspect dynamodb_net | grep -A 5 "Containers"
```

### Tester la connectivité entre conteneurs

```bash
# Depuis le conteneur app_nodejs, pingez DynamoDB
docker exec app_nodejs ping -c 3 172.18.0.10
```

**Sortie attendue :**
```
PING 172.18.0.10 (172.18.0.10): 56 data bytes
64 bytes from 172.18.0.10: seq=0 ttl=64 time=0.123 ms
...
```

### Lister les réseaux Docker

```bash
docker network ls
```

### Voir tous les conteneurs d'un réseau

```bash
docker network inspect dynamodb_net --format '{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{println}}{{end}}'
```

---

## 🧹 Étape 6 : Suppression complète

### 1. Arrêter et supprimer les conteneurs

```bash
docker-compose down
```

### 2. Supprimer le volume (ATTENTION : supprime les données)

```bash
docker volume rm dynamodb-ip-fixe_dynamodb_data
```

ou

```bash
docker volume prune
```

### 3. Supprimer le réseau Docker

**Important :** Comme le réseau a été créé **manuellement** et est marqué comme `external: true`, Docker Compose ne le supprime **pas** automatiquement.

Vous devez le supprimer vous-même :

```bash
docker network rm dynamodb_net
```

**Vérification :**
```bash
docker network ls | grep dynamodb
```

Ne doit rien retourner.

---

## 📊 Comparaison : Avec vs Sans IP fixe

| Critère | Sans IP fixe | Avec IP fixe |
|---------|--------------|--------------|
| **Configuration** | Simple | Légèrement plus complexe |
| **IP du conteneur** | Aléatoire (change) | Fixe (ne change jamais) |
| **Communication inter-conteneurs** | Par nom de service | Par IP ou nom |
| **Prévisibilité** | ❌ Faible | ✅ Forte |
| **Cas d'usage** | Dev simple | Multi-conteneurs, microservices |
| **Nettoyage** | Automatique | Manuel (réseau externe) |

---

## 🎓 Concepts clés à retenir

### 🌐 Qu'est-ce qu'un réseau Docker ?

Un **réseau Docker** est comme un "réseau local virtuel" pour vos conteneurs :
- Les conteneurs peuvent communiquer entre eux
- Isolation par rapport à d'autres réseaux
- Contrôle des adresses IP

### 📍 Subnet (sous-réseau)

`172.18.0.0/16` signifie :
- **172.18** : Préfixe fixe
- **0.0** à **255.255** : Plage disponible
- `/16` : Nombre de bits du préfixe réseau

**Exemple de plage :**
- IP réseau : `172.18.0.0`
- Première IP utilisable : `172.18.0.1`
- Dernière IP utilisable : `172.18.255.254`
- Broadcast : `172.18.255.255`

### 🔒 Réseau `external: true`

Quand vous marquez un réseau comme `external: true` :
- Docker Compose **n'essaie pas** de le créer
- Vous devez le créer manuellement **avant** le `docker-compose up`
- Docker Compose **ne le supprime pas** lors du `docker-compose down`

---

## ❓ Problèmes courants

### Erreur : "network dynamodb_net not found"

**Cause :** Le réseau n'a pas été créé avant le `docker-compose up`.

**Solution :**
```bash
docker network create --subnet=172.18.0.0/16 dynamodb_net
docker-compose up -d
```

### Erreur : "address already in use"

**Cause :** L'IP `172.18.0.10` est déjà utilisée par un autre conteneur.

**Solution 1 :** Stoppez l'autre conteneur :
```bash
docker ps
docker stop <nom_conteneur_utilisant_cette_ip>
```

**Solution 2 :** Changez l'IP dans votre `docker-compose.yml` :
```yaml
ipv4_address: 172.18.0.11  # Au lieu de .10
```

### Le conteneur ne démarre pas

**Diagnostic :**
```bash
docker-compose logs dynamodb
```

**Vérifications :**
1. Le réseau existe bien : `docker network ls`
2. L'IP est dans la plage : `172.18.0.0/16`
3. Pas de conflit avec d'autres conteneurs

---

## 💡 Bonnes pratiques

### 1. Conventions de nommage

Utilisez des noms cohérents pour vos réseaux :
```bash
# Mauvais
docker network create mynet

# Bon
docker network create app_backend_net
```

### 2. Plages d'IP personnalisées

Évitez les plages communes :
- ❌ `172.17.0.0/16` (réseau Docker par défaut)
- ✅ `172.18.0.0/16`, `172.19.0.0/16`, `10.5.0.0/16`

### 3. Documentation

Documentez vos IP fixes dans un fichier `NETWORK.md` :

```markdown
## Réseaux Docker

### dynamodb_net (172.18.0.0/16)
- 172.18.0.10 : DynamoDB Local
- 172.18.0.20 : Application backend
- 172.18.0.30 : Redis cache
```

### 4. Évitez les IP très basses

Réservez les premières IP pour Docker :
- ❌ `172.18.0.1`, `172.18.0.2`
- ✅ À partir de `172.18.0.10`

---

## 🎯 Cas d'usage pratiques

### Microservices

```yaml
services:
  dynamodb:
    networks:
      app_net:
        ipv4_address: 172.18.0.10

  api-users:
    networks:
      app_net:
        ipv4_address: 172.18.0.20

  api-products:
    networks:
      app_net:
        ipv4_address: 172.18.0.30
```

### Tests d'intégration

Les tests peuvent utiliser `http://172.18.0.10:8000` pour des connexions prévisibles.

### Monitoring

Un conteneur de monitoring (Prometheus, Grafana) peut scraper `172.18.0.10:8000/metrics`.

---

## 🎯 Prochaines étapes

Maintenant que vous maîtrisez les IP fixes, explorez :

1. **[DynamoDB avec Admin UI](03-dynamodb-admin-ui.md)** - Interface graphique pour gérer vos tables
2. **[Utilisation avec AWS CLI](04-utilisation-aws-cli.md)** - Créer et manipuler des tables
3. **[Annexe B - Gestion des réseaux](../annexes/B-gestion-reseaux.md)** - Réseaux Docker avancés

---

## 📚 Ressources complémentaires

- 🌐 [Docker Networks Documentation](https://docs.docker.com/network/)
- 🔧 [Bridge Networks](https://docs.docker.com/network/bridge/)
- 📖 [Docker Compose Networking](https://docs.docker.com/compose/networking/)

---

## ✅ Checklist de fin

Avant de passer à la fiche suivante, vérifiez que vous savez :

- [ ] Créer un réseau Docker avec `docker network create`
- [ ] Configurer une IP fixe dans `docker-compose.yml`
- [ ] Comprendre `external: true` pour les réseaux
- [ ] Vérifier l'IP d'un conteneur avec `docker inspect`
- [ ] Faire communiquer deux conteneurs via IP fixe
- [ ] Supprimer un réseau Docker manuellement

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
⬅️ Fiche précédente : [10.1 - Configuration basique](01-config-basique-docker-compose.md)
➡️ Fiche suivante : [10.3 - DynamoDB avec Admin UI](03-dynamodb-admin-ui.md)

---

*Dernière mise à jour : Novembre 2025*
