# 6.5 - Redis avec Redis Commander (Interface graphique)

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 🖥️ Qu'est-ce que Redis Commander ?

**Redis Commander** est une interface graphique web pour gérer Redis. C'est l'équivalent de phpMyAdmin pour MySQL, mais pour Redis.

**Avantages de Redis Commander :**
- ✅ **Interface web intuitive** : Pas besoin de mémoriser les commandes
- ✅ **Visualisation des données** : Voir toutes les clés et leurs valeurs
- ✅ **Édition en temps réel** : Modifier, ajouter, supprimer des clés
- ✅ **Support multi-bases** : Gérer plusieurs instances Redis
- ✅ **Navigation par pattern** : Filtrer les clés facilement
- ✅ **Import/Export** : Sauvegarder et restaurer des données
- ✅ **Analyse de mémoire** : Voir l'utilisation par type de données
- ✅ **Open source et léger** : Facile à déployer avec Docker

**Alternatives à Redis Commander :**
- **RedisInsight** (officiel) : Plus complet mais plus lourd
- **Redis Desktop Manager** : Application desktop (payante pour Pro)
- **phpRedisAdmin** : Interface PHP
- **redis-cli** : Client en ligne de commande (pas graphique)

---

## 🎯 Objectif de cette fiche

Vous allez apprendre à :
1. ✅ Déployer Redis Commander avec Docker Compose
2. ✅ Configurer la connexion à Redis
3. ✅ Naviguer dans l'interface web
4. ✅ Gérer vos données Redis visuellement
5. ✅ Utiliser Redis Commander avec authentification
6. ✅ Connecter Redis Commander à plusieurs instances Redis

---

## 🚀 Configuration de base

### **Étape 1 : Créer le dossier du projet**

```bash
mkdir redis-commander
cd redis-commander
```

### **Étape 2 : Créer le fichier `docker-compose.yml`**

Configuration simple avec Redis + Redis Commander :

```yaml
version: '3.8'

services:
  # Serveur Redis
  redis:
    image: redis:7-alpine
    container_name: redis_server
    restart: unless-stopped
    command: redis-server --requirepass MonMotDePasse123!
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - redis_net

  # Interface graphique Redis Commander
  redis-commander:
    image: ghcr.io/joeferner/redis-commander:latest
    container_name: redis_commander
    restart: unless-stopped
    environment:
      # Configuration de la connexion Redis
      - REDIS_HOSTS=local:redis:6379:0:MonMotDePasse123!
    ports:
      # Interface web accessible sur http://localhost:8081
      - "8081:8081"
    networks:
      - redis_net
    depends_on:
      - redis

volumes:
  redis_data:

networks:
  redis_net:
    driver: bridge
```

### **Étape 3 : Démarrer les services**

```bash
docker-compose up -d
```

**Vérification :**
```bash
docker-compose ps
```

Vous devriez voir deux conteneurs en état `Up` :
- `redis_server` (port 6379)
- `redis_commander` (port 8081)

### **Étape 4 : Accéder à l'interface web**

Ouvrez votre navigateur et allez à :

```
http://localhost:8081
```

🎉 **Vous devriez voir l'interface Redis Commander !**

---

## 🔐 Configuration avec authentification

### **Protéger l'accès à Redis Commander**

Par défaut, Redis Commander est accessible **sans mot de passe** ! Pour le sécuriser :

**Fichier `docker-compose.yml` avec authentification :**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis_server
    restart: unless-stopped
    command: redis-server --requirepass RedisPassword123!
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - redis_net

  redis-commander:
    image: ghcr.io/joeferner/redis-commander:latest
    container_name: redis_commander
    restart: unless-stopped
    environment:
      # Connexion Redis
      - REDIS_HOSTS=local:redis:6379:0:RedisPassword123!

      # 🔒 Authentification HTTP Basic pour Redis Commander
      - HTTP_USER=admin
      - HTTP_PASSWORD=SecureAdminPassword456!

      # Configuration optionnelle
      - ADDRESS=0.0.0.0
      - PORT=8081
    ports:
      - "8081:8081"
    networks:
      - redis_net
    depends_on:
      - redis

volumes:
  redis_data:

networks:
  redis_net:
    driver: bridge
```

**Maintenant, pour accéder à Redis Commander :**
1. Ouvrez `http://localhost:8081`
2. Entrez les identifiants :
   - **Utilisateur :** `admin`
   - **Mot de passe :** `SecureAdminPassword456!`

---

## 🌐 Configuration avec plusieurs instances Redis

### **Gérer plusieurs serveurs Redis depuis une seule interface**

**Fichier `docker-compose.yml` :**

