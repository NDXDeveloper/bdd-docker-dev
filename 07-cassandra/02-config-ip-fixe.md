# 7.2 Configuration avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Par défaut, Docker attribue des **adresses IP dynamiques** aux conteneurs. Cela signifie que l'adresse IP de votre conteneur Cassandra peut changer à chaque redémarrage.

**Pourquoi utiliser une IP fixe ?**

- 🔗 **Connexions fiables** : D'autres conteneurs peuvent se connecter à Cassandra via une IP qui ne change jamais
- 🎯 **Configuration simplifiée** : Plus besoin de chercher l'IP à chaque démarrage
- 📝 **Hardcoder l'IP** : Utile dans les fichiers de configuration d'applications
- 🧪 **Tests reproductibles** : Même environnement réseau à chaque fois

**Cas d'usage typiques :**
- Application qui se connecte à Cassandra via IP (pas via nom de conteneur)
- Plusieurs services qui doivent communiquer avec Cassandra
- Tests automatisés nécessitant des IPs prévisibles
- Apprentissage des réseaux Docker

---

## 🎯 Ce que vous allez apprendre

À la fin de cette fiche, vous saurez :
- ✅ Créer un réseau Docker personnalisé avec sous-réseau
- ✅ Configurer Cassandra avec une adresse IP fixe (`172.20.0.10`)
- ✅ Vérifier l'attribution de l'IP
- ✅ Se connecter à Cassandra via l'IP fixe
- ✅ Comprendre la configuration réseau de Cassandra

---

## 📦 Prérequis

Avant de commencer, assurez-vous d'avoir :

- **Docker** installé (version 20.10+)
- **Docker Compose** installé (version 2.0+)
- Avoir lu la [fiche 7.1 - Configuration basique](01-config-noeud-unique.md)

**Vérification rapide :**
```bash
docker --version
docker-compose --version
```

---

## 🌐 Étape 1 : Création du réseau Docker dédié

Pour assigner une IP fixe à un conteneur, nous devons d'abord créer un **réseau Docker personnalisé** avec un sous-réseau défini.

### Créer le réseau

Ouvrez un terminal et exécutez :

```bash
docker network create --subnet=172.20.0.0/16 cassandra_net
```

**Explication de la commande :**

| Élément | Description |
|---------|-------------|
| `docker network create` | Crée un nouveau réseau Docker |
| `--subnet=172.20.0.0/16` | Définit la plage d'adresses IP disponibles |
| `cassandra_net` | Nom du réseau (vous pouvez le changer) |

**Comprendre le sous-réseau :**
- `172.20.0.0/16` : Plage allant de `172.20.0.0` à `172.20.255.255`
- Cela donne **65 534 adresses IP disponibles** (largement suffisant !)
- Nous utiliserons `172.20.0.10` pour Cassandra

### Vérifier la création du réseau

```bash
docker network ls
```

**Vous devriez voir :**
```
NETWORK ID     NAME            DRIVER    SCOPE
abc123...      bridge          bridge    local
def456...      cassandra_net   bridge    local
ghi789...      host            host      local
```

### Inspecter le réseau (optionnel)

```bash
docker network inspect cassandra_net
```

**Résultat :**
```json
[
    {
        "Name": "cassandra_net",
        "Driver": "bridge",
        "IPAM": {
            "Config": [
                {
                    "Subnet": "172.20.0.0/16"
                }
            ]
        }
    }
]
```

---

## 📝 Étape 2 : Configuration du `docker-compose.yml`

Créez un nouveau dossier pour ce projet :

