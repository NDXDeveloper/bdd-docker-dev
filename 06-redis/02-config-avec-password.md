# 6.2 - Configuration de Redis avec mot de passe

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 🔒 Pourquoi protéger Redis avec un mot de passe ?

Par défaut, Redis **n'a aucune sécurité** ! N'importe qui pouvant accéder au port 6379 peut :
- ❌ Lire toutes vos données
- ❌ Modifier ou supprimer vos données
- ❌ Exécuter des commandes dangereuses (FLUSHALL, CONFIG, etc.)
- ❌ Utiliser Redis pour attaquer d'autres systèmes

**Même en développement**, c'est une bonne pratique d'activer l'authentification pour :
- ✅ Prendre de bonnes habitudes dès le début
- ✅ Éviter les accidents (suppression accidentelle)
- ✅ Se préparer à la production
- ✅ Protéger votre réseau local

> 💡 **Bon à savoir :** Redis utilise le mécanisme `AUTH` avec un mot de passe unique pour tous les utilisateurs (contrairement aux BDD SQL qui ont plusieurs utilisateurs).

---

## 🎯 Objectif de cette fiche

Vous allez apprendre à :
1. ✅ Configurer Redis avec un mot de passe
2. ✅ Utiliser un fichier `.env` pour stocker le mot de passe de manière sécurisée
3. ✅ Vous connecter à Redis en fournissant le mot de passe
4. ✅ Connecter vos applications avec authentification
5. ✅ Gérer les erreurs d'authentification

---

## 🚀 Méthode 1 : Mot de passe via variable d'environnement

### **Étape 1 : Créer le dossier du projet**

```bash
mkdir redis-secure
cd redis-secure
```

### **Étape 2 : Créer le fichier `.env`**

Le fichier `.env` permet de stocker vos mots de passe en dehors du code. **Ne jamais versionner ce fichier sur Git !**

Créez un fichier `.env` avec ce contenu :

```bash
# Mot de passe Redis
REDIS_PASSWORD=VotreMotDePasseSecurise123!
```

⚠️ **Important :** Remplacez `VotreMotDePasseSecurise123!` par un vrai mot de passe fort :
- Au moins 16 caractères
- Mélange de majuscules, minuscules, chiffres et symboles
- Pas de mots du dictionnaire

**Générateur de mot de passe en ligne de commande :**
```bash
# Sous Linux/macOS
openssl rand -base64 32

# Sous Windows (PowerShell)
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }))
```

### **Étape 3 : Créer le fichier `docker-compose.yml`**

```yaml
version: '3.8'

services:
  redis:
    # Image officielle Redis
    image: redis:7-alpine

    # Nom du conteneur
    container_name: redis_secure

    # Redémarre automatiquement
    restart: unless-stopped

    # Configuration avec mot de passe
    command: redis-server --requirepass ${REDIS_PASSWORD}

    # Variables d'environnement (récupérées depuis .env)
    environment:
      - REDIS_PASSWORD=${REDIS_PASSWORD}

    # Exposition du port
    ports:
      - "6379:6379"

    # Persistance des données
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### **Étape 4 : Démarrer Redis**

```bash
docker-compose up -d
```

✅ **Redis est maintenant protégé par mot de passe !**

### **Étape 5 : Vérifier que l'authentification fonctionne**

Essayons de nous connecter **sans** mot de passe :

```bash
docker exec -it redis_secure redis-cli
```

Une fois dans `redis-cli`, essayez une commande :

```redis
PING
```

**Résultat attendu :**
```
(error) NOAUTH Authentication required.
```

✅ Parfait ! Redis refuse l'accès sans authentification.

Tapez `EXIT` pour quitter.

---

## 🔐 Se connecter avec le mot de passe

### **Méthode 1 : S'authentifier après connexion**

```bash
# Se connecter sans authentification
docker exec -it redis_secure redis-cli