```yaml
version: '3.8'

services:
  # Redis pour développement
  redis_dev:
    image: redis:7-alpine
    container_name: redis_dev
    restart: unless-stopped
    command: redis-server --requirepass DevPass123!
    ports:
      - "6379:6379"
    volumes:
      - redis_dev_data:/data
    networks:
      - redis_net

  # Redis pour tests
  redis_test:
    image: redis:7-alpine
    container_name: redis_test
    restart: unless-stopped
    command: redis-server --requirepass TestPass123!
    ports:
      - "6380:6379"
    volumes:
      - redis_test_data:/data
    networks:
      - redis_net

  # Redis pour staging
  redis_staging:
    image: redis:7-alpine
    container_name: redis_staging
    restart: unless-stopped
    command: redis-server --requirepass StagingPass123!
    ports:
      - "6381:6379"
    volumes:
      - redis_staging_data:/data
    networks:
      - redis_net

  # Redis Commander pour gérer toutes les instances
  redis-commander:
    image: ghcr.io/joeferner/redis-commander:latest
    container_name: redis_commander
    restart: unless-stopped
    environment:
      # Syntaxe : label:host:port:db:password
      # Séparer plusieurs instances par des virgules
      - REDIS_HOSTS=dev:redis_dev:6379:0:DevPass123!,test:redis_test:6379:0:TestPass123!,staging:redis_staging:6379:0:StagingPass123!

      # Authentification Redis Commander
      - HTTP_USER=admin
      - HTTP_PASSWORD=AdminPassword789!
    ports:
      - "8081:8081"
    networks:
      - redis_net
    depends_on:
      - redis_dev
      - redis_test
      - redis_staging

volumes:
  redis_dev_data:
  redis_test_data:
  redis_staging_data:

networks:
  redis_net:
    driver: bridge
```

**Accès :**
Ouvrez `http://localhost:8081` et vous verrez un **sélecteur** pour choisir entre :
- `dev` (Redis développement)
- `test` (Redis tests)
- `staging` (Redis staging)

---

## 📖 Guide d'utilisation de l'interface

### **1. Page d'accueil**

Après connexion, vous arrivez sur la page principale avec :
- **En haut à gauche :** Sélecteur d'instance Redis (si plusieurs)
- **En haut à droite :** Sélecteur de base de données (0-15 par défaut)
- **Barre latérale gauche :** Liste des clés existantes
- **Zone principale :** Détails de la clé sélectionnée

### **2. Navigation dans les clés**

**Filtrer les clés :**
- Dans la barre de recherche, tapez un pattern : `user:*`, `session:*`, etc.
- Supporte les wildcards Redis : `*`, `?`, `[abc]`

**Organiser les clés :**
- Les clés avec `:` sont automatiquement regroupées (ex: `user:1`, `user:2` → dossier `user`)
- Cliquez sur un dossier pour voir toutes les clés qu'il contient

**Trier les clés :**
- Par nom (alphabétique)
- Par type (string, hash, list, set, zset)
- Par TTL (time to live)

### **3. Voir les détails d'une clé**