```bash
mkdir cassandra-ip-fixe
cd cassandra-ip-fixe
```

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  cassandra:
    image: cassandra:4.1
    container_name: cassandra_ip_fixe
    restart: unless-stopped

    environment:
      # Configuration du cluster
      CASSANDRA_CLUSTER_NAME: 'DevCluster'
      CASSANDRA_DC: 'datacenter1'
      CASSANDRA_RACK: 'rack1'

      # IMPORTANT : Configuration réseau pour IP fixe
      # Écouter sur toutes les interfaces
      CASSANDRA_LISTEN_ADDRESS: 'auto'
      # Adresse pour les clients CQL
      CASSANDRA_RPC_ADDRESS: '0.0.0.0'
      # Adresse de broadcast (celle que les clients utilisent)
      CASSANDRA_BROADCAST_ADDRESS: '172.20.0.10'
      # Nœud seed (point d'entrée du cluster)
      CASSANDRA_SEEDS: '172.20.0.10'

    ports:
      # Port CQL (Cassandra Query Language)
      - "9042:9042"
      # Port JMX (monitoring)
      - "7199:7199"

    volumes:
      # Volume pour persister les données
      - cassandra_data:/var/lib/cassandra

    # Configuration réseau : IP fixe
    networks:
      cassandra_net:
        ipv4_address: 172.20.0.10

    healthcheck:
      test: ["CMD-SHELL", "cqlsh -e 'describe cluster'"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s

# Déclaration du volume
volumes:
  cassandra_data:
    driver: local

# Déclaration du réseau externe
networks:
  cassandra_net:
    external: true
```

---

## 🔍 Explications détaillées

### Variables d'environnement pour le réseau

| Variable | Valeur | Description |
|----------|--------|-------------|
| `CASSANDRA_LISTEN_ADDRESS` | `auto` | Interface d'écoute (auto-détection) |
| `CASSANDRA_RPC_ADDRESS` | `0.0.0.0` | Écoute sur toutes les interfaces pour CQL |
| `CASSANDRA_BROADCAST_ADDRESS` | `172.20.0.10` | Adresse annoncée aux clients |
| `CASSANDRA_SEEDS` | `172.20.0.10` | Nœud seed (ici, lui-même) |

### Section `networks`

```yaml
networks:
  cassandra_net:
    ipv4_address: 172.20.0.10
```

- **`cassandra_net`** : Utilise le réseau que nous avons créé
- **`ipv4_address`** : Assigne l'IP fixe `172.20.0.10`

### Réseau externe

```yaml
networks:
  cassandra_net:
    external: true
```

- **`external: true`** : Indique que le réseau existe déjà (créé à l'étape 1)
- Docker Compose ne tentera pas de le créer

---

## ▶️ Étape 3 : Lancement du conteneur

Lancez Cassandra avec Docker Compose :

```bash
docker-compose up -d
```

**Attendez le démarrage complet** (30 secondes à 2 minutes) :

```bash
docker-compose logs -f cassandra
```

Attendez de voir :
```
Starting listening for CQL clients on /0.0.0.0:9042
Node is now in NORMAL state
```

Appuyez sur **Ctrl+C** pour sortir des logs.

---

## ✅ Étape 4 : Vérification de l'IP fixe

### Méthode 1 : Inspection du conteneur

```bash
docker inspect cassandra_ip_fixe | grep "IPAddress"
```

**Résultat attendu :**
```json
"IPAddress": "",
"SecondaryIPAddresses": null,
"IPAddress": "172.20.0.10"
```

✅ Vous devez voir `"IPAddress": "172.20.0.10"`.

### Méthode 2 : Avec Docker Compose

```bash
docker-compose exec cassandra hostname -i
```

**Résultat attendu :**
```
172.20.0.10
```

### Méthode 3 : État du cluster

```bash
docker exec cassandra_ip_fixe nodetool status
```

**Résultat attendu :**
```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address      Load       Tokens  Owns    Host ID      Rack
UN  172.20.0.10  108.5 KiB  16      100.0%  abc123...    rack1
```

✅ La colonne **Address** doit afficher `172.20.0.10`.

---

## 🔌 Étape 5 : Connexion à Cassandra

### Connexion depuis le conteneur (localhost)

```bash
docker exec -it cassandra_ip_fixe cqlsh
```

Vous êtes connecté via `127.0.0.1:9042` (localhost du conteneur).

### Connexion via l'IP fixe

```bash
docker exec -it cassandra_ip_fixe cqlsh 172.20.0.10
```

**Résultat :**
```
Connected to DevCluster at 172.20.0.10:9042
[cqlsh 6.1.0 | Cassandra 4.1.x | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh>
```

✅ La connexion s'établit bien via l'IP fixe !

---

## 🌐 Étape 6 : Connexion depuis un autre conteneur

Testons la connexion depuis un autre conteneur sur le même réseau.

### Créer un conteneur de test

```bash
docker run -it --rm \
  --network cassandra_net \
  cassandra:4.1 \
  cqlsh 172.20.0.10
```

**Explication :**
- `--network cassandra_net` : Rejoint le même réseau
- `cassandra:4.1` : Utilise la même image (pour avoir `cqlsh`)
- `cqlsh 172.20.0.10` : Se connecte à Cassandra via l'IP fixe

**Résultat attendu :**
```
Connected to DevCluster at 172.20.0.10:9042
cqlsh>
```

🎉 **Succès !** Un autre conteneur peut se connecter via l'IP fixe.

Tapez `EXIT;` pour quitter.

---

## 🎯 Cas d'usage pratiques

### 1. Application Node.js qui se connecte à Cassandra

**docker-compose.yml avec application :**

```yaml
version: '3.8'

services:
  cassandra:
    image: cassandra:4.1
    container_name: cassandra_ip_fixe
    # ... (configuration précédente)
    networks:
      cassandra_net:
        ipv4_address: 172.20.0.10

  app:
    image: node:18
    container_name: app_node
    command: node app.js
    volumes:
      - ./app:/usr/src/app
    working_dir: /usr/src/app
    networks:
      - cassandra_net
    depends_on:
      - cassandra

networks:
  cassandra_net:
    external: true
```

**Dans `app.js` :**
```javascript
const cassandra = require('cassandra-driver');

const client = new cassandra.Client({
  contactPoints: ['172.20.0.10'],  // IP fixe !
  localDataCenter: 'datacenter1',
  keyspace: 'demo'
});

client.connect()
  .then(() => console.log('Connecté à Cassandra'))
  .catch(err => console.error('Erreur:', err));
```

### 2. Configuration dans un fichier `.env`

**Fichier `.env` :**
```bash
CASSANDRA_HOST=172.20.0.10
CASSANDRA_PORT=9042
CASSANDRA_KEYSPACE=demo
```

**Dans `docker-compose.yml` :**
```yaml
services:
  app:
    environment:
      - CASSANDRA_HOST=${CASSANDRA_HOST}
      - CASSANDRA_PORT=${CASSANDRA_PORT}
```

---

## 🛠️ Étape 7 : Commandes utiles

### Vérifier la connectivité réseau

Depuis votre machine hôte (si le port 9042 est exposé) :

```bash
# Avec telnet
telnet 172.20.0.10 9042

# Avec nc (netcat)
nc -zv 172.20.0.10 9042
```

**Note :** Cela ne fonctionnera que si vous êtes dans le même réseau Docker.

### Lister les conteneurs sur le réseau

```bash
docker network inspect cassandra_net --format '{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{println}}{{end}}'
```

**Résultat :**
```
cassandra_ip_fixe: 172.20.0.10/16
```

### Ping depuis un autre conteneur

```bash
docker run --rm --network cassandra_net alpine ping -c 3 172.20.0.10
```

---

## 🚨 Troubleshooting

### Problème : Cassandra ne démarre pas

**Erreur possible :**
```
CASSANDRA_BROADCAST_ADDRESS is set to 172.20.0.10 but doesn't match any interface
```

**Solution :** Vérifiez que le réseau `cassandra_net` existe et que l'IP est bien dans le sous-réseau.

```bash
docker network ls
docker network inspect cassandra_net
```

### Problème : Conflit d'adresse IP

**Erreur :**
```
Error response from daemon: Address already in use
```

**Solution :** Un autre conteneur utilise déjà cette IP. Listez les conteneurs :

```bash
docker network inspect cassandra_net
```

Changez l'IP dans le `docker-compose.yml` (ex: `172.20.0.11`).

### Problème : "Connection refused" depuis un autre conteneur

**Solutions :**
1. Vérifiez que le conteneur client est sur le même réseau :
   ```bash
   docker inspect <nom_conteneur> | grep cassandra_net
   ```

2. Vérifiez que Cassandra est bien démarré :
   ```bash
   docker-compose ps
   ```

3. Attendez que le healthcheck soit "healthy".

---

## 🧹 Étape 8 : Nettoyage complet

### Arrêter le conteneur

```bash
docker-compose stop
```

### Supprimer le conteneur (données conservées)

```bash
docker-compose down
```

### Suppression complète (conteneur + données)

```bash
# Supprimer le conteneur et le volume
docker-compose down -v

# Supprimer le réseau
docker network rm cassandra_net

# Vérification
docker network ls
docker volume ls
```

---

## 💡 Conseils

### 1. Choisir une plage d'IP cohérente

Utilisez des plages privées standard :
- `172.16.0.0/12` → De `172.16.0.0` à `172.31.255.255`
- `192.168.0.0/16` → De `192.168.0.0` à `192.168.255.255`

### 2. Documenter les IPs

Créez un fichier `NETWORK.md` dans votre projet :
```markdown
# Configuration réseau

- Réseau : cassandra_net (172.20.0.0/16)
- Cassandra : 172.20.0.10
- Application : 172.20.0.20
```

### 3. Utiliser des variables d'environnement

Au lieu de hardcoder les IPs, utilisez des fichiers `.env` :

```bash
# .env
CASSANDRA_IP=172.20.0.10
```

```yaml
# docker-compose.yml
environment:
  - CASSANDRA_SEEDS=${CASSANDRA_IP}
```

### 4. IP fixe vs nom de conteneur

| Méthode | Avantages | Inconvénients |
|---------|-----------|---------------|
| **IP fixe** | Prévisible, facile à debugger | Configuration manuelle |
| **Nom de conteneur** | Automatique, flexible | IP change à chaque redémarrage |

**Recommandation :** Utilisez les noms de conteneurs pour le développement, les IPs fixes pour des cas spécifiques.

---

## 📚 Prochaines étapes

Maintenant que vous maîtrisez les IPs fixes, explorez :

- **[7.3 Cluster simple](03-cluster-simple.md)** : Créer un cluster à 3 nœuds avec IPs fixes
- **[7.4 Gestion des keyspaces](04-gestion-keyspaces.md)** : Stratégies de réplication avancées
- **[Annexe B - Gestion des réseaux](/annexes/B-gestion-reseaux.md)** : Approfondir Docker networking

---

## ❓ Questions fréquentes (FAQ)

**Q : Quelle plage d'IP utiliser ?**
R : Préférez `172.16.0.0/12` ou `192.168.0.0/16` (réseaux privés standards).

**Q : Puis-je changer l'IP après le premier lancement ?**
R : Oui, mais vous devez supprimer le conteneur (`docker-compose down`) et le recréer.

**Q : L'IP fixe fonctionne-t-elle depuis ma machine hôte ?**
R : Non, l'IP `172.20.0.10` n'est accessible que depuis d'autres conteneurs sur le même réseau Docker. Utilisez `localhost:9042` depuis votre machine.

**Q : Différence entre `CASSANDRA_LISTEN_ADDRESS` et `CASSANDRA_BROADCAST_ADDRESS` ?**
R :
- `LISTEN_ADDRESS` : Interface réseau d'écoute (interne au conteneur)
- `BROADCAST_ADDRESS` : Adresse annoncée aux clients (celle qu'ils utilisent pour se connecter)

**Q : Cassandra ne démarre pas avec l'IP fixe, que faire ?**
R : Vérifiez les logs avec `docker-compose logs cassandra` et assurez-vous que le réseau existe (`docker network ls`).

---

## 🔗 Ressources complémentaires

- [Documentation Docker - Réseaux](https://docs.docker.com/network/)
- [Cassandra - Configuration réseau](https://cassandra.apache.org/doc/latest/cassandra/configuration/cass_yaml_file.html#listen_address)
- [Annexe B - Gestion des réseaux Docker](/annexes/B-gestion-reseaux.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
