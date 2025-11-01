# 10.3 - DynamoDB Local avec Admin UI

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Jusqu'à présent, nous avons configuré DynamoDB Local, mais comment **visualiser** vos tables et vos données ? Comment créer une table sans écrire du code ?

**Le problème :** DynamoDB Local n'a pas d'interface graphique native. L'interface `/shell` est très basique et peu pratique pour le développement.

**La solution :** **dynamodb-admin**, une interface web moderne et intuitive qui vous permet de :
- 📊 **Visualiser** toutes vos tables
- ✏️ **Créer** de nouvelles tables facilement
- 🔍 **Explorer** les données (scan, query)
- ➕ **Ajouter** des éléments manuellement
- ✂️ **Modifier et supprimer** des données
- 📈 **Voir les statistiques** des tables

**Ce que vous allez apprendre :**
- Installer dynamodb-admin avec Docker Compose
- Connecter l'interface à DynamoDB Local
- Utiliser l'interface graphique pour gérer vos tables
- Configuration avancée avec plusieurs environnements

---

## 🎯 Prérequis

✅ Avoir lu la [fiche 10.1 - Configuration basique](01-config-basique-docker-compose.md)
✅ Docker et Docker Compose installés
✅ Comprendre les bases de DynamoDB (tables, items)
✅ 10 minutes de votre temps ⏱️

---

## 🖥️ Présentation de dynamodb-admin

**dynamodb-admin** est un outil open source créé par la communauté :
- 🆓 **Gratuit** et open source
- 🌐 **Interface web** accessible via navigateur
- 🚀 **Léger** (conteneur Node.js)
- 🔧 **Compatible** avec DynamoDB Local et AWS DynamoDB

**Aperçu visuel :**

```
┌─────────────────────────────────────────────┐
│         Interface dynamodb-admin            │
│  http://localhost:8001                      │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │ Tables  │  Items  │  Query  │ JSON  │   │
│  ├─────────────────────────────────────┤   │
│  │ users       │ 42 items │ Create     │   │
│  │ products    │ 15 items │ Delete     │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
                    ↓
          Connexion HTTP
                    ↓
┌─────────────────────────────────────────────┐
│         DynamoDB Local                      │
│  Port 8000                                  │
└─────────────────────────────────────────────┘
```

---

## 🚀 Étape 1 : Configuration basique avec Admin UI

### A. Créer le dossier du projet

```bash
# Créer et entrer dans le dossier
mkdir dynamodb-with-admin
cd dynamodb-with-admin
```

### B. Créer le fichier docker-compose.yml

Créez un fichier `docker-compose.yml` avec deux services :

```yaml
version: '3.8'

services:
  # Service DynamoDB Local
  dynamodb:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_local
    restart: unless-stopped

    ports:
      - "8000:8000"

    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"

    volumes:
      - dynamodb_data:/data

  # Service Admin UI
  dynamodb-admin:
    image: aaronshaf/dynamodb-admin:latest
    container_name: dynamodb_admin
    restart: unless-stopped

    ports:
      # Interface accessible sur http://localhost:8001
      - "8001:8001"

    environment:
      # URL de connexion à DynamoDB Local
      DYNAMO_ENDPOINT: "http://dynamodb:8000"

      # Région AWS (factice pour DynamoDB Local)
      AWS_REGION: "us-east-1"

      # Credentials (factices, mais requis)
      AWS_ACCESS_KEY_ID: "fakeAccessKeyId"
      AWS_SECRET_ACCESS_KEY: "fakeSecretAccessKey"

    # Dépend de DynamoDB (démarre après)
    depends_on:
      - dynamodb

volumes:
  dynamodb_data:
```

---

## 🔍 Comprendre la configuration

### Service dynamodb (identique à la fiche 10.1)

Rien de nouveau ici, c'est la configuration basique de DynamoDB Local.

### Service dynamodb-admin (NOUVEAU)

```yaml
dynamodb-admin:
  image: aaronshaf/dynamodb-admin:latest
```
- **Image officielle** de dynamodb-admin sur Docker Hub
- Maintenue par la communauté

#### Ports

```yaml
ports:
  - "8001:8001"
```
- **8001** (gauche) : Port sur votre PC
- **8001** (droite) : Port dans le conteneur
- Vous accéderez à l'interface via `http://localhost:8001`

**💡 Pourquoi 8001 ?** Pour ne pas entrer en conflit avec DynamoDB qui utilise le port 8000.

#### Variables d'environnement

```yaml
environment:
  DYNAMO_ENDPOINT: "http://dynamodb:8000"
```

