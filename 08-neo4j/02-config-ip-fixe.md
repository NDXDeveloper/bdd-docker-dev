# 8.2 Configuration avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Dans la [configuration basique](01-config-basique-docker-compose.md), Neo4j obtient automatiquement une adresse IP dynamique de Docker. Cette IP peut changer à chaque redémarrage du conteneur, ce qui peut poser problème dans certains cas.

### 🤔 Pourquoi une IP fixe ?

| Cas d'usage | Problème avec IP dynamique | Solution avec IP fixe |
|-------------|---------------------------|---------------------|
| **Plusieurs conteneurs** | L'app ne trouve plus Neo4j après redémarrage | IP toujours identique |
| **Tests automatisés** | Scripts de test cassés à chaque redémarrage | Configuration stable |
| **Réseau personnalisé** | Communication inter-conteneurs aléatoire | Adressage prévisible |
| **Monitoring** | Outils de supervision perdent la trace | Cible fixe |

### 🎯 Ce que vous allez apprendre

Dans ce tutoriel, vous allez :
1. Créer un **réseau Docker personnalisé** avec une plage d'adresses définie
2. Assigner une **adresse IP statique** à votre conteneur Neo4j
3. Configurer Neo4j pour accepter les connexions sur cette IP
4. Tester la connexion depuis un autre conteneur

---

## 🚀 Étape 1 : Créer le réseau Docker dédié

Avant de lancer Neo4j, nous devons créer un réseau Docker personnalisé qui nous permettra de contrôler les adresses IP.

### Créer le réseau

Ouvrez un terminal et exécutez :

```bash
docker network create --subnet=172.19.0.0/16 neo4j_network
```

**Explication :**
- `docker network create` : Crée un nouveau réseau Docker
- `--subnet=172.19.0.0/16` : Définit la plage d'adresses IP disponibles
  - `172.19.0.0/16` signifie : de `172.19.0.1` à `172.19.255.254`
  - Vous pouvez utiliser une autre plage (ex: `172.20.0.0/16`, `10.5.0.0/16`)
- `neo4j_network` : Nom du réseau (choisissez ce que vous voulez)

**Résultat attendu :**
```
4f8a9b2c1d3e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0
```
(Un identifiant unique pour le réseau)

### Vérifier la création du réseau

```bash
docker network ls
```

**Résultat :**
```
NETWORK ID     NAME            DRIVER    SCOPE
4f8a9b2c1d3e   neo4j_network   bridge    local
abc123def456   bridge          bridge    local
...
```

### Inspecter le réseau (optionnel)

```bash
docker network inspect neo4j_network
```

Vous verrez les détails du réseau, notamment le subnet `172.19.0.0/16`.

---

## 📁 Étape 2 : Créer le projet

Créez un nouveau dossier pour ce projet :

```bash
mkdir neo4j-ip-fixe
cd neo4j-ip-fixe
```

---

## 📝 Étape 3 : Créer le fichier `docker-compose.yml`

Créez un fichier `docker-compose.yml` avec le contenu suivant :

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.13
    container_name: neo4j_static_ip
    restart: unless-stopped

    ports:
      - "7474:7474"  # Neo4j Browser (HTTP)
      - "7687:7687"  # Bolt protocol

    environment:
      # Désactive l'authentification (pour le dev)
      NEO4J_AUTH: none
      NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

      # Configuration réseau : écouter sur toutes les interfaces
      # Par défaut, Neo4j n'écoute que sur localhost
      NEO4J_server_default__listen__address: 0.0.0.0

      # Adresse publique pour les connexions Bolt
      NEO4J_server_bolt_advertised__address: 172.19.0.10:7687

      # Adresse publique pour HTTP
      NEO4J_server_http_advertised__address: 172.19.0.10:7474

    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
      - neo4j_import:/var/lib/neo4j/import
      - neo4j_plugins:/plugins

    # Configuration du réseau et de l'IP fixe
    networks:
      neo4j_network:
        ipv4_address: 172.19.0.10