# Puis s'authentifier dans redis-cli
AUTH VotreMotDePasseSecurise123!
```

**Résultat :** `OK`

Maintenant vous pouvez utiliser Redis normalement :
```redis
PING
```
**Résultat :** `PONG`

### **Méthode 2 : S'authentifier directement à la connexion**

Plus rapide et plus pratique :

```bash
docker exec -it redis_secure redis-cli -a VotreMotDePasseSecurise123!
```

⚠️ **Avertissement affiché :**
```
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
```

C'est normal. Docker affiche cet avertissement car le mot de passe peut apparaître dans l'historique de commandes. Pour un usage en développement local, ce n'est pas grave.

### **Méthode 3 : Utiliser la variable d'environnement**

Encore plus sécurisé :

```bash
# Charger le mot de passe depuis .env
export REDIS_PASSWORD=$(grep REDIS_PASSWORD .env | cut -d '=' -f2)

# Se connecter
docker exec -it redis_secure redis-cli -a $REDIS_PASSWORD
```

---

## 🧪 Tester l'authentification

Une fois connecté avec le mot de passe :

```redis
# Test de connexion
PING
# Résultat : PONG

# Stocker des données
SET utilisateur:1 "Alice"
SET utilisateur:2 "Bob"

# Récupérer des données
GET utilisateur:1
# Résultat : "Alice"

# Lister les clés
KEYS *
# Résultat : 1) "utilisateur:1" 2) "utilisateur:2"

# Quitter
EXIT
```

---

## 💻 Se connecter depuis une application

### **Python avec redis-py**

Installez la bibliothèque Redis :
```bash
pip install redis
```

Créez un fichier `test_redis_auth.py` :

```python
import redis
import os
from dotenv import load_dotenv

# Charger les variables depuis .env
load_dotenv()

# Récupérer le mot de passe
password = os.getenv('REDIS_PASSWORD')

# Connexion avec authentification
r = redis.Redis(
    host='localhost',
    port=6379,
    password=password,
    decode_responses=True
)

# Test de connexion
try:
    r.ping()
    print("✅ Connexion réussie avec authentification !")
except redis.AuthenticationError:
    print("❌ Erreur d'authentification - Mot de passe incorrect")
except Exception as e:
    print(f"❌ Erreur : {e}")

# Utilisation normale
r.set('langage', 'Python')
print(f"Valeur stockée : {r.get('langage')}")

# Compteur
r.set('visiteurs', 0)
r.incr('visiteurs')
r.incr('visiteurs')
print(f"Nombre de visiteurs : {r.get('visiteurs')}")
```

**Installation de python-dotenv :**
```bash
pip install python-dotenv
```

**Exécution :**
```bash
python test_redis_auth.py
```

**Résultat attendu :**
```
✅ Connexion réussie avec authentification !
Valeur stockée : Python
Nombre de visiteurs : 2
```

### **Node.js avec redis**

Installez le package Redis :
```bash
npm install redis dotenv
```

Créez un fichier `test_redis_auth.js` :

```javascript
require('dotenv').config();
const redis = require('redis');

// Récupérer le mot de passe depuis .env
const password = process.env.REDIS_PASSWORD;

// Connexion avec authentification
const client = redis.createClient({
  host: 'localhost',
  port: 6379,
  password: password
});

client.on('connect', () => {
  console.log('✅ Connexion réussie avec authentification !');
});

client.on('error', (err) => {
  console.error('❌ Erreur :', err.message);
});

// Utilisation normale
client.set('framework', 'Express', redis.print);

client.get('framework', (err, reply) => {
  if (err) {
    console.error('❌ Erreur :', err);
  } else {
    console.log('Valeur stockée :', reply);
  }
  client.quit();
});
```

**Exécution :**
```bash
node test_redis_auth.js
```

### **PHP avec predis**

Installez Predis via Composer :
```bash
composer require predis/predis
composer require vlucas/phpdotenv
```

Créez un fichier `test_redis_auth.php` :

```php
<?php
require 'vendor/autoload.php';

// Charger les variables d'environnement
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

$password = $_ENV['REDIS_PASSWORD'];

// Connexion avec authentification
$redis = new Predis\Client([
    'scheme' => 'tcp',
    'host'   => 'localhost',
    'port'   => 6379,
    'password' => $password,
]);

try {
    $redis->ping();
    echo "✅ Connexion réussie avec authentification !\n";

    // Utilisation normale
    $redis->set('langage', 'PHP');
    echo "Valeur stockée : " . $redis->get('langage') . "\n";

} catch (Exception $e) {
    echo "❌ Erreur : " . $e->getMessage() . "\n";
}
?>
```

**Exécution :**
```bash
php test_redis_auth.php
```

---

## 🚀 Méthode 2 : Mot de passe via fichier de configuration

Pour une configuration plus avancée, utilisez un fichier `redis.conf`.

### **Étape 1 : Créer le fichier `redis.conf`**

```bash
# Protection par mot de passe
requirepass VotreMotDePasseSecurise123!

