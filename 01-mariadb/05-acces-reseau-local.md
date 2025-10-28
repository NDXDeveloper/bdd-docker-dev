# Accès à MariaDB depuis le réseau local

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Cette fiche vous apprend à configurer votre conteneur MariaDB pour qu'il soit accessible depuis d'autres ordinateurs de votre réseau local (domicile, bureau, etc.). C'est très utile pour le travail en équipe ou pour tester votre application depuis différents appareils.

**Ce que vous allez apprendre :**
- Comprendre la différence entre accès local et réseau
- Identifier l'adresse IP de votre machine hôte
- Configurer le pare-feu pour autoriser les connexions
- Créer des utilisateurs adaptés pour l'accès réseau
- Tester la connexion depuis un autre ordinateur
- Résoudre les problèmes courants

**Durée estimée :** 20 minutes

---

## 🎯 Prérequis

Avant de commencer, vous devez :

- ✅ Avoir suivi la [fiche 1.1 - Configuration basique](01-config-basique-docker-compose.md)
- ✅ Avoir lu la [fiche 1.4 - Gestion des utilisateurs](04-gestion-utilisateurs.md)
- ✅ Avoir deux ordinateurs sur le même réseau (même WiFi/Ethernet)

---

## 🤔 Comprendre l'accès réseau

### Les trois niveaux d'accès

```
┌─────────────────────────────────────────────────────────┐
│  1. ACCÈS CONTENEUR (172.18.0.10)                       │
│     └─ Accessible uniquement par d'autres conteneurs    │
│                                                         │
│  2. ACCÈS LOCALHOST (127.0.0.1:3306)                    │
│     └─ Accessible depuis votre machine uniquement       │
│                                                         │
│  3. ACCÈS RÉSEAU LOCAL (192.168.1.50:3306)              │
│     └─ Accessible depuis d'autres machines du réseau    │
└─────────────────────────────────────────────────────────┘
```

### Schéma de connexion réseau

```
┌──────────────────────────────────────────────────────────┐
│  Réseau Local (ex: WiFi maison - 192.168.1.0/24)         │
│                                                          │
│  ┌─────────────────────┐        ┌──────────────────────┐ │
│  │ ORDINATEUR A        │        │ ORDINATEUR B         │ │
│  │ (Serveur Docker)    │        │ (Client)             │ │
│  │ IP: 192.168.1.50    │◄───────│ IP: 192.168.1.100    │ │
│  │                     │        │                      │ │
│  │ ┌─────────────────┐ │        │ [Client SQL]         │ │
│  │ │ Docker          │ │        │ DBeaver, HeidiSQL... │ │
│  │ │                 │ │        │                      │ │
│  │ │ MariaDB         │ │        │ Se connecte à:       │ │
│  │ │ Port 3306       │ │        │ 192.168.1.50:3306    │ │
│  │ └─────────────────┘ │        │                      │ │
│  └─────────────────────┘        └──────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

---

## 📍 Étape 1 : Identifier l'adresse IP de votre machine hôte

### 1.1 Sur Windows

**Méthode graphique :**
1. Ouvrir les Paramètres → Réseau et Internet → WiFi (ou Ethernet)
2. Cliquer sur votre réseau actif
3. Chercher "Adresse IPv4"

**Méthode ligne de commande :**
```cmd
ipconfig
```

**Exemple de sortie :**
```
Carte réseau sans fil Wi-Fi :

   Adresse IPv4. . . . . . . . . . . . . .: 192.168.1.50
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . : 192.168.1.1
```

➡️ **Notez l'adresse IPv4** : `192.168.1.50` (exemple)

### 1.2 Sur Linux

```bash
# Méthode 1 (moderne)
ip addr show

# Méthode 2 (classique)
ifconfig

# Méthode 3 (simple)
hostname -I
```

**Exemple de sortie :**
```
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.50/24 brd 192.168.1.255 scope global dynamic
```

➡️ **Notez l'adresse** : `192.168.1.50`

### 1.3 Sur macOS

```bash
# Méthode simple
ifconfig | grep "inet "