**Explication :**
- `DYNAMO_ENDPOINT` : Adresse de DynamoDB Local
- `http://dynamodb:8000` : Utilise le **nom de service** (`dynamodb`) au lieu de `localhost`
  - Docker Compose crée automatiquement un réseau privé
  - Les conteneurs se parlent par leur nom de service

**Pourquoi pas `localhost:8000` ?**
- Dans le conteneur `dynamodb-admin`, `localhost` désigne **lui-même**, pas DynamoDB
- Le nom de service `dynamodb` est résolu automatiquement par Docker vers l'IP du conteneur DynamoDB

```yaml
AWS_REGION: "us-east-1"
AWS_ACCESS_KEY_ID: "fakeAccessKeyId"
AWS_SECRET_ACCESS_KEY: "fakeSecretAccessKey"
```

**Explication :**
- Ces variables sont **requises** par le SDK AWS
- Les valeurs sont **factices** car DynamoDB Local n'a pas d'authentification réelle
- Vous pouvez mettre n'importe quelle valeur (elles ne sont pas vérifiées)

#### Dépendances

```yaml
depends_on:
  - dynamodb
```

**Explication :**
- Assure que DynamoDB démarre **avant** dynamodb-admin
- Évite les erreurs de connexion au démarrage

---

## ▶️ Étape 2 : Démarrer les services

Dans le dossier contenant votre `docker-compose.yml` :

```bash
docker-compose up -d
```

**Sortie attendue :**
```
Creating network "dynamodb-with-admin_default" with the default driver
Creating volume "dynamodb-with-admin_dynamodb_data" with local driver
Creating dynamodb_local ... done
Creating dynamodb_admin ... done
```

✅ Les deux conteneurs sont maintenant en cours d'exécution !

### Vérifier l'état

```bash
docker-compose ps
```

**Sortie attendue :**
```
     Name                   Command              State           Ports
--------------------------------------------------------------------------------
dynamodb_admin   node bin/dynamodb-admin.js     Up      0.0.0.0:8001->8001/tcp
dynamodb_local   java -jar DynamoDBLocal.jar    Up      0.0.0.0:8000->8000/tcp
```

