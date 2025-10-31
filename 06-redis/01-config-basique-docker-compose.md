# 6.1 - Configuration basique de Redis avec docker-compose

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que Redis ?

**Redis** (Remote Dictionary Server) est une base de données **en mémoire** ultra-rapide qui fonctionne avec un système de **clé-valeur**. Elle est principalement utilisée comme :

- 🚀 **Cache** pour accélérer les applications web
- 📊 **Stockage de sessions** utilisateur
- 📨 **File d'attente** de messages
- ⚡ **Base de données temps réel** (leaderboards, compteurs)

**Points clés de Redis :**
- ⚡ **Extrêmement rapide** : Tout est en RAM
- 🔄 **Persistance optionnelle** : Peut sauvegarder sur disque
- 🛠️ **Structures de données riches** : Strings, Lists, Sets, Hashes, etc.
- 🌐 **Simple à utiliser** : Commandes intuitives

---

## 🎯 Objectif de cette fiche

Vous allez apprendre à :
1. ✅ Démarrer Redis en **moins de 2 minutes**
2. ✅ Utiliser Docker Compose pour la configuration
3. ✅ Vous connecter à Redis depuis votre machine
4. ✅ Tester des commandes Redis de base
5. ✅ Gérer le conteneur (arrêt, redémarrage, suppression)

**Aucune installation de Redis nécessaire sur votre machine !** 🎉

---

## 🚀 Étape 1 : Créer le fichier `docker-compose.yml`

Créez un nouveau dossier pour votre projet Redis :

```bash
mkdir redis-dev
cd redis-dev
```

Ensuite, créez un fichier `docker-compose.yml` avec ce contenu :

```yaml
version: '3.8'

services:
  redis:
    # Image officielle Redis (dernière version stable)
    image: redis:7-alpine

    # Nom du conteneur (pour faciliter la gestion)
    container_name: redis_dev

    # Redémarre automatiquement en cas d'erreur
    restart: unless-stopped

    # Expose le port Redis
    ports:
      # Port hôte : Port conteneur
      # Redis écoute par défaut sur le port 6379
      - "6379:6379"

    # Volume pour la persistance des données (optionnel)
    volumes:
      - redis_data:/data

# Déclaration du volume pour stocker les données
volumes:
  redis_data:
```

---

## 📝 Explication de la configuration

Décortiquons chaque partie du fichier :

### **Image**
```yaml
image: redis:7-alpine
```
- `redis:7` : Version 7 de Redis (dernière version stable)
- `alpine` : Image légère basée sur Alpine Linux (plus petite, plus rapide)
- Autres options possibles : `redis:7`, `redis:latest`

### **Container Name**
```yaml
container_name: redis_dev
```
- Nom personnalisé du conteneur
- Facilite les commandes : `docker logs redis_dev`, `docker stop redis_dev`, etc.

### **Restart Policy**
```yaml
restart: unless-stopped
```
- Le conteneur redémarre automatiquement en cas de crash
- Ne redémarre pas si vous l'arrêtez manuellement

### **Ports**
```yaml
ports:
  - "6379:6379"
```
- **6379** : Port par défaut de Redis
- Format : `port_hôte:port_conteneur`
- Permet de se connecter depuis votre machine sur `localhost:6379`

### **Volumes**
```yaml
volumes:
  - redis_data:/data
```
- Crée un volume Docker nommé `redis_data`
- `/data` : Dossier où Redis sauvegarde ses données (snapshots RDB)
- **Important** : Sans volume, les données sont perdues à l'arrêt du conteneur

---

## ▶️ Étape 2 : Démarrer Redis

Dans le dossier contenant votre `docker-compose.yml`, lancez :

```bash
docker-compose up -d
```

**Détail de la commande :**
- `docker-compose up` : Lance les services définis dans le fichier
- `-d` : Mode "detached" (arrière-plan)

**Sortie attendue :**
```
Creating network "redis-dev_default" with the default driver
Creating volume "redis-dev_redis_data" with default driver
Creating redis_dev ... done
```

✅ **Redis est maintenant en cours d'exécution !**

---

## 🔍 Étape 3 : Vérifier que Redis fonctionne

### **1. Vérifier l'état du conteneur**

```bash
docker-compose ps
```

**Résultat attendu :**
```
   Name                 Command               State           Ports
----------------------------------------------------------------------------
redis_dev   docker-entrypoint.sh redis...   Up      0.0.0.0:6379->6379/tcp
```

Si `State` affiche `Up`, tout va bien ! 🎉