# Ou via les Préférences Système
# Préférences Système → Réseau → WiFi → Avancé → TCP/IP
```

**Exemple :**
```
inet 192.168.1.50 netmask 0xffffff00 broadcast 192.168.1.255
```

➡️ **Notez l'adresse** : `192.168.1.50`

### 💡 Astuces pour identifier la bonne IP

- **Recherchez des IP commençant par :**
  - `192.168.x.x` (réseaux domestiques courants)
  - `10.x.x.x` (réseaux d'entreprise)
  - `172.16.x.x` à `172.31.x.x` (réseaux privés)

- **Ignorez :**
  - `127.0.0.1` (localhost)
  - `169.254.x.x` (autoconfiguration, problème réseau)
  - Les adresses Docker (`172.17.x.x`, `172.18.x.x`, etc.)

---

## 🐳 Étape 2 : Vérifier la configuration Docker Compose

### 2.1 Le port mapping est essentiel

Votre `docker-compose.yml` **doit** contenir la section `ports` :

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.11
    container_name: mariadb_reseau
    restart: unless-stopped

    environment:
      MYSQL_ROOT_PASSWORD: mon_mot_de_passe_securise

    # 🔑 SECTION CRITIQUE : Expose le port sur toutes les interfaces
    ports:
      - "3306:3306"

    volumes:
      - mariadb_data:/var/lib/mysql

volumes:
  mariadb_data:
```

### 2.2 Comprendre le port mapping

```yaml
ports:
  - "3306:3306"
```

Signifie :
- **Port gauche (3306)** : Port sur votre machine hôte (accessible via 192.168.1.50:3306)
- **Port droite (3306)** : Port dans le conteneur (où MariaDB écoute)

### 2.3 Vérifier que le port est bien exposé

```bash
# Voir les ports exposés
docker ps

# Résultat attendu :
# PORTS
# 0.0.0.0:3306->3306/tcp
#    ↑
#    └─ 0.0.0.0 signifie "toutes les interfaces" (WiFi, Ethernet, etc.)
```

Si vous voyez `127.0.0.1:3306->3306/tcp`, le port n'est accessible QUE depuis localhost !

### 2.4 Configuration pour localhost uniquement (comparaison)

**Si vous vouliez restreindre à localhost (pas notre objectif) :**
```yaml
ports:
  - "127.0.0.1:3306:3306"  # Accessible UNIQUEMENT depuis la machine hôte
```

**Pour l'accès réseau (notre configuration) :**
```yaml
ports:
  - "3306:3306"  # Accessible depuis le réseau local
  # Équivalent à "0.0.0.0:3306:3306"
```

---

## 🔥 Étape 3 : Configurer le pare-feu

C'est l'**étape la plus importante** et la source la plus courante de problèmes !

### 3.1 Sur Windows (Pare-feu Windows Defender)

#### Méthode graphique (recommandée pour débutants)

1. **Ouvrir le Pare-feu avancé :**
   - Appuyez sur `Win + R`
   - Tapez `wf.msc` et appuyez sur Entrée

2. **Créer une règle entrante :**
   - Cliquez sur "**Règles de trafic entrant**" (dans le panneau gauche)
   - Cliquez sur "**Nouvelle règle...**" (dans le panneau droit)

3. **Assistant de création :**
   - **Type de règle :** Port → Suivant
   - **Protocole :** TCP
   - **Ports locaux spécifiques :** 3306 → Suivant
   - **Action :** Autoriser la connexion → Suivant
   - **Profils :** Cochez **Privé** et **Domaine** (décochez Public) → Suivant
   - **Nom :** "MariaDB Docker 3306"
   - **Description :** "Autorise les connexions MariaDB depuis le réseau local"
   - → Terminer

4. **Vérifier la règle :**
   - Cherchez votre nouvelle règle dans la liste
   - Vérifiez qu'elle est **activée** (coche verte)

#### Méthode PowerShell (avancée)

