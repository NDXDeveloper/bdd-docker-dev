# 6.4 - Configuration de Redis avec IP fixe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 🌐 Pourquoi utiliser une IP fixe ?

Par défaut, Docker attribue une **adresse IP aléatoire** à chaque conteneur. Cette IP change à chaque redémarrage, ce qui peut poser problème dans certains cas.

**Problèmes avec une IP dynamique :**
- ❌ L'IP change à chaque `docker-compose down/up`
- ❌ Difficile de configurer des règles de pare-feu
- ❌ Compliqué de référencer le conteneur depuis d'autres services
- ❌ Configuration réseau imprévisible

**Avantages d'une IP fixe :**
- ✅ Adresse prévisible et stable
- ✅ Facilite la configuration réseau
- ✅ Simplifie le débogage
- ✅ Meilleur contrôle pour les environnements multi-conteneurs
- ✅ Permet des règles de pare-feu précises

---

## 🎯 Objectif de cette fiche

Vous allez apprendre à :
1. ✅ Créer un réseau Docker personnalisé
2. ✅ Assigner une adresse IP fixe à Redis
3. ✅ Configurer plusieurs conteneurs Redis avec des IPs différentes
4. ✅ Connecter d'autres conteneurs à Redis via son IP fixe
5. ✅ Gérer et supprimer proprement les réseaux Docker

---

## 📚 Concepts de base : Réseaux Docker

### **Types de réseaux Docker**

| Type | Description | Cas d'usage |
|------|-------------|-------------|
| **bridge** | Réseau isolé par défaut | Développement local |
| **host** | Partage le réseau de l'hôte | Performances maximales |
| **overlay** | Réseau multi-hôtes | Docker Swarm, Kubernetes |
| **macvlan** | Assigne une MAC au conteneur | Intégration réseau physique |
| **none** | Aucun réseau | Isolation totale |

Pour assigner une IP fixe, nous utiliserons un réseau **bridge personnalisé**.

### **Plages d'adresses privées**

Utilisez toujours des plages d'adresses **privées** :

| Plage | CIDR | Nombre d'adresses | Usage typique |
|-------|------|-------------------|---------------|
| 10.0.0.0 - 10.255.255.255 | /8 | ~16 millions | Réseaux d'entreprise |
| 172.16.0.0 - 172.31.255.255 | /12 | ~1 million | Réseaux moyens |
| 192.168.0.0 - 192.168.255.255 | /16 | ~65 000 | Réseaux domestiques |

**Exemples pour Docker :**
- `172.20.0.0/16` → IPs de `172.20.0.1` à `172.20.255.254`
- `192.168.100.0/24` → IPs de `192.168.100.1` à `192.168.100.254`
- `10.5.0.0/16` → IPs de `10.5.0.1` à `10.5.255.254`

---

## 🚀 Méthode 1 : IP fixe simple

### **Étape 1 : Créer le réseau Docker**

Avant de lancer le conteneur, créez un réseau personnalisé :

```bash
docker network create \
  --driver=bridge \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  redis_network
```

**Explication des paramètres :**
- `--driver=bridge` : Type de réseau (bridge = isolé)
- `--subnet=172.20.0.0/16` : Plage d'adresses disponibles
- `--gateway=172.20.0.1` : Passerelle du réseau (optionnel)
- `redis_network` : Nom du réseau

### **Étape 2 : Créer le dossier du projet**

```bash
mkdir redis-ip-fixe
cd redis-ip-fixe
```

### **Étape 3 : Créer le fichier `docker-compose.yml`**

```yaml
version: '3.8'

services:
  redis:
    # Image officielle Redis
    image: redis:7-alpine

    # Nom du conteneur
    container_name: redis_ip_fixe

    # Redémarrage automatique
    restart: unless-stopped

    # Commande avec mot de passe (optionnel)
    command: redis-server --requirepass MotDePasseSecurise123!

    # Exposition du port
    ports:
      - "6379:6379"

    # Persistance des données
    volumes:
      - redis_data:/data

    # Configuration réseau
    networks:
      redis_network:
        # ⭐ IP fixe assignée au conteneur
        ipv4_address: 172.20.0.10

# Déclaration des volumes
volumes:
  redis_data:

# Déclaration du réseau (externe = créé manuellement)
networks:
  redis_network:
    external: true
```