### **2. Voir les logs**

```bash
docker-compose logs
```

Ou en temps réel :
```bash
docker-compose logs -f
```

(Appuyez sur `Ctrl+C` pour quitter)

---

## 🔌 Étape 4 : Se connecter à Redis

### **Méthode 1 : Depuis le conteneur (CLI Redis intégré)**

La méthode la plus simple pour tester :

```bash
docker exec -it redis_dev redis-cli
```

**Explication :**
- `docker exec` : Exécute une commande dans un conteneur en cours
- `-it` : Mode interactif avec terminal
- `redis_dev` : Nom de notre conteneur
- `redis-cli` : Client en ligne de commande de Redis

Vous devriez voir :
```
127.0.0.1:6379>
```

🎉 **Vous êtes connecté à Redis !**

### **Méthode 2 : Depuis votre machine (avec redis-cli installé)**

Si vous avez `redis-cli` installé sur votre système :

```bash
redis-cli -h localhost -p 6379
```

---

## 🧪 Étape 5 : Tester Redis avec quelques commandes

Une fois connecté via `redis-cli`, essayez ces commandes :

### **1. Vérifier la connexion**
```redis
PING
```
**Résultat attendu :** `PONG`

### **2. Stocker une valeur**
```redis
SET nom "Nicolas"
```
**Résultat :** `OK`

### **3. Récupérer une valeur**
```redis
GET nom
```
**Résultat :** `"Nicolas"`

### **4. Stocker un nombre**
```redis
SET compteur 0
```

### **5. Incrémenter un compteur**
```redis
INCR compteur
```
**Résultat :** `(integer) 1`

```redis
INCR compteur
```
**Résultat :** `(integer) 2`

### **6. Lister toutes les clés**
```redis
KEYS *
```
**Résultat :** `1) "nom" 2) "compteur"`

### **7. Supprimer une clé**
```redis
DEL nom
```
**Résultat :** `(integer) 1`

### **8. Vérifier que la clé est supprimée**
```redis
GET nom
```
**Résultat :** `(nil)`

### **9. Quitter redis-cli**
```redis
EXIT
```

Ou appuyez sur `Ctrl+D`

---

## 📊 Étape 6 : Se connecter depuis une application

### **Exemple avec Python**

Si vous avez Python installé, installez d'abord la bibliothèque Redis :

```bash
pip install redis
```

Créez un fichier `test_redis.py` :

```python
import redis

# Connexion à Redis
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Test de connexion
print("Connexion réussie !" if r.ping() else "Échec de connexion")

# Écriture
r.set('langage', 'Python')

# Lecture
valeur = r.get('langage')
print(f"Valeur stockée : {valeur}")

# Incrément
r.set('visiteurs', 0)
r.incr('visiteurs')
r.incr('visiteurs')
print(f"Nombre de visiteurs : {r.get('visiteurs')}")
```

Exécutez :
```bash
python test_redis.py
```

**Résultat attendu :**
```
Connexion réussie !
Valeur stockée : Python
Nombre de visiteurs : 2
```

### **Exemple avec Node.js**

Installez le package Redis :
```bash
npm install redis
```

Créez un fichier `test_redis.js` :

```javascript
const redis = require('redis');

// Connexion à Redis
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

client.on('connect', () => {
  console.log('Connexion réussie !');
});

// Écriture
client.set('framework', 'Express', redis.print);

// Lecture
client.get('framework', (err, reply) => {
  console.log('Valeur stockée :', reply);
  client.quit();
});
```

Exécutez :
```bash
node test_redis.js
```

---

## 🛠️ Gestion du conteneur

### **Arrêter Redis (sans supprimer les données)**
```bash
docker-compose stop
```

### **Redémarrer Redis**
```bash
docker-compose start
```

### **Voir les logs en temps réel**
```bash
docker-compose logs -f
```

### **Redémarrer complètement (down puis up)**
```bash
docker-compose restart
```

### **Obtenir des informations sur le serveur Redis**
```bash
docker exec redis_dev redis-cli INFO server
```

---

## 🧹 Suppression complète

### **Arrêter et supprimer le conteneur**
```bash
docker-compose down
```

Cette commande :
- ✅ Arrête le conteneur
- ✅ Supprime le conteneur
- ❌ **Ne supprime PAS** les données (volume `redis_data`)

### **Supprimer aussi les données**
```bash
docker-compose down -v
```

⚠️ **ATTENTION** : Cette commande supprime **définitivement** toutes les données stockées dans Redis !