```powershell
# Exécuter PowerShell en Administrateur
New-NetFirewallRule -DisplayName "MariaDB Docker" -Direction Inbound -Protocol TCP -LocalPort 3306 -Action Allow -Profile Private,Domain
```

### 3.2 Sur Linux (UFW - Ubuntu/Debian)

```bash
# Vérifier si UFW est actif
sudo ufw status

# Autoriser le port 3306 en TCP
sudo ufw allow 3306/tcp

# Vérifier que la règle est ajoutée
sudo ufw status numbered
```

**Sortie attendue :**
```
Status: active

To                         Action      From
--                         ------      ----
3306/tcp                   ALLOW       Anywhere
```

### 3.3 Sur Linux (firewalld - CentOS/Fedora/RHEL)

```bash
# Autoriser le port 3306
sudo firewall-cmd --permanent --add-port=3306/tcp

# Recharger le pare-feu
sudo firewall-cmd --reload

# Vérifier
sudo firewall-cmd --list-ports
```

### 3.4 Sur macOS

macOS n'a généralement pas de pare-feu actif par défaut pour les connexions entrantes.

**Si vous avez activé le pare-feu :**
1. Préférences Système → Sécurité et confidentialité → Pare-feu
2. Cliquer sur "Options du pare-feu..."
3. Ajouter Docker (l'application) et autoriser les connexions entrantes

### 3.5 Tester l'ouverture du port

**Depuis votre machine hôte :**
```bash
# Windows
netstat -an | findstr :3306

# Linux/macOS
netstat -an | grep :3306
# ou
ss -tuln | grep :3306
```

**Résultat attendu :**
```
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN
           ↑
           └─ 0.0.0.0 = écoute sur toutes les interfaces
```

---

## 👤 Étape 4 : Créer un utilisateur avec accès réseau

### 4.1 Se connecter à MariaDB

```bash
docker exec -it mariadb_reseau mariadb -u root -p
```

### 4.2 Créer un utilisateur accessible depuis le réseau

```sql
-- Utilisateur accessible depuis N'IMPORTE QUELLE IP (%)
CREATE USER 'user_reseau'@'%' IDENTIFIED BY 'password_reseau_securise';

-- Donner les permissions (exemple : toutes permissions sur une base)
GRANT ALL PRIVILEGES ON ma_base.* TO 'user_reseau'@'%';

-- Appliquer les changements
FLUSH PRIVILEGES;

-- Vérifier la création
SELECT User, Host FROM mysql.user WHERE User = 'user_reseau';
```

**Résultat attendu :**
```
+--------------+------+
| User         | Host |
+--------------+------+
| user_reseau  | %    |
+--------------+------+
```

### 4.3 Option : Restreindre à un sous-réseau spécifique (plus sécurisé)

```sql
-- Accessible uniquement depuis le réseau 192.168.1.x
CREATE USER 'user_reseau_local'@'192.168.1.%' IDENTIFIED BY 'password_local';

-- Permissions
GRANT ALL PRIVILEGES ON ma_base.* TO 'user_reseau_local'@'192.168.1.%';

FLUSH PRIVILEGES;
```

### 4.4 Pourquoi pas `'user'@'localhost'` ?

**Important :** Un utilisateur créé avec `@'localhost'` ne fonctionne que pour les connexions depuis **l'intérieur du conteneur**.

```sql
-- ❌ NE FONCTIONNERA PAS depuis un autre PC
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';

-- ✅ FONCTIONNERA depuis le réseau
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
```

---

## 🧪 Étape 5 : Tester depuis votre machine hôte

Avant de tester depuis un autre PC, vérifiez que tout fonctionne localement.

### 5.1 Test avec le client MySQL/MariaDB

```bash
# Si vous avez le client MySQL/MariaDB installé sur votre machine
mysql -h 192.168.1.50 -P 3306 -u user_reseau -p

# Ou via localhost (devrait fonctionner aussi)
mysql -h localhost -P 3306 -u user_reseau -p
```

### 5.2 Test avec un client graphique

Utilisez un client comme **DBeaver**, **HeidiSQL**, ou **MySQL Workbench** :

| Paramètre | Valeur |
|-----------|--------|
| Type | MySQL ou MariaDB |
| Hôte | `192.168.1.50` (votre IP) |
| Port | `3306` |
| Utilisateur | `user_reseau` |
| Mot de passe | `password_reseau_securise` |
| Base de données | `ma_base` (ou laisser vide) |

Cliquez sur "Tester la connexion" → Devrait réussir ✅

---

## 💻 Étape 6 : Tester depuis un autre ordinateur du réseau

### 6.1 Prérequis sur l'ordinateur client

- ✅ Être sur le **même réseau** que l'ordinateur serveur (même WiFi/Ethernet)
- ✅ Avoir un **client SQL** installé (DBeaver, HeidiSQL, etc.)
- ✅ Connaître l'**adresse IP** du serveur (ex: 192.168.1.50)

### 6.2 Paramètres de connexion

**Depuis l'ordinateur B (client) :**

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| Hôte | `192.168.1.50` | IP de l'ordinateur A (serveur) |
| Port | `3306` | Port exposé par Docker |
| Utilisateur | `user_reseau` | Utilisateur créé avec `@'%'` |
| Mot de passe | `password_reseau_securise` | Mot de passe de l'utilisateur |

### 6.3 Test avec DBeaver (exemple)

1. **Nouvelle connexion :**
   - Fichier → Nouveau → Connexion à une base de données
   - Sélectionnez "MariaDB" ou "MySQL"

2. **Configuration :**
   - Server Host : `192.168.1.50`
   - Port : `3306`
   - Database : (laisser vide ou spécifier `ma_base`)
   - Username : `user_reseau`
   - Password : `password_reseau_securise`

3. **Tester :**
   - Cliquez sur "Test Connection"
   - **Succès** ✅ : "Connected"
   - **Échec** ❌ : Voir la section Dépannage

### 6.4 Test avec ligne de commande (si client installé)

```bash
# Depuis l'ordinateur B
mysql -h 192.168.1.50 -P 3306 -u user_reseau -p
```

---

## 🛠️ Dépannage (Troubleshooting)

### Problème 1 : "Connection refused" ou "Can't connect to MySQL server"

**Causes possibles :**

1. **MariaDB n'est pas démarré**
   ```bash
   # Sur l'ordinateur A (serveur)
   docker ps
   # Vérifiez que le conteneur est bien "Up"
   ```

2. **Port mapping manquant**
   ```bash
   docker ps
   # Vérifiez la colonne PORTS : doit afficher "0.0.0.0:3306->3306/tcp"
   ```

3. **Mauvaise IP**
   - Vérifiez l'IP avec `ipconfig` (Windows) ou `ip addr` (Linux)
   - Assurez-vous d'utiliser l'IP du réseau local (192.168.x.x), pas Docker (172.x.x.x)

### Problème 2 : "Access denied for user"

**Causes possibles :**

1. **Utilisateur créé avec `@'localhost'` au lieu de `@'%'`**
   ```sql
   -- Vérifier
   SELECT User, Host FROM mysql.user WHERE User = 'user_reseau';

   -- Si Host = 'localhost', recréer l'utilisateur :
   DROP USER 'user_reseau'@'localhost';
   CREATE USER 'user_reseau'@'%' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON ma_base.* TO 'user_reseau'@'%';
   FLUSH PRIVILEGES;
   ```

2. **Mauvais mot de passe**
   - Vérifiez que vous utilisez le bon mot de passe
   - Réinitialisez si nécessaire :
   ```sql
   ALTER USER 'user_reseau'@'%' IDENTIFIED BY 'nouveau_password';
   FLUSH PRIVILEGES;
   ```

### Problème 3 : Timeout ou pas de réponse

**Cause principale : Pare-feu bloqué**

**Windows :**
```powershell
# Vérifier les règles du pare-feu
Get-NetFirewallRule -DisplayName "*MariaDB*"

# Tester le port
Test-NetConnection -ComputerName localhost -Port 3306
```

**Linux :**
```bash
# Vérifier UFW
sudo ufw status

# Tester le port
telnet 192.168.1.50 3306
# ou
nc -zv 192.168.1.50 3306
```

**Solutions :**
- Relire l'Étape 3 et recréer la règle pare-feu
- Désactiver **temporairement** le pare-feu pour tester (puis le réactiver)

### Problème 4 : Connexion depuis Internet (pas le réseau local)

⚠️ **Cela ne fonctionnera PAS par défaut** car les adresses `192.168.x.x` sont privées.

**Pour permettre l'accès depuis Internet (avancé) :**
1. Configurer le **port forwarding** sur votre routeur (box internet)
2. Utiliser un **VPN** (plus sécurisé)
3. Utiliser un **tunnel SSH** (recommandé pour les tests)

### Problème 5 : "Host is not allowed to connect"

**Cause :** MariaDB refuse la connexion depuis cette IP.

**Solution :**
```sql
-- Vérifier les utilisateurs et leurs hôtes autorisés
SELECT User, Host FROM mysql.user;

-- Si votre utilisateur n'a pas le bon Host, le recréer
CREATE USER 'user_reseau'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user_reseau'@'%';
FLUSH PRIVILEGES;
```

---

## 🔒 Sécurité et bonnes pratiques

### ✅ Recommandations

1. **Utiliser des mots de passe forts**
   ```sql
   -- ✅ BON
   IDENTIFIED BY 'Xk9$mP2#qL8@vN5&wR7!';

   -- ❌ MAUVAIS
   IDENTIFIED BY 'password123';
   ```

2. **Limiter l'accès par sous-réseau**
   ```sql
   -- ✅ MEILLEUR (limite au réseau local)
   CREATE USER 'user'@'192.168.1.%' IDENTIFIED BY 'password';

   -- ⚠️ MOINS SÛRE (accès depuis partout)
   CREATE USER 'user'@'%' IDENTIFIED BY 'password';
   ```

3. **Principe du moindre privilège**
   ```sql
   -- ❌ TROP de privilèges
   GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';

   -- ✅ Privilèges adaptés
   GRANT SELECT, INSERT, UPDATE ON ma_base.* TO 'user'@'%';
   ```

4. **Ne pas exposer root sur le réseau**
   ```sql
   -- ❌ DANGEREUX
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';

   -- ✅ Créer un admin dédié
   CREATE USER 'admin_reseau'@'192.168.1.%' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON *.* TO 'admin_reseau'@'192.168.1.%';
   ```

5. **Changer le port par défaut (optionnel)**
   ```yaml
   # Dans docker-compose.yml
   ports:
     - "3307:3306"  # Port externe 3307 au lieu de 3306
   ```

### ⚠️ Risques à connaître

- **Réseau non sécurisé** : Sur un WiFi public, vos données peuvent être interceptées
- **Mots de passe faibles** : Faciles à deviner par force brute
- **Accès trop permissif** : `@'%'` autorise le monde entier (si exposé sur Internet)

### 🛡️ Solutions de sécurisation avancées

1. **VPN** : Créer un réseau privé virtuel
2. **SSH Tunnel** : Chiffrer la connexion via SSH
3. **SSL/TLS** : Activer le chiffrement dans MariaDB
4. **Fail2ban** : Bloquer les tentatives de connexion répétées

---

## 📊 Récapitulatif des configurations

### Configuration minimale (pour tester)

```yaml
# docker-compose.yml
version: '3.8'
services:
  mariadb:
    image: mariadb:10.11
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
  mariadb_data:
```

```sql
-- SQL
CREATE USER 'test'@'%' IDENTIFIED BY 'test_password';
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
FLUSH PRIVILEGES;
```

```bash
# Pare-feu (Windows)
New-NetFirewallRule -DisplayName "MariaDB" -Direction Inbound -Protocol TCP -LocalPort 3306 -Action Allow -Profile Private

# Pare-feu (Linux)
sudo ufw allow 3306/tcp
```

### Configuration sécurisée (pour production)

```yaml
# docker-compose.yml
version: '3.8'
services:
  mariadb:
    image: mariadb:10.11
    ports:
      - "3307:3306"  # Port non-standard
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}  # Depuis fichier .env
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf
volumes:
  mariadb_data:
```

```sql
-- SQL
CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'Xk9$mP2#qL8@vN5!';
GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'app_user'@'192.168.1.%';
FLUSH PRIVILEGES;
```

---

## ✅ Checklist de vérification

Avant de contacter le support, vérifiez :

- [ ] Le conteneur MariaDB est bien démarré (`docker ps`)
- [ ] Le port 3306 est exposé dans `docker-compose.yml`
- [ ] Vous utilisez la bonne IP (celle du réseau local, pas Docker)
- [ ] Le pare-feu autorise le port 3306
- [ ] L'utilisateur existe avec le bon Host (`@'%'` ou `@'192.168.1.%'`)
- [ ] Le mot de passe est correct
- [ ] Les deux ordinateurs sont sur le même réseau
- [ ] Vous utilisez un client SQL compatible (DBeaver, HeidiSQL, etc.)

---

## 🚀 Prochaines étapes

Félicitations ! Vous maîtrisez maintenant l'accès réseau à MariaDB. Vous pouvez explorer :

- **[Annexe D - Sécurité](../annexes/D-securite-bonnes-pratiques.md)** - Sécuriser davantage votre installation
- **[Annexe E - Dépannage](../annexes/E-depannage.md)** - Résoudre d'autres problèmes courants
- **[Cas pratiques](../cas-pratiques/)** - Mettre en place des architectures complètes

---

## 📚 Ressources complémentaires

- [Documentation MariaDB - Connexions réseau](https://mariadb.com/kb/en/configuring-mariadb-for-remote-client-access/)
- [Documentation Docker - Port mapping](https://docs.docker.com/config/containers/container-networking/#published-ports)
- [Guide des pare-feu Windows](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-firewall/)

---

## ❓ FAQ - Questions fréquentes

**Q : Puis-je me connecter depuis Internet (pas mon réseau local) ?**
R : Par défaut non, car votre IP `192.168.x.x` est privée. Il faudrait configurer le port forwarding sur votre routeur (déconseillé pour des raisons de sécurité). Privilégiez un VPN ou un tunnel SSH.

**Q : Pourquoi utiliser 192.168.1.50 et pas 172.18.0.10 (l'IP Docker) ?**
R : L'IP Docker (`172.18.0.10`) n'est accessible que par d'autres conteneurs sur le même réseau Docker. Pour accéder depuis le réseau physique, utilisez l'IP de la machine hôte (`192.168.x.x`).

**Q : Est-ce que l'IP fixe Docker (fiche 1.3) aide pour l'accès réseau ?**
R : Non, l'IP fixe Docker concerne uniquement la communication entre conteneurs. L'accès réseau se fait toujours via l'IP de la machine hôte + port mapping.

**Q : Puis-je utiliser plusieurs ports (ex: 3306 ET 3307) ?**
R : Oui, vous pouvez mapper plusieurs ports :
```yaml
ports:
  - "3306:3306"
  - "3307:3306"  # Même conteneur accessible sur deux ports
```

**Q : Mon collègue a une erreur "Unknown database", pourquoi ?**
R : L'utilisateur existe mais n'a peut-être pas de permissions sur la base spécifiée. Vérifiez avec :
```sql
SHOW GRANTS FOR 'user'@'%';
GRANT ALL PRIVILEGES ON la_base.* TO 'user'@'%';
```

**Q : Comment voir qui est connecté actuellement ?**
R :
```sql
SHOW PROCESSLIST;
-- Affiche tous les clients connectés avec leur IP
```

**Q : Le port 3306 est déjà utilisé sur ma machine, que faire ?**
R : Changez le port externe :
```yaml
ports:
  - "3307:3306"  # Accessible via 192.168.1.50:3307
```

**Q : Est-ce que Docker Desktop (Windows/Mac) change quelque chose ?**
R : Non, le principe reste le même. Docker Desktop crée une VM interne mais expose les ports sur l'IP de votre machine physique.

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