volumes:
  neo4j_data:
  neo4j_logs:
  neo4j_import:
  neo4j_plugins:

# Déclaration du réseau externe (créé à l'étape 1)
networks:
  neo4j_network:
    external: true
```

---

## 🔍 Explication détaillée de la configuration

### 1️⃣ Configuration réseau de Neo4j

```yaml
environment:
  NEO4J_server_default__listen__address: 0.0.0.0
```

**Pourquoi `0.0.0.0` ?**
- Par défaut, Neo4j écoute uniquement sur `127.0.0.1` (localhost)
- `0.0.0.0` signifie : "écouter sur **toutes les interfaces réseau**"
- Sans cela, les connexions depuis l'extérieur du conteneur seraient refusées

### 2️⃣ Adresses annoncées (advertised addresses)

```yaml
NEO4J_server_bolt_advertised__address: 172.19.0.10:7687
NEO4J_server_http_advertised__address: 172.19.0.10:7474
```

**À quoi ça sert ?**
- Neo4j doit dire aux clients comment le contacter
- Ces paramètres définissent l'adresse IP que Neo4j communique aux clients
- **Important** : Utilisez l'IP fixe que vous allez assigner (ici `172.19.0.10`)

### 3️⃣ Assignation de l'IP fixe

```yaml
networks:
  neo4j_network:
    ipv4_address: 172.19.0.10
```

**Explication :**
- `neo4j_network` : Le réseau créé à l'étape 1
- `ipv4_address: 172.19.0.10` : L'IP statique souhaitée
- Cette IP doit être dans la plage du subnet (`172.19.0.0/16`)

⚠️ **Attention :** Ne choisissez pas `172.19.0.1` (réservée pour la passerelle Docker).

### 4️⃣ Déclaration du réseau externe

```yaml
networks:
  neo4j_network:
    external: true
```

**Pourquoi `external: true` ?**
- Indique à Docker Compose que le réseau existe déjà (créé manuellement à l'étape 1)
- Sans cela, Docker Compose essaierait de créer un nouveau réseau et échouerait

---

## ▶️ Étape 4 : Démarrer Neo4j

Dans le dossier contenant `docker-compose.yml`, lancez :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating volume "neo4j-ip-fixe_neo4j_data" with default driver
Creating volume "neo4j-ip-fixe_neo4j_logs" with default driver
Creating volume "neo4j-ip-fixe_neo4j_import" with default driver
Creating volume "neo4j-ip-fixe_neo4j_plugins" with default driver
Creating neo4j_static_ip ... done
```

### Vérifier le conteneur

```bash
docker-compose ps
```

**Résultat :**
```
      Name                    Command               State           Ports
----------------------------------------------------------------------------------
neo4j_static_ip   tini -g -- /startup/docker ...   Up      0.0.0.0:7474->7474/tcp,
                                                            0.0.0.0:7687->7687/tcp
```

---

## ✅ Étape 5 : Vérifier l'adresse IP assignée

### Méthode 1 : Avec `docker inspect`

```bash
docker inspect neo4j_static_ip | grep "IPAddress"
```

**Résultat attendu :**
```json
"IPAddress": "",
"SecondaryIPAddresses": null,
"IPAddress": "172.19.0.10",
```

Vous devriez voir `172.19.0.10` !

### Méthode 2 : Avec `docker network inspect`

```bash
docker network inspect neo4j_network
```

Cherchez la section `"Containers"`, vous verrez :

```json
"Containers": {
    "abc123...": {
        "Name": "neo4j_static_ip",
        "IPv4Address": "172.19.0.10/16",
        ...
    }
}
```

---

## 🌐 Étape 6 : Accéder à Neo4j

### Via le navigateur (depuis votre machine hôte)

```
http://localhost:7474
```

Connexion :
- **Connect URL** : `neo4j://localhost:7687`
- **Authentication** : "No authentication" (si `NEO4J_AUTH: none`)

### Via l'IP fixe (depuis un autre conteneur du même réseau)