# Persistance des données (snapshot RDB)
save 900 1
save 300 10
save 60 10000

# Nom du fichier de sauvegarde
dbfilename dump.rdb

# Dossier de sauvegarde
dir /data

# Log level (debug, verbose, notice, warning)
loglevel notice

# Nombre maximum de clients
maxclients 10000

# Limite de mémoire (exemple : 256 MB)
maxmemory 256mb

# Politique d'éviction quand la mémoire est pleine
maxmemory-policy allkeys-lru
```

### **Étape 2 : Créer le fichier `docker-compose.yml`**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis_secure_conf
    restart: unless-stopped

    # Utiliser le fichier de configuration personnalisé
    command: redis-server /usr/local/etc/redis/redis.conf

    ports:
      - "6379:6379"

    volumes:
      # Monter le fichier de configuration
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      # Persistance des données
      - redis_data:/data

volumes:
  redis_data:
```

### **Étape 3 : Démarrer Redis**

```bash
docker-compose up -d
```

La connexion se fait exactement de la même manière que dans la Méthode 1.

---

## 🔄 Changer le mot de passe

### **Si vous utilisez la Méthode 1 (variable d'environnement)**

1. Modifiez le fichier `.env` :
```bash
REDIS_PASSWORD=NouveauMotDePasseTresSecurise456!
```

2. Redémarrez le conteneur :
```bash
docker-compose down
docker-compose up -d
```

### **Si vous utilisez la Méthode 2 (redis.conf)**

1. Modifiez le fichier `redis.conf` :
```bash
requirepass NouveauMotDePasseTresSecurise456!
```

2. Redémarrez le conteneur :
```bash
docker-compose restart
```

### **Changer le mot de passe à chaud (sans redémarrage)**

Connectez-vous avec l'ancien mot de passe :
```bash
docker exec -it redis_secure redis-cli -a AncienMotDePasse
```

Puis exécutez :
```redis
CONFIG SET requirepass NouveauMotDePasse
```

**Résultat :** `OK`

Maintenant, reconnectez-vous avec le nouveau mot de passe :
```redis
AUTH NouveauMotDePasse
```

⚠️ **Attention :** Cette modification n'est **pas persistante**. Au redémarrage du conteneur, l'ancien mot de passe sera restauré. Pour une modification permanente, modifiez le fichier `.env` ou `redis.conf`.

---

## 🛡️ Sécurité supplémentaire

### **1. Désactiver les commandes dangereuses**

Dans votre `redis.conf`, ajoutez :

```bash
# Désactiver des commandes sensibles
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG ""
rename-command SHUTDOWN ""
rename-command SAVE ""
```

Ces commandes seront complètement désactivées.

Ou renommez-les avec un nom secret :
```bash
rename-command CONFIG "MonCommandeSecrete8574"
```

Pour utiliser la commande renommée :
```redis
MonCommandeSecrete8574 GET requirepass
```

### **2. Limiter les adresses IP autorisées**

Dans `redis.conf` :

```bash
# N'écouter que sur localhost (pas d'accès externe)
bind 127.0.0.1

# Ou autoriser une IP spécifique
bind 127.0.0.1 192.168.1.100
```

### **3. Mode protégé**

```bash
# Active le mode protégé (activé par défaut)
protected-mode yes
```

En mode protégé, Redis refuse les connexions externes si :
- Aucun mot de passe n'est configuré
- Aucun `bind` n'est défini

### **4. Chiffrement SSL/TLS (Redis 6+)**