**Points clés :**
- `ipv4_address: 172.20.0.10` : Notre IP fixe choisie
- `external: true` : Le réseau existe déjà (créé à l'étape 1)

### **Étape 4 : Démarrer Redis**

```bash
docker-compose up -d
```

### **Étape 5 : Vérifier l'IP assignée**

```bash
# Méthode 1 : via docker inspect
docker inspect redis_ip_fixe | grep IPAddress

# Méthode 2 : via docker network inspect
docker network inspect redis_network

# Méthode 3 : affichage formaté
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis_ip_fixe
```

**Résultat attendu :**
```
172.20.0.10
```

✅ Parfait ! Redis a bien l'IP fixe `172.20.0.10`.

---

## 🔌 Se connecter via l'IP fixe

### **Depuis un autre conteneur Docker**

Créez un conteneur Alpine pour tester :

```bash
# Lancer un conteneur temporaire sur le même réseau
docker run -it --rm \
  --network redis_network \
  alpine sh

# Installer redis-cli
apk add redis

# Se connecter à Redis via son IP fixe
redis-cli -h 172.20.0.10 -p 6379 -a MotDePasseSecurise123!

# Tester
PING
# Résultat : PONG
```

### **Depuis votre machine hôte**

Utilisez `localhost` ou `127.0.0.1` (le port est exposé) :

```bash
docker exec -it redis_ip_fixe redis-cli -a MotDePasseSecurise123!
```

Ou avec `redis-cli` installé localement :
```bash
redis-cli -h localhost -p 6379 -a MotDePasseSecurise123!
```

### **Depuis une application**

**Python :**
```python
import redis

# Connexion via IP fixe (depuis un autre conteneur)
r = redis.Redis(
    host='172.20.0.10',
    port=6379,
    password='MotDePasseSecurise123!',
    decode_responses=True
)

print(r.ping())  # True
```

**Node.js :**
```javascript
const redis = require('redis');

const client = redis.createClient({
  host: '172.20.0.10',
  port: 6379,
  password: 'MotDePasseSecurise123!'
});

client.on('connect', () => {
  console.log('Connecté à Redis via IP fixe !');
});
```

---

## 🚀 Méthode 2 : Réseau créé automatiquement par Docker Compose

Plus simple : laissez Docker Compose créer le réseau automatiquement.

### **Fichier `docker-compose.yml` complet**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis_auto_network
    restart: unless-stopped
    command: redis-server --requirepass SecurePass123!
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      redis_net:
        ipv4_address: 172.21.0.10

volumes:
  redis_data:

# Docker Compose crée automatiquement le réseau
networks:
  redis_net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.21.0.0/16
          gateway: 172.21.0.1
```

**Avantages de cette méthode :**
- ✅ Pas besoin de créer le réseau manuellement
- ✅ Configuration centralisée dans un seul fichier
- ✅ Nettoyage automatique avec `docker-compose down`

**Démarrage :**
```bash
docker-compose up -d
```

**Vérification :**
```bash
docker inspect redis_auto_network | grep IPAddress
# Résultat : 172.21.0.10
```

---

## 🔄 Configuration multi-conteneurs Redis

### **Cas d'usage : Plusieurs instances Redis**

Créez plusieurs Redis avec des IPs différentes (par exemple : dev, test, prod simulés).

**Fichier `docker-compose.yml` :**

```yaml
version: '3.8'

services:
  # Redis pour développement
  redis_dev:
    image: redis:7-alpine
    container_name: redis_dev
    restart: unless-stopped
    command: redis-server --requirepass DevPassword123!
    ports:
      - "6379:6379"
    volumes:
      - redis_dev_data:/data
    networks:
      redis_multi_net:
        ipv4_address: 172.22.0.10

  # Redis pour tests
  redis_test:
    image: redis:7-alpine
    container_name: redis_test
    restart: unless-stopped
    command: redis-server --requirepass TestPassword123!
    ports:
      - "6380:6379"
    volumes:
      - redis_test_data:/data
    networks:
      redis_multi_net:
        ipv4_address: 172.22.0.11

  # Redis pour staging
  redis_staging:
    image: redis:7-alpine
    container_name: redis_staging
    restart: unless-stopped
    command: redis-server --requirepass StagingPassword123!
    ports:
      - "6381:6379"
    volumes:
      - redis_staging_data:/data
    networks:
      redis_multi_net:
        ipv4_address: 172.22.0.12

volumes:
  redis_dev_data:
  redis_test_data:
  redis_staging_data:

networks:
  redis_multi_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.22.0.0/16
```

**Démarrage :**
```bash
docker-compose up -d
```

**Connexions :**

| Instance | IP fixe | Port hôte | Mot de passe |
|----------|---------|-----------|--------------|
| Dev | `172.22.0.10` | 6379 | DevPassword123! |
| Test | `172.22.0.11` | 6380 | TestPassword123! |
| Staging | `172.22.0.12` | 6381 | StagingPassword123! |

**Tests depuis l'hôte :**
```bash
# Redis Dev
redis-cli -h localhost -p 6379 -a DevPassword123!

# Redis Test
redis-cli -h localhost -p 6380 -a TestPassword123!

# Redis Staging
redis-cli -h localhost -p 6381 -a StagingPassword123!
```

**Tests depuis un conteneur sur le même réseau :**
```bash
docker run -it --rm --network redis-ip-fixe_redis_multi_net redis:7-alpine sh

# Se connecter aux différentes instances
redis-cli -h 172.22.0.10 -p 6379 -a DevPassword123!
redis-cli -h 172.22.0.11 -p 6379 -a TestPassword123!
redis-cli -h 172.22.0.12 -p 6379 -a StagingPassword123!
```

---

## 🐍 Application complète avec IP fixe

### **Stack : Application Python + Redis**

**Structure du projet :**
```
app-redis/
├── docker-compose.yml
├── app/
│   ├── Dockerfile
│   └── app.py
└── .env
```

**Fichier `docker-compose.yml` :**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: app_redis
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      app_network:
        ipv4_address: 172.23.0.10

  app:
    build: ./app
    container_name: app_python
    restart: unless-stopped
    depends_on:
      - redis
    environment:
      - REDIS_HOST=172.23.0.10
      - REDIS_PORT=6379
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    ports:
      - "5000:5000"
    networks:
      app_network:
        ipv4_address: 172.23.0.20

volumes:
  redis_data:

networks:
  app_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.23.0.0/16
```

**Fichier `.env` :**
```bash
REDIS_PASSWORD=SuperSecretPassword123!
```

**Fichier `app/Dockerfile` :**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install redis flask

COPY app.py .

CMD ["python", "app.py"]
```

**Fichier `app/app.py` :**
```python
from flask import Flask, jsonify
import redis
import os

app = Flask(__name__)

# Connexion à Redis via IP fixe
r = redis.Redis(
    host=os.getenv('REDIS_HOST', '172.23.0.10'),
    port=int(os.getenv('REDIS_PORT', 6379)),
    password=os.getenv('REDIS_PASSWORD'),
    decode_responses=True
)

@app.route('/')
def index():
    return jsonify({
        'status': 'ok',
        'redis_connected': r.ping()
    })

@app.route('/count')
def count():
    # Incrémenter un compteur
    visits = r.incr('visits')
    return jsonify({
        'visits': visits
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Démarrage :**
```bash
docker-compose up -d --build
```

**Tests :**
```bash
# Tester l'application
curl http://localhost:5000/
# {"redis_connected":true,"status":"ok"}

curl http://localhost:5000/count
# {"visits":1}

curl http://localhost:5000/count
# {"visits":2}
```

---

## 🛠️ Commandes utiles pour gérer les réseaux

### **Lister les réseaux**
```bash
docker network ls
```

### **Inspecter un réseau**
```bash
docker network inspect redis_network
```

### **Voir les conteneurs connectés à un réseau**
```bash
docker network inspect redis_network --format='{{range .Containers}}{{.Name}} - {{.IPv4Address}}{{println}}{{end}}'
```

### **Connecter un conteneur existant à un réseau**
```bash
docker network connect redis_network mon_conteneur
```

### **Déconnecter un conteneur d'un réseau**
```bash
docker network disconnect redis_network mon_conteneur
```

### **Supprimer un réseau**
```bash
# Arrêter tous les conteneurs utilisant le réseau d'abord
docker-compose down

# Puis supprimer le réseau
docker network rm redis_network
```

### **Supprimer tous les réseaux inutilisés**
```bash
docker network prune
```

---

## 🧹 Nettoyage complet

### **Supprimer tout (conteneurs + réseaux + volumes)**

```bash
# Arrêter et supprimer les conteneurs
docker-compose down

# Supprimer aussi les volumes (⚠️ perte de données)
docker-compose down -v

# Supprimer le réseau (si créé manuellement)
docker network rm redis_network
```

### **Vérifier le nettoyage**
```bash
# Vérifier les conteneurs
docker ps -a | grep redis

# Vérifier les réseaux
docker network ls | grep redis

# Vérifier les volumes
docker volume ls | grep redis
```

---

## 🔍 Dépannage

### **Problème : "Address already in use"**

**Erreur :**
```
Error response from daemon: user specified IP address is already in use
```

**Causes possibles :**
1. Un autre conteneur utilise déjà cette IP
2. Le réseau contient des "fantômes" (conteneurs supprimés mal)

**Solutions :**
```bash
# Voir les conteneurs utilisant le réseau
docker network inspect redis_network

# Nettoyer les réseaux inutilisés
docker network prune

# Recréer le réseau
docker network rm redis_network
docker network create --subnet=172.20.0.0/16 redis_network
```

### **Problème : "Network not found"**

**Erreur :**
```
ERROR: Network redis_network declared as external, but could not be found
```

**Solution :**
Créez le réseau avant de lancer Docker Compose :
```bash
docker network create --subnet=172.20.0.0/16 redis_network
docker-compose up -d
```

### **Problème : Conteneurs ne peuvent pas communiquer**

**Vérifications :**
```bash
# 1. Vérifier que les conteneurs sont sur le même réseau
docker network inspect redis_network

# 2. Tester la connectivité depuis un conteneur
docker exec -it redis_ip_fixe ping 172.20.0.11

# 3. Vérifier les règles de pare-feu (iptables)
sudo iptables -L -n
```

### **Problème : Conflit de sous-réseaux**

**Erreur :**
```
Pool overlaps with other one on this address space
```

**Solution :**
Choisissez une plage différente :
```bash
# Au lieu de 172.20.0.0/16
docker network create --subnet=172.25.0.0/16 redis_network
```

---

## 📊 Comparaison des méthodes

| Critère | Réseau externe | Réseau dans Compose |
|---------|---------------|---------------------|
| **Simplicité** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Flexibilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Partage entre projets** | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Nettoyage automatique** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Recommandé pour** | Multi-projets | Projet isolé |

---

## ❓ Questions fréquentes

### **Pourquoi utiliser 172.x au lieu de 192.168.x ?**

Les deux fonctionnent. Conventions :
- `192.168.x.x` : Souvent utilisé par les box internet (risque de conflit)
- `172.x.x.x` : Moins utilisé, donc plus sûr pour Docker
- `10.x.x.x` : Grande plage, idéal pour de gros réseaux

### **Puis-je utiliser n'importe quelle IP ?**

Non, l'IP doit être **dans la plage du subnet** défini.

Exemple avec `subnet: 172.20.0.0/16` :
- ✅ Valide : `172.20.0.10`, `172.20.100.50`, `172.20.255.254`
- ❌ Invalide : `172.21.0.10`, `192.168.1.10`, `172.20.0.1` (gateway)

### **L'IP fixe persiste-t-elle après un redémarrage ?**

Oui ! C'est tout l'intérêt. Même après :
- `docker-compose down` puis `up`
- Redémarrage de la machine
- Mise à jour de l'image

L'IP reste `172.20.0.10`.

### **Puis-je changer l'IP fixe ?**

Oui, modifiez le `docker-compose.yml` :
```yaml
ipv4_address: 172.20.0.20  # Nouvelle IP
```

Puis recréez le conteneur :
```bash
docker-compose down
docker-compose up -d
```

### **Comment voir toutes les IPs utilisées sur un réseau ?**

```bash
docker network inspect redis_network --format='{{range .Containers}}{{.IPv4Address}} - {{.Name}}{{println}}{{end}}'
```

### **Les performances sont-elles impactées ?**

Non, aucun impact sur les performances. L'IP fixe est juste une configuration réseau.

---

## 🎉 Récapitulatif

Vous savez maintenant :
- ✅ Créer un réseau Docker personnalisé avec subnet
- ✅ Assigner une IP fixe à un conteneur Redis
- ✅ Configurer plusieurs conteneurs avec des IPs différentes
- ✅ Connecter des applications à Redis via IP fixe
- ✅ Choisir entre réseau externe ou interne à Compose
- ✅ Déboguer les problèmes de réseau
- ✅ Nettoyer proprement réseaux et conteneurs

**Prochaines étapes :**
- 🖥️ [6.5 - Redis avec Redis Commander (GUI)](05-redis-commander.md)
- 🌐 [Annexe B - Gestion des réseaux Docker](../annexes/B-gestion-reseaux.md)
- 🔒 [Annexe D - Sécurité et bonnes pratiques](../annexes/D-securite-bonnes-pratiques.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