Si vous avez un autre conteneur sur le réseau `neo4j_network`, utilisez :

```
neo4j://172.19.0.10:7687
```

---

## 🧪 Étape 7 : Tester la connexion depuis un autre conteneur

Créons un conteneur temporaire pour tester la connexion à Neo4j via son IP fixe.

### Lancer un conteneur Alpine (Linux léger)

```bash
docker run -it --rm --network neo4j_network alpine sh
```

**Explication :**
- `--network neo4j_network` : Connecte ce conteneur au même réseau que Neo4j
- `alpine` : Image Linux minimaliste
- `sh` : Ouvre un shell interactif

### Depuis le conteneur Alpine, tester la connexion

```bash
# Installer telnet (outil de test réseau)
apk add --no-cache busybox-extras

# Tester le port HTTP (7474)
telnet 172.19.0.10 7474

# Si la connexion réussit, vous verrez :
# Connected to 172.19.0.10
```

**Tapez Ctrl+] puis `quit` pour sortir de telnet.**

```bash
# Tester le port Bolt (7687)
telnet 172.19.0.10 7687
```

Si les deux connexions réussissent, votre IP fixe fonctionne parfaitement ! 🎉

**Quitter le conteneur Alpine :**
```bash
exit
```

---

## 📊 Exemple d'utilisation : Connecter une application Python

Voici comment une application Python sur le même réseau Docker se connecterait à Neo4j.

### Créer un fichier `docker-compose-app.yml`

```yaml
version: '3.8'

services:
  python_app:
    image: python:3.11-slim
    container_name: neo4j_client_app

    # Installer le driver Neo4j et maintenir le conteneur actif
    command: >
      sh -c "pip install neo4j &&
             python -c 'from neo4j import GraphDatabase;
             driver = GraphDatabase.driver(\"neo4j://172.19.0.10:7687\");
             with driver.session() as session:
                 result = session.run(\"RETURN 1 AS num\");
                 print(\"Connexion réussie:\", result.single()[\"num\"]);
             driver.close()' &&
             tail -f /dev/null"

    networks:
      - neo4j_network

networks:
  neo4j_network:
    external: true
```

### Lancer l'application

```bash
docker-compose -f docker-compose-app.yml up -d
```

### Voir les logs

```bash
docker-compose -f docker-compose-app.yml logs python_app
```

**Résultat attendu :**
```
Connexion réussie: 1
```

---

## 🔧 Configuration avancée (optionnelle)

### Utiliser plusieurs conteneurs avec IPs fixes

Vous pouvez créer plusieurs conteneurs Neo4j sur le même réseau :

```yaml
services:
  neo4j_prod:
    image: neo4j:5.13
    container_name: neo4j_prod
    # ... (même config)
    networks:
      neo4j_network:
        ipv4_address: 172.19.0.10

  neo4j_test:
    image: neo4j:5.13
    container_name: neo4j_test
    ports:
      - "7475:7474"  # Ports différents !
      - "7688:7687"
    # ... (même config)
    networks:
      neo4j_network:
        ipv4_address: 172.19.0.11  # IP différente
```

### Activer l'authentification avec IP fixe

Modifiez la section `environment` :

```yaml
environment:
  # Utilisateur : neo4j, Mot de passe : VotreMotDePasse123
  NEO4J_AUTH: neo4j/VotreMotDePasse123
  NEO4J_ACCEPT_LICENSE_AGREEMENT: 'yes'

  # Configurations réseau (inchangées)
  NEO4J_server_default__listen__address: 0.0.0.0
  NEO4J_server_bolt_advertised__address: 172.19.0.10:7687
  NEO4J_server_http_advertised__address: 172.19.0.10:7474
```

Pour se connecter depuis une app :

```python
from neo4j import GraphDatabase

driver = GraphDatabase.driver(
    "neo4j://172.19.0.10:7687",
    auth=("neo4j", "VotreMotDePasse123")
)
```

---

## 🗑️ Nettoyage complet

Pour supprimer tout l'environnement :