L'option `-v` (pour `--volumes`) supprime également les volumes Docker.

---

## 🎯 Cas d'usage typiques

### **Cache d'application web**
```redis
# Stocker un résultat de requête pendant 1 heure (3600 secondes)
SET "user:1234:profile" '{"nom":"Dupont","email":"dupont@example.com"}' EX 3600
```

### **Compteur de vues de page**
```redis
INCR "page:accueil:vues"
```

### **Session utilisateur**
```redis
# Session valide 24h (86400 secondes)
SET "session:abc123" '{"user_id":42,"role":"admin"}' EX 86400
```

### **File d'attente de tâches**
```redis
# Ajouter une tâche
LPUSH "file:emails" '{"to":"user@example.com","subject":"Bienvenue"}'

# Récupérer et traiter une tâche
RPOP "file:emails"
```

---

## 💡 Bonnes pratiques

### **1. Utilisez des noms de clés structurés**

❌ **Mauvais :**
```redis
SET user "données"
SET produit "données"
```

✅ **Bon :**
```redis
SET "user:1234:nom" "Dupont"
SET "produit:5678:prix" "29.99"
```

### **2. Définissez des expirations (TTL)**

Pour le cache et les données temporaires :
```redis
# Expire après 1 heure
SET "cache:article:42" "contenu" EX 3600

# Expire après 5 minutes
SET "token:xyz" "valeur" EX 300
```

### **3. Surveillez la mémoire**

Redis stocke tout en RAM. Vérifiez l'utilisation :
```bash
docker exec redis_dev redis-cli INFO memory
```

### **4. Utilisez des volumes pour la persistance**

Toujours inclure un volume dans votre `docker-compose.yml` pour ne pas perdre vos données :
```yaml
volumes:
  - redis_data:/data
```

---

## ❓ Questions fréquentes

### **Redis stocke-t-il vraiment tout en mémoire ?**
Oui ! C'est pour cela qu'il est si rapide. Par défaut, Redis peut sauvegarder périodiquement sur disque (snapshots RDB), mais le fonctionnement principal est en RAM.

### **Que se passe-t-il si je redémarre le conteneur ?**
Grâce au volume Docker (`redis_data`), vos données sont conservées. Redis charge automatiquement la dernière sauvegarde au démarrage.

### **Redis est-il sécurisé par défaut ?**
Non ! Dans cette configuration basique, Redis n'a **pas de mot de passe**. N'exposez jamais ce conteneur sur Internet. Pour un usage en production, consultez la fiche **6.2 - Configuration avec mot de passe**.

### **Puis-je utiliser cette config en production ?**
Non. Cette configuration est pour le **développement uniquement**. Pour la production :
- Activez un mot de passe (AUTH)
- Configurez la persistance (AOF)
- Limitez les commandes dangereuses
- Utilisez un réseau isolé

### **Comment voir toutes les données stockées ?**
```bash
docker exec redis_dev redis-cli KEYS "*"
```

⚠️ **Attention** : N'utilisez jamais `KEYS *` en production sur une base volumineuse (très lent).

### **Redis peut-il perdre des données ?**
Oui, si :
- Le serveur plante avant une sauvegarde RDB
- Vous supprimez le volume Docker
- La RAM est saturée et Redis évince des clés

---

## 📚 Ressources complémentaires

- 📖 [Documentation officielle Redis](https://redis.io/docs/)
- 🐳 [Image Docker officielle Redis](https://hub.docker.com/_/redis)
- 🎓 [Try Redis - Tutoriel interactif](https://try.redis.io/)
- 📝 [Liste complète des commandes Redis](https://redis.io/commands/)

---

## 🎉 Récapitulatif

Vous savez maintenant :
- ✅ Démarrer Redis avec Docker Compose en quelques secondes
- ✅ Vous connecter via `redis-cli`
- ✅ Exécuter des commandes Redis de base (SET, GET, INCR, etc.)
- ✅ Connecter une application (Python, Node.js)
- ✅ Gérer le cycle de vie du conteneur
- ✅ Supprimer proprement Redis et ses données

**Prochaines étapes :**
- 🔒 [6.2 - Configuration avec mot de passe](02-config-avec-password.md)
- ⚙️ [6.3 - Configuration avancée avec redis.conf](03-config-avec-redisconf.md)
- 🌐 [6.4 - Configuration avec IP fixe](04-config-ip-fixe.md)
- 🖥️ [6.5 - Redis avec Redis Commander (GUI)](05-redis-commander.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