Les deux doivent être **Up** (en cours d'exécution).

---

## 🌐 Étape 3 : Accéder à l'interface Admin

### Ouvrir l'interface

Dans votre navigateur, allez sur :

```
http://localhost:8001
```

**Vous devriez voir :**
- Une interface moderne avec un fond sombre
- Un titre "DynamoDB Admin"
- Une section "Tables" (vide pour l'instant)
- Un bouton "Create Table"

🎉 **Félicitations !** Votre interface Admin UI est fonctionnelle !

---

## 🛠️ Étape 4 : Utiliser l'interface Admin

### A. Créer une table

1. **Cliquez sur "Create Table"** en haut à droite

2. **Remplissez le formulaire :**
   - **Table name** : `users`
   - **Hash attribute name** : `userId` (type String)
   - **Range attribute name** : (laisser vide pour l'instant)

3. **Cliquez sur "Submit"**

✅ Votre première table est créée !

### B. Ajouter des données

1. **Cliquez sur le nom de la table** (`users`) dans la liste

2. **Cliquez sur "Create item"**

3. **Remplissez les champs :**
   ```json
   {
     "userId": "user001",
     "name": "Alice Dupont",
     "email": "alice@example.com",
     "age": 28
   }
   ```

4. **Cliquez sur "Create"**

5. **Répétez** pour ajouter d'autres utilisateurs :
   ```json
   {
     "userId": "user002",
     "name": "Bob Martin",
     "email": "bob@example.com",
     "age": 35
   }
   ```

### C. Visualiser les données

Dans la vue de la table `users`, vous verrez :
- La liste de tous les items (utilisateurs)
- Chaque attribut dans une colonne
- Le nombre total d'items

### D. Modifier un item

1. **Cliquez sur un item** dans la liste
2. **Modifiez** les valeurs (par exemple, changez l'âge)
3. **Cliquez sur "Save"**

### E. Supprimer un item

1. **Cochez** la case à gauche d'un item
2. **Cliquez sur "Delete"**
3. **Confirmez** la suppression

### F. Scanner une table

**Scan** = Lire tous les items d'une table

1. Allez dans la vue de la table
2. Par défaut, un scan est effectué automatiquement
3. Vous voyez tous les items

### G. Query (recherche)

**Query** = Rechercher des items spécifiques

1. Cliquez sur l'onglet **"Query"**
2. Entrez une valeur pour la clé primaire :
   - **userId** (Hash Key) : `user001`
3. Cliquez sur **"Execute"**

Seul l'item avec `userId = user001` s'affiche.

### H. Voir le JSON brut

1. Cliquez sur l'onglet **"JSON"**
2. Vous voyez la structure DynamoDB complète :
   ```json
   {
     "userId": {"S": "user001"},
     "name": {"S": "Alice Dupont"},
     "email": {"S": "alice@example.com"},
     "age": {"N": "28"}
   }
   ```

**Note :** DynamoDB stocke les types explicitement :
- `{"S": "..."}` = String
- `{"N": "..."}` = Number
- `{"BOOL": true}` = Boolean
- etc.

### I. Supprimer une table

1. **Cliquez sur "Delete Table"** en haut à droite
2. **Confirmez** la suppression

⚠️ **Attention :** Cette action est irréversible (toutes les données sont perdues).

---

## 🔧 Étape 5 : Configuration avancée avec IP fixe

Si vous avez besoin d'IP fixes (par exemple pour des microservices), voici la configuration :

### A. Créer le réseau Docker

```bash
docker network create --subnet=172.18.0.0/16 dynamodb_net
```

### B. Modifier le docker-compose.yml

```yaml
version: '3.8'

services:
  dynamodb:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_local
    restart: unless-stopped
    ports:
      - "8000:8000"
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"
    volumes:
      - dynamodb_data:/data
    networks:
      dynamodb_net:
        ipv4_address: 172.18.0.10

  dynamodb-admin:
    image: aaronshaf/dynamodb-admin:latest
    container_name: dynamodb_admin
    restart: unless-stopped
    ports:
      - "8001:8001"
    environment:
      # Utiliser l'IP fixe au lieu du nom de service
      DYNAMO_ENDPOINT: "http://172.18.0.10:8000"
      AWS_REGION: "us-east-1"
      AWS_ACCESS_KEY_ID: "fakeAccessKeyId"
      AWS_SECRET_ACCESS_KEY: "fakeSecretAccessKey"
    depends_on:
      - dynamodb
    networks:
      dynamodb_net:
        ipv4_address: 172.18.0.20

volumes:
  dynamodb_data:

networks:
  dynamodb_net:
    external: true
```

**Changements :**
- `DYNAMO_ENDPOINT: "http://172.18.0.10:8000"` utilise l'IP fixe
- Les deux services ont des IP fixes dans le même réseau

---

## 🌍 Configuration multi-environnements

Vous pouvez configurer plusieurs environnements DynamoDB simultanément :

```yaml
version: '3.8'

services:
  # Environnement de DÉVELOPPEMENT
  dynamodb-dev:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_dev
    ports:
      - "8000:8000"
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"
    volumes:
      - dynamodb_dev_data:/data

  admin-dev:
    image: aaronshaf/dynamodb-admin:latest
    container_name: admin_dev
    ports:
      - "8001:8001"
    environment:
      DYNAMO_ENDPOINT: "http://dynamodb-dev:8000"
      AWS_REGION: "us-east-1"
      AWS_ACCESS_KEY_ID: "dev"
      AWS_SECRET_ACCESS_KEY: "dev"
    depends_on:
      - dynamodb-dev

  # Environnement de TEST
  dynamodb-test:
    image: amazon/dynamodb-local:latest
    container_name: dynamodb_test
    ports:
      - "8002:8000"  # Port différent !
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath /data"
    volumes:
      - dynamodb_test_data:/data

  admin-test:
    image: aaronshaf/dynamodb-admin:latest
    container_name: admin_test
    ports:
      - "8003:8001"  # Port différent !
    environment:
      DYNAMO_ENDPOINT: "http://dynamodb-test:8000"
      AWS_REGION: "us-east-1"
      AWS_ACCESS_KEY_ID: "test"
      AWS_SECRET_ACCESS_KEY: "test"
    depends_on:
      - dynamodb-test

volumes:
  dynamodb_dev_data:
  dynamodb_test_data:
```

**Accès :**
- **Dev** : DynamoDB sur `localhost:8000`, Admin sur `localhost:8001`
- **Test** : DynamoDB sur `localhost:8002`, Admin sur `localhost:8003`

---

## 📊 Fonctionnalités avancées de l'Admin UI

### 1. Filtrer les résultats

Dans la vue table, utilisez la barre de recherche en haut pour filtrer :
```
name contains "Alice"
```

### 2. Pagination

Si votre table contient beaucoup d'items :
- Les résultats sont paginés automatiquement
- Utilisez les boutons "Next" et "Previous"

### 3. Export de données

**Note :** L'export direct n'est pas disponible dans dynamodb-admin.

**Alternative :** Utilisez AWS CLI (voir [fiche 10.4](04-utilisation-aws-cli.md)) :
```bash
aws dynamodb scan --table-name users --endpoint-url http://localhost:8000
```

### 4. Capacités de lecture/écriture

Dans DynamoDB Local, ces paramètres sont ignorés (ils sont pertinents uniquement sur AWS).

---

## 🛠️ Commandes utiles

### Voir les logs d'Admin UI

```bash
docker-compose logs -f dynamodb-admin
```

**Utile pour :**
- Déboguer les problèmes de connexion
- Voir les erreurs

### Redémarrer uniquement Admin UI

```bash
docker-compose restart dynamodb-admin
```

### Reconstruire l'image (après mise à jour)

```bash
docker-compose pull dynamodb-admin
docker-compose up -d dynamodb-admin
```

---

## 🧹 Étape 6 : Suppression complète

### Arrêter et supprimer les conteneurs

```bash
docker-compose down
```

### Supprimer les volumes (ATTENTION : supprime les données)

```bash
docker volume rm dynamodb-with-admin_dynamodb_data
```

ou

```bash
docker volume prune
```

---

## 🔒 Sécurité et bonnes pratiques

### ⚠️ Ne PAS exposer Admin UI en production

**dynamodb-admin est fait pour le DÉVELOPPEMENT uniquement.**

**Risques :**
- ❌ Pas d'authentification
- ❌ Accès complet en lecture/écriture
- ❌ Suppression de tables possible

**En production :**
- Utilisez la console AWS officielle
- Ou créez votre propre interface avec authentification

### 🔐 Limiter l'accès au réseau

Si vous voulez sécuriser l'accès à Admin UI :

```yaml
dynamodb-admin:
  # ... autres configs
  ports:
    - "127.0.0.1:8001:8001"  # Accessible uniquement sur localhost
```

### 📝 Utiliser des fichiers .env

Créez un fichier `.env` :

```bash
# .env
DYNAMO_ENDPOINT=http://dynamodb:8000
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=local_dev
AWS_SECRET_ACCESS_KEY=local_dev_secret
ADMIN_PORT=8001
```

Puis dans `docker-compose.yml` :

```yaml
dynamodb-admin:
  # ...
  ports:
    - "${ADMIN_PORT}:8001"
  environment:
    DYNAMO_ENDPOINT: "${DYNAMO_ENDPOINT}"
    AWS_REGION: "${AWS_REGION}"
    AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
    AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

**Ajoutez `.env` à votre `.gitignore` :**
```bash
echo ".env" >> .gitignore
```

---

## ❓ Problèmes courants

### Admin UI ne se connecte pas à DynamoDB

**Symptôme :** Page blanche ou erreur "Cannot connect"

**Solutions :**

1. **Vérifier que DynamoDB tourne :**
   ```bash
   docker-compose ps
   curl http://localhost:8000/shell
   ```

2. **Vérifier les logs :**
   ```bash
   docker-compose logs dynamodb-admin
   ```

3. **Vérifier DYNAMO_ENDPOINT :**
   - Doit être `http://dynamodb:8000` (nom de service)
   - **PAS** `http://localhost:8000`

4. **Redémarrer Admin UI :**
   ```bash
   docker-compose restart dynamodb-admin
   ```

### Erreur "Network dynamodb_net not found" (avec IP fixe)

**Cause :** Le réseau n'a pas été créé.

**Solution :**
```bash
docker network create --subnet=172.18.0.0/16 dynamodb_net
docker-compose up -d
```

### Interface Admin lente

**Causes possibles :**
- Table avec beaucoup d'items (scan complet)
- Manque de ressources Docker

**Solutions :**
- Limiter le nombre d'items affichés (pagination)
- Augmenter la RAM allouée à Docker Desktop

### Impossible de supprimer une table

**Symptôme :** Erreur lors de la suppression

**Solution :**
1. Vérifier les logs : `docker-compose logs dynamodb-admin`
2. Essayer via AWS CLI (voir [fiche 10.4](04-utilisation-aws-cli.md))
3. Redémarrer DynamoDB : `docker-compose restart dynamodb`

---

## 🎓 Concepts clés à retenir

### 🌐 Communication entre conteneurs

Dans Docker Compose :
- Les conteneurs peuvent se parler par leur **nom de service**
- `http://dynamodb:8000` fonctionne car `dynamodb` est résolu automatiquement
- Pas besoin de `localhost` entre conteneurs

### 🔌 depends_on

```yaml
depends_on:
  - dynamodb
```

**Effet :**
- Démarre DynamoDB **avant** Admin UI
- **Ne garantit PAS** que DynamoDB est prêt (juste démarré)

**Pour une vraie santé :** Utilisez `healthcheck` (configuration avancée).

### 📊 Différence Scan vs Query

| Opération | Description | Performance | Coût (AWS) |
|-----------|-------------|-------------|------------|
| **Scan** | Lit toute la table | ❌ Lent | 💰💰💰 Cher |
| **Query** | Recherche sur clé primaire | ✅ Rapide | 💰 Économique |

**Recommandation :** Utilisez **Query** en production, **Scan** uniquement pour le développement/debug.

---

## 💡 Astuces pro

### 1. Raccourcis clavier dans l'Admin UI

- `Ctrl + K` (ou `Cmd + K` sur Mac) : Recherche rapide
- `Ctrl + Enter` : Soumettre un formulaire

### 2. Copier/coller de JSON

Vous pouvez copier un JSON complet et le coller dans "Create item" :

```json
{
  "userId": "user003",
  "name": "Charlie Brown",
  "email": "charlie@example.com",
  "age": 42,
  "premium": true,
  "tags": ["developer", "senior"]
}
```

### 3. Créer des données de test rapidement

Utilisez un script avec AWS CLI (voir [fiche 10.4](04-utilisation-aws-cli.md)) pour insérer 100+ items d'un coup.

### 4. Thème sombre/clair

dynamodb-admin utilise un thème sombre par défaut. Pour changer (si supporté dans votre version), vérifiez les paramètres en haut à droite.

---

## 🔄 Alternatives à dynamodb-admin

| Outil | Type | Avantages | Inconvénients |
|-------|------|-----------|---------------|
| **dynamodb-admin** | Web UI | ✅ Simple, léger | ❌ Fonctionnalités limitées |
| **AWS Console** | Web (AWS) | ✅ Complet | ❌ Nécessite compte AWS |
| **NoSQL Workbench** | Desktop | ✅ Officiel AWS | ❌ Lourd, installation requise |
| **AWS CLI** | Terminal | ✅ Automatisation | ❌ Pas d'interface graphique |
| **TablePlus** | Desktop | ✅ Multi-BDD | 💰 Payant (version Pro) |

**Recommandation pour le dev :** dynamodb-admin (gratuit, simple, suffisant).

---

## 🎯 Cas d'usage pratiques

### Développement local

```yaml
# Environnement complet de dev
services:
  dynamodb:
    # ... config DynamoDB

  dynamodb-admin:
    # ... config Admin UI

  backend:
    # Votre API qui utilise DynamoDB
    environment:
      DYNAMODB_ENDPOINT: "http://dynamodb:8000"
```

### Tests d'intégration

Utilisez Admin UI pour :
- Vérifier les données après un test
- Préparer des données de test manuellement
- Déboguer les requêtes

### Prototypage

Créez rapidement des tables et testez vos requêtes avant de coder.

---

## 🎯 Prochaines étapes

Maintenant que vous maîtrisez l'Admin UI, explorez :

1. **[Utilisation avec AWS CLI](04-utilisation-aws-cli.md)** - Automatiser la création de tables et l'insertion de données
2. **[Cas pratique - Environnement dev complet](../cas-pratiques/04-env-dev-complet.md)** - Multi-BDD avec Admin UIs
3. **[Annexe F - Outils de gestion](../annexes/F-outils-gestion.md)** - Comparaison des outils BDD

---

## 📚 Ressources complémentaires

- 🐙 [dynamodb-admin sur GitHub](https://github.com/aaronshaf/dynamodb-admin)
- 🐳 [dynamodb-admin sur Docker Hub](https://hub.docker.com/r/aaronshaf/dynamodb-admin)
- 📖 [Documentation DynamoDB](https://docs.aws.amazon.com/dynamodb/)
- 🔧 [NoSQL Workbench (officiel AWS)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html)

---

## ✅ Checklist de fin

Avant de passer à la fiche suivante, vérifiez que vous savez :

- [ ] Configurer dynamodb-admin dans docker-compose.yml
- [ ] Accéder à l'interface sur http://localhost:8001
- [ ] Créer une table via l'Admin UI
- [ ] Ajouter, modifier et supprimer des items
- [ ] Comprendre la différence entre Scan et Query
- [ ] Utiliser DYNAMO_ENDPOINT avec le nom de service
- [ ] Dépanner les problèmes de connexion courants

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
⬅️ Fiche précédente : [10.2 - Configuration avec IP fixe](02-config-ip-fixe.md)
➡️ Fiche suivante : [10.4 - Utilisation avec AWS CLI](04-utilisation-aws-cli.md)

---

*Dernière mise à jour : Novembre 2025*