```bash
# 1. Arrêter et supprimer le conteneur Neo4j
docker-compose down

# 2. Supprimer les volumes (SUPPRIME LES DONNÉES)
docker-compose down -v

# 3. Supprimer le réseau personnalisé
docker network rm neo4j_network
```

**Vérifier la suppression :**
```bash
docker network ls
# neo4j_network ne devrait plus apparaître
```

---

## 🐛 Dépannage

### Erreur : "network neo4j_network not found"

**Cause :** Le réseau n'a pas été créé à l'étape 1.

**Solution :**
```bash
docker network create --subnet=172.19.0.0/16 neo4j_network
```

### Erreur : "address already in use"

**Cause :** Un autre conteneur utilise déjà l'IP `172.19.0.10`.

**Solutions :**
1. Lister les conteneurs sur le réseau :
   ```bash
   docker network inspect neo4j_network
   ```
2. Choisir une IP différente (ex: `172.19.0.20`)
3. Ou arrêter l'autre conteneur

### Neo4j ne répond pas sur l'IP fixe

**Vérifications :**

1. **Le conteneur tourne-t-il ?**
   ```bash
   docker-compose ps
   ```

2. **L'IP est-elle correctement assignée ?**
   ```bash
   docker inspect neo4j_static_ip | grep "IPAddress"
   ```

3. **Neo4j écoute-t-il sur 0.0.0.0 ?**
   ```bash
   docker-compose logs neo4j | grep "listen"
   ```
   Vous devriez voir des lignes contenant `0.0.0.0`.

4. **Tester depuis l'intérieur du conteneur :**
   ```bash
   docker exec -it neo4j_static_ip cypher-shell -a neo4j://172.19.0.10:7687
   ```

### Impossible de se connecter depuis un autre conteneur

**Cause :** L'autre conteneur n'est pas sur le réseau `neo4j_network`.

**Solution :** Ajoutez `--network neo4j_network` ou configurez le réseau dans son `docker-compose.yml`.

---

## 📚 Comparaison des méthodes de connexion

| Méthode | Depuis où | URL | Quand l'utiliser |
|---------|-----------|-----|------------------|
| **localhost** | Machine hôte | `neo4j://localhost:7687` | Développement local, Neo4j Browser |
| **IP fixe (réseau Docker)** | Autres conteneurs | `neo4j://172.19.0.10:7687` | Communication inter-conteneurs |
| **Nom de service** | Même docker-compose | `neo4j://neo4j:7687` | Stack multi-services (voir cas pratiques) |

---

## 💡 Quand utiliser une IP fixe ?

### ✅ Utilisez une IP fixe quand :

- Vous avez **plusieurs conteneurs** qui doivent communiquer
- Vous faites des **tests automatisés** avec des configurations réseau
- Vous devez **simuler un environnement de production** avec des IPs fixes
- Vous utilisez des **outils de monitoring** qui nécessitent une cible stable

### ❌ Pas besoin d'IP fixe quand :

- Vous utilisez Neo4j **seul** pour le développement
- Vous utilisez le **nom de service** dans `docker-compose` (ex: `services: neo4j:`)
- Vous accédez seulement via **localhost** depuis votre machine hôte

**Astuce :** Pour la plupart des projets, utiliser les **noms de service Docker Compose** est plus simple que les IPs fixes !

---

## ✅ Récapitulatif

Vous avez appris à :

✅ Créer un réseau Docker personnalisé avec un subnet
✅ Assigner une adresse IP statique à un conteneur Neo4j
✅ Configurer Neo4j pour accepter les connexions sur cette IP
✅ Vérifier l'adresse IP assignée
✅ Tester la connexion depuis un autre conteneur
✅ Connecter une application à Neo4j via l'IP fixe
✅ Gérer les problèmes courants

**Prochaines étapes :**
- [Configuration avancée avec plugins](03-config-avec-plugins.md)
- [Import de données CSV](04-import-donnees-csv.md)
- [Cas pratique : Stack complète](../../cas-pratiques/)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