Cliquez sur une clé dans la liste pour voir :
- **Type de données** : String, Hash, List, Set, Sorted Set
- **Valeur(s)** : Contenu de la clé
- **TTL** : Temps avant expiration (-1 = pas d'expiration)
- **Taille** : Taille en mémoire
- **Encodage** : Encodage interne Redis

### **4. Modifier une clé**

**Pour une String :**
1. Cliquez sur la clé
2. Modifiez la valeur dans le champ de texte
3. Cliquez sur **"Save"**

**Pour un Hash :**
1. Cliquez sur la clé
2. Cliquez sur **"Edit"** pour un champ spécifique
3. Modifiez la valeur
4. Cliquez sur **"Save"**

**Pour une List :**
1. Cliquez sur la clé
2. Cliquez sur **"+"** pour ajouter un élément
3. Cliquez sur **"×"** pour supprimer un élément

### **5. Ajouter une nouvelle clé**

1. Cliquez sur le bouton **"Add Key"** en haut
2. Choisissez le **type de données** (String, Hash, List, Set, Sorted Set)
3. Entrez le **nom de la clé**
4. Entrez la **valeur** (selon le type)
5. (Optionnel) Définissez un **TTL** en secondes
6. Cliquez sur **"Save"**

**Exemple - Ajouter une String :**
- **Type :** String
- **Key :** `user:123:name`
- **Value :** `Alice Dupont`
- **TTL :** `3600` (expire dans 1 heure)

**Exemple - Ajouter un Hash :**
- **Type :** Hash
- **Key :** `user:123`
- **Fields :**
  - `name` → `Alice Dupont`
  - `email` → `alice@example.com`
  - `age` → `30`

### **6. Supprimer une clé**

1. Cliquez sur la clé dans la liste
2. Cliquez sur le bouton **"Delete"** (icône poubelle)
3. Confirmez la suppression

**Supprimer plusieurs clés :**
1. Utilisez un pattern dans la barre de recherche : `test:*`
2. Cochez les clés à supprimer
3. Cliquez sur **"Delete Selected"**

⚠️ **Attention :** La suppression est **immédiate et irréversible** !

### **7. Définir un TTL (expiration)**

1. Cliquez sur une clé
2. Dans la section **"TTL"**, entrez le nombre de secondes
3. Cliquez sur **"Set TTL"**

**Exemples de TTL :**
- `60` = expire dans 1 minute
- `3600` = expire dans 1 heure
- `86400` = expire dans 24 heures
- `-1` = ne jamais expirer

**Supprimer le TTL :**
1. Entrez `-1` dans le champ TTL
2. Cliquez sur **"Set TTL"**

### **8. Exporter des données**

1. Cliquez sur le bouton **"Export"**
2. Choisissez le format :
   - **JSON** : Pour sauvegarder les données
   - **Redis CLI** : Commandes Redis à rejouer
3. Téléchargez le fichier

### **9. Importer des données**

1. Cliquez sur **"Import"**
2. Sélectionnez un fichier JSON ou Redis CLI
3. Confirmez l'import

⚠️ **Attention :** L'import **écrase** les clés existantes avec le même nom.

### **10. Monitorer en temps réel**

1. Cliquez sur **"Console"** en haut
2. Vous voyez toutes les commandes Redis exécutées en temps réel
3. Utile pour déboguer les applications

### **11. Informations serveur**

1. Cliquez sur **"Server Info"**
2. Vous voyez :
   - Version de Redis
   - Mémoire utilisée
   - Nombre de clés
   - Statistiques de connexion
   - Configuration actuelle

---

## 🔧 Variables d'environnement

### **Configuration de Redis Commander**

| Variable | Description | Exemple |
|----------|-------------|---------|
| `REDIS_HOSTS` | Liste des serveurs Redis | `local:redis:6379:0:password` |
| `HTTP_USER` | Utilisateur HTTP Basic Auth | `admin` |
| `HTTP_PASSWORD` | Mot de passe HTTP Basic Auth | `SecurePass123!` |
| `ADDRESS` | Adresse d'écoute | `0.0.0.0` (toutes) |
| `PORT` | Port d'écoute | `8081` |
| `ROOT_PATTERN` | Pattern par défaut | `*` |
| `REDIS_LABEL` | Label par défaut | `local` |
| `REDIS_DB` | Base de données par défaut | `0` |
| `READ_ONLY` | Mode lecture seule | `true` ou `false` |

### **Format de REDIS_HOSTS**

**Syntaxe :**
```
label:host:port:db_number:password
```

**Exemples :**

```bash
# Un seul Redis sans mot de passe
REDIS_HOSTS=local:redis:6379:0

# Un seul Redis avec mot de passe
REDIS_HOSTS=local:redis:6379:0:MyPassword123!

# Plusieurs Redis
REDIS_HOSTS=dev:redis1:6379:0:Pass1,prod:redis2:6379:0:Pass2

# Redis sur l'hôte (depuis Docker)
REDIS_HOSTS=local:host.docker.internal:6379:0:Pass
```

### **Mode lecture seule**

Pour empêcher toute modification :

```yaml
environment:
  - REDIS_HOSTS=local:redis:6379:0:password
  - READ_ONLY=true
```

Avec `READ_ONLY=true` :
- ✅ Vous pouvez **voir** toutes les données
- ❌ Vous **ne pouvez pas** ajouter, modifier ou supprimer

---

## 🎨 Configuration avec fichier .env

Pour sécuriser vos mots de passe, utilisez un fichier `.env`.

### **Fichier `.env` :**

```bash
# Mot de passe Redis
REDIS_PASSWORD=RedisSecurePassword123!

# Authentification Redis Commander
REDIS_COMMANDER_USER=admin
REDIS_COMMANDER_PASSWORD=AdminSecurePassword456!
```

### **Fichier `docker-compose.yml` :**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis_server
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - redis_net

  redis-commander:
    image: ghcr.io/joeferner/redis-commander:latest
    container_name: redis_commander
    restart: unless-stopped
    environment:
      - REDIS_HOSTS=local:redis:6379:0:${REDIS_PASSWORD}
      - HTTP_USER=${REDIS_COMMANDER_USER}
      - HTTP_PASSWORD=${REDIS_COMMANDER_PASSWORD}
    ports:
      - "8081:8081"
    networks:
      - redis_net
    depends_on:
      - redis

volumes:
  redis_data:

networks:
  redis_net:
    driver: bridge
```

**Démarrage :**
```bash
docker-compose up -d
```

Docker Compose charge automatiquement le fichier `.env`.

⚠️ **Important :** Ajoutez `.env` à votre `.gitignore` !

---

## 🔒 Sécurité

### **Bonnes pratiques**

1. **Toujours activer HTTP_USER et HTTP_PASSWORD** :
   ```yaml
   environment:
     - HTTP_USER=admin
     - HTTP_PASSWORD=VotreMotDePasseTresFort!
   ```

2. **Ne pas exposer Redis Commander sur Internet** :
   ```yaml
   ports:
     - "127.0.0.1:8081:8081"  # Accessible uniquement en local
   ```

3. **Utiliser HTTPS en production** :
   Placez Redis Commander derrière un reverse proxy (Nginx, Traefik) avec certificat SSL.

4. **Mode lecture seule pour les utilisateurs non-admin** :
   ```yaml
   environment:
     - READ_ONLY=true
   ```

5. **Limiter les patterns de clés visibles** :
   ```yaml
   environment:
     - ROOT_PATTERN=user:*  # Ne montre que les clés "user:*"
   ```

6. **Sauvegarder régulièrement** :
   Utilisez la fonction Export pour faire des backups.

---

## 🔍 Dépannage

### **Problème : "Connection refused" ou "ECONNREFUSED"**

**Causes possibles :**
1. Redis n'est pas démarré
2. Mauvais nom d'hôte dans `REDIS_HOSTS`
3. Mauvais mot de passe

**Solutions :**
```bash
# Vérifier que Redis fonctionne
docker-compose ps

# Vérifier les logs de Redis
docker-compose logs redis

# Vérifier les logs de Redis Commander
docker-compose logs redis-commander

# Tester la connexion manuellement
docker exec -it redis_server redis-cli -a VotreMotDePasse PING
```

### **Problème : "WRONGPASS invalid username-password pair"**

Le mot de passe dans `REDIS_HOSTS` est incorrect.

**Solution :**
Vérifiez que le mot de passe correspond :
```yaml
# Dans le service redis
command: redis-server --requirepass MonMotDePasse123!

# Dans redis-commander
environment:
  - REDIS_HOSTS=local:redis:6379:0:MonMotDePasse123!
```

### **Problème : Page blanche ou "Cannot GET /"**

Redis Commander n'a pas démarré correctement.

**Solution :**
```bash
# Voir les logs
docker-compose logs redis-commander

# Redémarrer
docker-compose restart redis-commander
```

### **Problème : Modifications non sauvegardées**

**Causes possibles :**
1. Mode `READ_ONLY=true` activé
2. Redis est en mode protected sans authentification
3. Problème de permissions

**Solution :**
```bash
# Vérifier la configuration
docker exec redis-commander env | grep READ_ONLY

# Désactiver READ_ONLY si nécessaire
# Dans docker-compose.yml, supprimer ou mettre à false
```

### **Problème : Redis Commander lent avec beaucoup de clés**

**Solution :**
Utilisez des patterns pour limiter les clés affichées :
```yaml
environment:
  - ROOT_PATTERN=user:*  # Ne charge que les clés commençant par "user:"
```

Ou dans l'interface, utilisez la barre de recherche pour filtrer.

---

## 📊 Alternatives à Redis Commander

| Outil | Type | Avantages | Inconvénients |
|-------|------|-----------|---------------|
| **Redis Commander** | Web | Léger, simple | Interface basique |
| **RedisInsight** | Desktop/Web | Officiel, complet | Plus lourd |
| **Redis Desktop Manager** | Desktop | Riche en features | Payant (Pro) |
| **phpRedisAdmin** | Web | Comme phpMyAdmin | Nécessite PHP |
| **redis-cli** | CLI | Léger, rapide | Pas d'interface graphique |
| **Medis** | Desktop (macOS) | Joli, moderne | macOS uniquement |

---

## 🎉 Récapitulatif

Vous savez maintenant :
- ✅ Déployer Redis Commander avec Docker Compose
- ✅ Configurer l'authentification (Redis + Redis Commander)
- ✅ Gérer plusieurs instances Redis depuis une seule interface
- ✅ Utiliser l'interface pour voir, ajouter, modifier, supprimer des clés
- ✅ Exporter et importer des données
- ✅ Configurer le mode lecture seule
- ✅ Sécuriser l'accès à Redis Commander
- ✅ Résoudre les problèmes courants

**Prochaines étapes :**
- 📚 [Annexe A - Référence des commandes Docker](../annexes/A-reference-commandes.md)
- 🌐 [Annexe B - Gestion des réseaux Docker](../annexes/B-gestion-reseaux.md)
- 🔒 [Annexe D - Sécurité et bonnes pratiques](../annexes/D-securite-bonnes-pratiques.md)
- 🛠️ [Annexe F - Outils de gestion](../annexes/F-outils-gestion.md)

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

*Dernière mise à jour : Octobre 2025*