Pour chiffrer les communications, consultez la documentation officielle :
[Redis TLS Support](https://redis.io/docs/manual/security/encryption/)

---

## ❓ Questions fréquentes

### **Mon application ne se connecte plus après avoir ajouté un mot de passe. Pourquoi ?**

Vous avez oublié de fournir le mot de passe dans la configuration de connexion :

```python
# ❌ Sans mot de passe (ne fonctionne plus)
r = redis.Redis(host='localhost', port=6379)

# ✅ Avec mot de passe
r = redis.Redis(host='localhost', port=6379, password='VotreMotDePasse')
```

### **Comment voir le mot de passe actuellement configuré ?**

Connectez-vous avec authentification, puis :

```redis
CONFIG GET requirepass
```

**Résultat :**
```
1) "requirepass"
2) "VotreMotDePasseSecurise123!"
```

### **Puis-je avoir plusieurs utilisateurs avec des droits différents ?**

Oui, depuis **Redis 6** avec le système ACL (Access Control List). Voir la fiche **6.3 - Configuration avancée avec redis.conf** pour plus de détails.

Pour Redis < 6, il n'y a qu'un seul mot de passe global.

### **Le mot de passe est-il chiffré dans redis.conf ?**

Non, le mot de passe est stocké **en clair** dans le fichier de configuration. C'est pourquoi :
- Il faut protéger l'accès au fichier `redis.conf`
- Utiliser des permissions restrictives : `chmod 600 redis.conf`
- Ne jamais versionner ce fichier sur Git

### **Que se passe-t-il si j'oublie mon mot de passe ?**

Si vous avez accès au serveur Docker :

1. Arrêtez Redis
2. Modifiez `.env` ou `redis.conf` pour changer le mot de passe
3. Redémarrez Redis

Si vous n'avez pas accès aux fichiers, vous devrez recréer le conteneur (perte des données).

### **Redis peut-il bloquer après plusieurs tentatives échouées ?**

Non, par défaut Redis **ne bloque pas** les connexions après plusieurs échecs. Il est recommandé d'utiliser un pare-feu (fail2ban) pour détecter et bloquer les attaques par force brute.

---

## 🔒 Fichier `.gitignore` recommandé

Créez un fichier `.gitignore` pour ne **jamais** versionner vos mots de passe :

```bash
# Fichiers avec mots de passe
.env
.env.local
.env.*.local

# Fichiers de configuration avec secrets
redis.conf

# Dossiers de données
data/
redis_data/

# Logs
*.log
```

---

## 📊 Comparaison des méthodes

| Critère | Variable d'env (.env) | Fichier redis.conf |
|---------|----------------------|-------------------|
| **Simplicité** | ⭐⭐⭐⭐⭐ Très simple | ⭐⭐⭐ Moyen |
| **Flexibilité** | ⭐⭐⭐ Limitée | ⭐⭐⭐⭐⭐ Très flexible |
| **Sécurité** | ⭐⭐⭐⭐ Bonne | ⭐⭐⭐⭐ Bonne |
| **Configuration** | Mot de passe uniquement | Toutes les options Redis |
| **Recommandé pour** | Développement simple | Configuration avancée |

---

## ✅ Checklist de sécurité

Avant de considérer votre Redis sécurisé :

- [ ] ✅ Mot de passe configuré (`requirepass`)
- [ ] ✅ Mot de passe fort (16+ caractères, complexe)
- [ ] ✅ Fichier `.env` ou `redis.conf` non versionné sur Git
- [ ] ✅ Commandes dangereuses désactivées ou renommées
- [ ] ✅ `bind` configuré pour limiter les accès
- [ ] ✅ `protected-mode` activé
- [ ] ✅ Pare-feu configuré (si accessible depuis l'extérieur)
- [ ] ✅ Backups réguliers des données
- [ ] ✅ Logs surveillés
- [ ] ✅ Redis à jour (dernière version stable)

---

## 🎉 Récapitulatif

Vous savez maintenant :
- ✅ Configurer Redis avec un mot de passe (2 méthodes)
- ✅ Utiliser un fichier `.env` pour sécuriser vos secrets
- ✅ Vous connecter à Redis avec authentification
- ✅ Connecter vos applications (Python, Node.js, PHP)
- ✅ Changer le mot de passe
- ✅ Appliquer des mesures de sécurité supplémentaires
- ✅ Éviter les erreurs courantes d'authentification

**Prochaines étapes :**
- ⚙️ [6.3 - Configuration avancée avec redis.conf](03-config-avec-redisconf.md)
- 🌐 [6.4 - Configuration avec IP fixe](04-config-ip-fixe.md)
- 🖥️ [6.5 - Redis avec Redis Commander (GUI)](05-redis-commander.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
