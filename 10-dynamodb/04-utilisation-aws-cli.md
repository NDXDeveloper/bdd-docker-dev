# 10.4 - Utilisation de DynamoDB Local avec AWS CLI

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

Jusqu'à présent, nous avons utilisé l'interface graphique (Admin UI) pour gérer DynamoDB Local. Mais comment **automatiser** la création de tables ? Comment **scripter** l'insertion de milliers de données de test ? Comment **intégrer** DynamoDB dans vos pipelines de développement ?

**La solution :** **AWS CLI** (Command Line Interface), l'outil en ligne de commande officiel d'Amazon pour interagir avec tous les services AWS, y compris DynamoDB Local.

**Ce que vous allez apprendre :**
- 🛠️ Installer et configurer AWS CLI
- 📊 Créer et gérer des tables
- ➕ Insérer, lire, modifier et supprimer des données
- 🔍 Effectuer des requêtes (Query) et des scans
- 📝 Automatiser avec des scripts
- 🚀 Importer des données depuis JSON/CSV

**Pourquoi utiliser AWS CLI ?**
- ✅ **Automatisation** : Scripts de création de base de données
- ✅ **Reproductibilité** : Mêmes commandes = mêmes résultats
- ✅ **CI/CD** : Intégration dans vos pipelines
- ✅ **Puissance** : Toutes les fonctionnalités DynamoDB disponibles
- ✅ **Transfert** : Code réutilisable pour DynamoDB AWS

---

## 🎯 Prérequis

✅ Avoir lu la [fiche 10.1 - Configuration basique](01-config-basique-docker-compose.md)
✅ DynamoDB Local en cours d'exécution
✅ Notions de base en ligne de commande
✅ 15 minutes de votre temps ⏱️

---

## 🔧 Étape 1 : Installation de AWS CLI

AWS CLI est disponible sur Windows, macOS et Linux.

### Installation sur Windows

**Méthode 1 : Installeur MSI (recommandé)**

1. Téléchargez l'installeur : [AWS CLI pour Windows](https://awscli.amazonaws.com/AWSCLIV2.msi)
2. Exécutez le fichier `.msi`
3. Suivez l'assistant d'installation

**Méthode 2 : Via Chocolatey**

```powershell
choco install awscli
```

### Installation sur macOS

**Méthode 1 : Installeur PKG (recommandé)**

1. Téléchargez l'installeur : [AWS CLI pour macOS](https://awscli.amazonaws.com/AWSCLIV2.pkg)
2. Exécutez le fichier `.pkg`
3. Suivez l'assistant d'installation

**Méthode 2 : Via Homebrew**

```bash
brew install awscli
```

### Installation sur Linux

**Ubuntu/Debian :**

```bash
sudo apt update
sudo apt install awscli -y
```

**Fedora/CentOS/RHEL :**

```bash
sudo yum install awscli -y
```

**Installation universelle (toutes distributions) :**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Vérification de l'installation

```bash
aws --version
```

**Sortie attendue :**
```
aws-cli/2.x.x Python/3.x.x ...
```

✅ AWS CLI est installé !

---

## ⚙️ Étape 2 : Configuration de AWS CLI pour DynamoDB Local

### A. Configuration des credentials (factices)

Même si DynamoDB Local n'a pas d'authentification réelle, AWS CLI requiert des credentials. Nous allons configurer des valeurs factices.

```bash
aws configure
```

**Répondez aux questions :**

```
AWS Access Key ID [None]: fakeAccessKeyId
AWS Secret Access Key [None]: fakeSecretAccessKey
Default region name [None]: us-east-1
Default output format [None]: json
```

**Explication :**
- **Access Key ID** et **Secret Access Key** : Valeurs factices (non vérifiées par DynamoDB Local)
- **Region** : `us-east-1` (convention, mais toute région fonctionne en local)
- **Output format** : `json` (format de sortie des commandes)

### B. Vérifier la configuration

```bash
cat ~/.aws/credentials
```

**Sortie :**
```ini
[default]
aws_access_key_id = fakeAccessKeyId
aws_secret_access_key = fakeSecretAccessKey
```

```bash
cat ~/.aws/config
```

**Sortie :**
```ini
[default]
region = us-east-1
output = json
```

### C. Créer un alias pour DynamoDB Local

Pour éviter de taper `--endpoint-url http://localhost:8000` à chaque fois, créez un alias.

**Bash/Zsh (Linux/macOS) :**

Ajoutez dans `~/.bashrc` ou `~/.zshrc` :

```bash
alias aws-local='aws --endpoint-url http://localhost:8000'
```

Puis rechargez :
```bash
source ~/.bashrc  # ou source ~/.zshrc
```

**PowerShell (Windows) :**

Ajoutez dans votre profil PowerShell (`$PROFILE`) :

```powershell
function aws-local {
    aws --endpoint-url http://localhost:8000 @args
}
```

**Utilisation :**
```bash
# Au lieu de :
aws dynamodb list-tables --endpoint-url http://localhost:8000

# Utilisez :
aws-local dynamodb list-tables
```

---

## 📊 Étape 3 : Créer votre première table

### A. Créer une table simple

Créons une table `users` avec :
- **Clé primaire (Hash Key)** : `userId` (String)

```bash
aws dynamodb create-table \
    --table-name users \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --endpoint-url http://localhost:8000
```

**Explication de la commande :**

- `create-table` : Commande de création
- `--table-name users` : Nom de la table
- `--attribute-definitions` : Définit les attributs utilisés comme clés
  - `AttributeName=userId` : Nom de l'attribut
  - `AttributeType=S` : Type String (S = String, N = Number, B = Binary)
- `--key-schema` : Définit la structure de la clé primaire
  - `AttributeName=userId` : Attribut à utiliser
  - `KeyType=HASH` : Type de clé (HASH = clé primaire simple)
- `--billing-mode PAY_PER_REQUEST` : Mode de facturation (ignoré en local, mais requis)
- `--endpoint-url` : Pointe vers DynamoDB Local

**Sortie attendue :**
```json
{
    "TableDescription": {
        "TableName": "users",
        "TableStatus": "ACTIVE",
        "KeySchema": [
            {
                "AttributeName": "userId",
                "KeyType": "HASH"
            }
        ],
        ...
    }
}
```

✅ Table créée !

### B. Créer une table avec clé composée

Créons une table `orders` avec :
- **Hash Key** : `customerId` (String)
- **Range Key** : `orderDate` (String)

```bash
aws dynamodb create-table \
    --table-name orders \
    --attribute-definitions \
        AttributeName=customerId,AttributeType=S \
        AttributeName=orderDate,AttributeType=S \
    --key-schema \
        AttributeName=customerId,KeyType=HASH \
        AttributeName=orderDate,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --endpoint-url http://localhost:8000
```

**Différence :**
- Deux attributs définis
- `KeyType=RANGE` pour la clé de tri (sort key)

### C. Lister les tables

```bash
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

**Sortie :**
```json
{
    "TableNames": [
        "orders",
        "users"
    ]
}
```

---

## ➕ Étape 4 : Insérer des données (Put Item)

### A. Insérer un item simple

```bash
aws dynamodb put-item \
    --table-name users \
    --item '{
        "userId": {"S": "user001"},
        "name": {"S": "Alice Dupont"},
        "email": {"S": "alice@example.com"},
        "age": {"N": "28"}
    }' \
    --endpoint-url http://localhost:8000
```

**Explication de la structure JSON :**

```json
{
    "userId": {"S": "user001"},    // String
    "name": {"S": "Alice Dupont"}, // String
    "email": {"S": "alice@example.com"}, // String
    "age": {"N": "28"}              // Number
}
```

**Types DynamoDB :**
- `S` : String (chaîne de caractères)
- `N` : Number (nombre)
- `BOOL` : Boolean (true/false)
- `NULL` : Null
- `M` : Map (objet)
- `L` : List (tableau)
- `SS` : String Set (ensemble de chaînes)
- `NS` : Number Set (ensemble de nombres)

**Pas de sortie = Succès** (par défaut)

### B. Insérer plusieurs items

**user002 :**
```bash
aws dynamodb put-item \
    --table-name users \
    --item '{
        "userId": {"S": "user002"},
        "name": {"S": "Bob Martin"},
        "email": {"S": "bob@example.com"},
        "age": {"N": "35"},
        "premium": {"BOOL": true}
    }' \
    --endpoint-url http://localhost:8000
```

**user003 avec attributs complexes :**
```bash
aws dynamodb put-item \
    --table-name users \
    --item '{
        "userId": {"S": "user003"},
        "name": {"S": "Charlie Brown"},
        "email": {"S": "charlie@example.com"},
        "age": {"N": "42"},
        "address": {
            "M": {
                "street": {"S": "123 Main St"},
                "city": {"S": "Paris"},
                "zipCode": {"S": "75001"}
            }
        },
        "tags": {
            "L": [
                {"S": "developer"},
                {"S": "senior"}
            ]
        }
    }' \
    --endpoint-url http://localhost:8000
```

**Explication des types complexes :**
- `"M": {...}` : Map (objet imbriqué)
- `"L": [...]` : List (tableau)

### C. Insérer dans une table avec clé composée

```bash
aws dynamodb put-item \
    --table-name orders \
    --item '{
        "customerId": {"S": "user001"},
        "orderDate": {"S": "2025-11-01"},
        "amount": {"N": "149.99"},
        "status": {"S": "shipped"}
    }' \
    --endpoint-url http://localhost:8000
```

---

## 🔍 Étape 5 : Lire des données (Get Item)

### A. Lire un item spécifique

```bash
aws dynamodb get-item \
    --table-name users \
    --key '{
        "userId": {"S": "user001"}
    }' \
    --endpoint-url http://localhost:8000
```

**Sortie :**
```json
{
    "Item": {
        "userId": {"S": "user001"},
        "name": {"S": "Alice Dupont"},
        "email": {"S": "alice@example.com"},
        "age": {"N": "28"}
    }
}
```

### B. Lire avec clé composée

```bash
aws dynamodb get-item \
    --table-name orders \
    --key '{
        "customerId": {"S": "user001"},
        "orderDate": {"S": "2025-11-01"}
    }' \
    --endpoint-url http://localhost:8000
```

**Important :** Avec une clé composée, vous devez fournir **les deux** attributs (hash + range).

### C. Projeter seulement certains attributs

```bash
aws dynamodb get-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --projection-expression "name,email" \
    --endpoint-url http://localhost:8000
```

**Sortie :**
```json
{
    "Item": {
        "name": {"S": "Alice Dupont"},
        "email": {"S": "alice@example.com"}
    }
}
```

Seuls `name` et `email` sont retournés (optimisation).

---

## 📋 Étape 6 : Scanner une table (Scan)

**Scan** = Lire **tous** les items d'une table.

⚠️ **Attention :** Opération coûteuse en production, utilisez avec parcimonie.

### A. Scan basique

```bash
aws dynamodb scan \
    --table-name users \
    --endpoint-url http://localhost:8000
```

**Sortie :**
```json
{
    "Items": [
        {
            "userId": {"S": "user001"},
            "name": {"S": "Alice Dupont"},
            ...
        },
        {
            "userId": {"S": "user002"},
            "name": {"S": "Bob Martin"},
            ...
        },
        ...
    ],
    "Count": 3,
    "ScannedCount": 3
}
```

### B. Scan avec filtre

Filtrer uniquement les utilisateurs premium :

```bash
aws dynamodb scan \
    --table-name users \
    --filter-expression "premium = :val" \
    --expression-attribute-values '{":val": {"BOOL": true}}' \
    --endpoint-url http://localhost:8000
```

**Explication :**
- `--filter-expression` : Condition de filtrage
- `premium = :val` : Syntaxe DynamoDB (`:val` est un placeholder)
- `--expression-attribute-values` : Définit la valeur de `:val`

### C. Scan avec projection

```bash
aws dynamodb scan \
    --table-name users \
    --projection-expression "userId,name" \
    --endpoint-url http://localhost:8000
```

Retourne uniquement `userId` et `name` pour chaque item.

---

## 🎯 Étape 7 : Requêter des données (Query)

**Query** = Rechercher des items basés sur la **clé primaire** (beaucoup plus rapide que Scan).

### A. Query sur une table avec hash key seule

```bash
aws dynamodb query \
    --table-name users \
    --key-condition-expression "userId = :id" \
    --expression-attribute-values '{":id": {"S": "user001"}}' \
    --endpoint-url http://localhost:8000
```

**Sortie :**
```json
{
    "Items": [
        {
            "userId": {"S": "user001"},
            "name": {"S": "Alice Dupont"},
            ...
        }
    ],
    "Count": 1
}
```

### B. Query sur une table avec clé composée

Trouver toutes les commandes d'un client :

```bash
aws dynamodb query \
    --table-name orders \
    --key-condition-expression "customerId = :cid" \
    --expression-attribute-values '{":cid": {"S": "user001"}}' \
    --endpoint-url http://localhost:8000
```

### C. Query avec range key condition

Trouver les commandes d'un client après une certaine date :

```bash
aws dynamodb query \
    --table-name orders \
    --key-condition-expression "customerId = :cid AND orderDate > :date" \
    --expression-attribute-values '{
        ":cid": {"S": "user001"},
        ":date": {"S": "2025-01-01"}
    }' \
    --endpoint-url http://localhost:8000
```

**Opérateurs disponibles pour range key :**
- `=` : Égal
- `<` : Inférieur
- `<=` : Inférieur ou égal
- `>` : Supérieur
- `>=` : Supérieur ou égal
- `BETWEEN :val1 AND :val2` : Entre deux valeurs
- `begins_with(:val)` : Commence par

---

## ✏️ Étape 8 : Mettre à jour des données (Update Item)

### A. Mise à jour simple

Changer l'âge de user001 :

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --update-expression "SET age = :newAge" \
    --expression-attribute-values '{":newAge": {"N": "29"}}' \
    --endpoint-url http://localhost:8000
```

### B. Ajouter un nouvel attribut

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --update-expression "SET premium = :val" \
    --expression-attribute-values '{":val": {"BOOL": true}}' \
    --endpoint-url http://localhost:8000
```

### C. Incrémenter une valeur

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --update-expression "SET age = age + :inc" \
    --expression-attribute-values '{":inc": {"N": "1"}}' \
    --endpoint-url http://localhost:8000
```

L'âge augmente de 1.

### D. Supprimer un attribut

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user002"}}' \
    --update-expression "REMOVE premium" \
    --endpoint-url http://localhost:8000
```

### E. Retourner les valeurs après mise à jour

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --update-expression "SET age = :newAge" \
    --expression-attribute-values '{":newAge": {"N": "30"}}' \
    --return-values ALL_NEW \
    --endpoint-url http://localhost:8000
```

**`--return-values` options :**
- `NONE` : Ne retourne rien (défaut)
- `ALL_OLD` : Valeurs avant mise à jour
- `ALL_NEW` : Valeurs après mise à jour
- `UPDATED_OLD` : Seulement les attributs modifiés (avant)
- `UPDATED_NEW` : Seulement les attributs modifiés (après)

---

## 🗑️ Étape 9 : Supprimer des données (Delete Item)

### A. Supprimer un item

```bash
aws dynamodb delete-item \
    --table-name users \
    --key '{"userId": {"S": "user003"}}' \
    --endpoint-url http://localhost:8000
```

### B. Supprimer avec condition

Supprimer uniquement si l'âge est supérieur à 40 :

```bash
aws dynamodb delete-item \
    --table-name users \
    --key '{"userId": {"S": "user002"}}' \
    --condition-expression "age > :threshold" \
    --expression-attribute-values '{":threshold": {"N": "40"}}' \
    --endpoint-url http://localhost:8000
```

### C. Supprimer et retourner les valeurs

```bash
aws dynamodb delete-item \
    --table-name users \
    --key '{"userId": {"S": "user002"}}' \
    --return-values ALL_OLD \
    --endpoint-url http://localhost:8000
```

Affiche l'item avant sa suppression.

---

## 📦 Étape 10 : Opérations par lot (Batch)

### A. Batch Write (écriture multiple)

Insérer ou supprimer plusieurs items en une seule requête :

```bash
aws dynamodb batch-write-item \
    --request-items '{
        "users": [
            {
                "PutRequest": {
                    "Item": {
                        "userId": {"S": "user004"},
                        "name": {"S": "David Lee"},
                        "age": {"N": "25"}
                    }
                }
            },
            {
                "PutRequest": {
                    "Item": {
                        "userId": {"S": "user005"},
                        "name": {"S": "Emma Wilson"},
                        "age": {"N": "32"}
                    }
                }
            }
        ]
    }' \
    --endpoint-url http://localhost:8000
```

**Limite :** Maximum 25 items par requête.

### B. Batch Get (lecture multiple)

```bash
aws dynamodb batch-get-item \
    --request-items '{
        "users": {
            "Keys": [
                {"userId": {"S": "user001"}},
                {"userId": {"S": "user002"}},
                {"userId": {"S": "user004"}}
            ]
        }
    }' \
    --endpoint-url http://localhost:8000
```

Récupère plusieurs items en une seule requête.

---

## 🗑️ Étape 11 : Gérer les tables

### A. Décrire une table

```bash
aws dynamodb describe-table \
    --table-name users \
    --endpoint-url http://localhost:8000
```

**Informations retournées :**
- Statut de la table
- Schéma des clés
- Nombre d'items
- Taille de la table
- Index (si présents)

### B. Supprimer une table

```bash
aws dynamodb delete-table \
    --table-name users \
    --endpoint-url http://localhost:8000
```

⚠️ **Attention :** Action irréversible, toutes les données sont perdues.

---

## 📝 Étape 12 : Automatisation avec scripts

### A. Script Bash de création d'environnement

Créez un fichier `setup-dynamodb.sh` :

```bash
#!/bin/bash

ENDPOINT="http://localhost:8000"

echo "🚀 Configuration de DynamoDB Local..."

# Créer la table users
echo "📊 Création de la table users..."
aws dynamodb create-table \
    --table-name users \
    --attribute-definitions AttributeName=userId,AttributeType=S \
    --key-schema AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --endpoint-url $ENDPOINT

# Créer la table orders
echo "📊 Création de la table orders..."
aws dynamodb create-table \
    --table-name orders \
    --attribute-definitions \
        AttributeName=customerId,AttributeType=S \
        AttributeName=orderDate,AttributeType=S \
    --key-schema \
        AttributeName=customerId,KeyType=HASH \
        AttributeName=orderDate,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST \
    --endpoint-url $ENDPOINT

# Insérer des données de test
echo "➕ Insertion de données de test..."
aws dynamodb put-item \
    --table-name users \
    --item '{"userId": {"S": "user001"}, "name": {"S": "Alice"}, "age": {"N": "28"}}' \
    --endpoint-url $ENDPOINT

aws dynamodb put-item \
    --table-name users \
    --item '{"userId": {"S": "user002"}, "name": {"S": "Bob"}, "age": {"N": "35"}}' \
    --endpoint-url $ENDPOINT

echo "✅ Configuration terminée !"
```

**Rendre le script exécutable :**
```bash
chmod +x setup-dynamodb.sh
```

**Exécuter :**
```bash
./setup-dynamodb.sh
```

### B. Script Python pour import CSV

Créez un fichier `import-csv.py` :

```python
import boto3
import csv

# Configuration
dynamodb = boto3.client(
    'dynamodb',
    endpoint_url='http://localhost:8000',
    region_name='us-east-1',
    aws_access_key_id='fakeKey',
    aws_secret_access_key='fakeSecret'
)

# Lire le fichier CSV
with open('users.csv', 'r') as file:
    reader = csv.DictReader(file)

    for row in reader:
        # Insérer chaque ligne dans DynamoDB
        dynamodb.put_item(
            TableName='users',
            Item={
                'userId': {'S': row['userId']},
                'name': {'S': row['name']},
                'email': {'S': row['email']},
                'age': {'N': row['age']}
            }
        )
        print(f"✅ Imported: {row['name']}")

print("🎉 Import terminé !")
```

**Fichier CSV exemple (`users.csv`) :**
```csv
userId,name,email,age
user001,Alice Dupont,alice@example.com,28
user002,Bob Martin,bob@example.com,35
user003,Charlie Brown,charlie@example.com,42
```

**Exécuter :**
```bash
pip install boto3
python import-csv.py
```

### C. Script de nettoyage

Créez un fichier `cleanup-dynamodb.sh` :

```bash
#!/bin/bash

ENDPOINT="http://localhost:8000"

echo "🧹 Nettoyage de DynamoDB Local..."

# Lister toutes les tables
TABLES=$(aws dynamodb list-tables --endpoint-url $ENDPOINT --output text --query 'TableNames[]')

# Supprimer chaque table
for TABLE in $TABLES; do
    echo "🗑️  Suppression de la table: $TABLE"
    aws dynamodb delete-table --table-name $TABLE --endpoint-url $ENDPOINT
done

echo "✅ Nettoyage terminé !"
```

---

## 📊 Étape 13 : Export et import de données

### A. Export d'une table en JSON

```bash
aws dynamodb scan \
    --table-name users \
    --endpoint-url http://localhost:8000 \
    --output json > users-export.json
```

### B. Format simplifié avec jq

Si vous avez `jq` installé :

```bash
aws dynamodb scan \
    --table-name users \
    --endpoint-url http://localhost:8000 \
    | jq '.Items' > users-simple.json
```

### C. Import depuis JSON

Utilisez un script Python ou Bash pour lire le JSON et insérer les items avec `put-item`.

---

## 🎓 Concepts clés à retenir

### 🔑 Types de clés

| Type | Description | Obligatoire |
|------|-------------|-------------|
| **Hash Key** | Clé de partition | ✅ Oui |
| **Range Key** | Clé de tri | ❌ Optionnel |

**Combinaison :**
- Hash Key seule : Identifie **uniquement** un item
- Hash + Range : Identifie un item dans une partition

### 📊 Scan vs Query

| Critère | Scan | Query |
|---------|------|-------|
| **Lecture** | Toute la table | Seulement les items correspondants |
| **Performance** | ❌ Lent | ✅ Rapide |
| **Coût (AWS)** | 💰💰💰 Élevé | 💰 Faible |
| **Usage** | Dev/Debug | Production |

**Recommandation :** Privilégiez toujours Query quand possible.

### 🧩 Expressions DynamoDB

**Update Expression :**
- `SET` : Définir ou modifier
- `REMOVE` : Supprimer un attribut
- `ADD` : Incrémenter/ajouter à un set
- `DELETE` : Supprimer d'un set

**Condition Expression :**
- Utilisé pour les mises à jour/suppressions conditionnelles
- Exemple : `age > :threshold`

**Filter Expression :**
- Utilisé avec Scan/Query pour filtrer les résultats
- ⚠️ Le filtrage se fait **après** la lecture (pas d'optimisation)

---

## ❓ Problèmes courants

### Erreur "Unable to locate credentials"

**Cause :** AWS CLI n'a pas trouvé de credentials.

**Solution :**
```bash
aws configure
# Entrez des valeurs factices
```

### Erreur "Could not connect to the endpoint URL"

**Cause :** DynamoDB Local n'est pas démarré ou l'endpoint est incorrect.

**Solutions :**
1. Vérifier que DynamoDB tourne :
   ```bash
   docker ps | grep dynamodb
   curl http://localhost:8000/shell
   ```

2. Vérifier l'endpoint dans la commande :
   ```bash
   --endpoint-url http://localhost:8000
   ```

### Erreur "ValidationException: One or more parameter values were invalid"

**Cause :** Structure JSON incorrecte ou types mal définis.

**Solution :** Vérifiez la syntaxe :
```json
// ❌ Incorrect
{"age": 28}

// ✅ Correct
{"age": {"N": "28"}}
```

### Erreur "ResourceNotFoundException: Requested resource not found"

**Cause :** La table n'existe pas.

**Solution :**
```bash
# Vérifier les tables existantes
aws dynamodb list-tables --endpoint-url http://localhost:8000

# Créer la table si nécessaire
aws dynamodb create-table ...
```

---

## 💡 Bonnes pratiques

### 1. Utilisez des variables d'environnement

Créez un fichier `.env` :
```bash
export DYNAMODB_ENDPOINT="http://localhost:8000"
export AWS_REGION="us-east-1"
```

Chargez-le :
```bash
source .env
```

Puis utilisez dans vos commandes :
```bash
aws dynamodb list-tables --endpoint-url $DYNAMODB_ENDPOINT
```

### 2. Créez des scripts réutilisables

Organisez vos scripts dans un dossier :
```
project/
├── scripts/
│   ├── setup.sh
│   ├── seed-data.sh
│   ├── cleanup.sh
│   └── backup.sh
├── data/
│   ├── users.json
│   └── orders.json
└── docker-compose.yml
```

### 3. Versionnez vos schémas de tables

Créez des fichiers JSON pour vos définitions de tables :

**tables/users-table.json :**
```json
{
  "TableName": "users",
  "AttributeDefinitions": [
    {
      "AttributeName": "userId",
      "AttributeType": "S"
    }
  ],
  "KeySchema": [
    {
      "AttributeName": "userId",
      "KeyType": "HASH"
    }
  ],
  "BillingMode": "PAY_PER_REQUEST"
}
```

Puis créez avec :
```bash
aws dynamodb create-table \
    --cli-input-json file://tables/users-table.json \
    --endpoint-url http://localhost:8000
```

### 4. Documenter vos requêtes courantes

Créez un fichier `QUERIES.md` avec vos requêtes fréquentes :

```markdown
## Requêtes DynamoDB

### Obtenir un utilisateur
```bash
aws dynamodb get-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --endpoint-url http://localhost:8000
```

### Lister tous les utilisateurs premium
```bash
aws dynamodb scan \
    --table-name users \
    --filter-expression "premium = :val" \
    --expression-attribute-values '{":val": {"BOOL": true}}' \
    --endpoint-url http://localhost:8000
```
```

### 5. Testez vos commandes avant production

- ✅ Testez d'abord sur DynamoDB Local
- ✅ Vérifiez les résultats avec `--dry-run` (si disponible)
- ✅ Utilisez `--return-values` pour confirmer les modifications
- ⚠️ **Jamais** de scripts de suppression automatique en production

---

## 🚀 Cas d'usage avancés

### A. Création d'index secondaires

**Local Secondary Index (LSI) :**

```bash
aws dynamodb create-table \
    --table-name products \
    --attribute-definitions \
        AttributeName=productId,AttributeType=S \
        AttributeName=category,AttributeType=S \
        AttributeName=price,AttributeType=N \
    --key-schema \
        AttributeName=productId,KeyType=HASH \
    --local-secondary-indexes \
        '[{
            "IndexName": "CategoryPriceIndex",
            "KeySchema": [
                {"AttributeName": "productId", "KeyType": "HASH"},
                {"AttributeName": "price", "KeyType": "RANGE"}
            ],
            "Projection": {"ProjectionType": "ALL"}
        }]' \
    --billing-mode PAY_PER_REQUEST \
    --endpoint-url http://localhost:8000
```

**Global Secondary Index (GSI) :**

Les GSI ne peuvent pas être créés lors de la création de table en local (limitation). Utilisez `update-table` après création.

### B. Transactions

**Transaction Write :**

```bash
aws dynamodb transact-write-items \
    --transact-items '[
        {
            "Put": {
                "TableName": "users",
                "Item": {
                    "userId": {"S": "user006"},
                    "name": {"S": "Frank"},
                    "balance": {"N": "100"}
                }
            }
        },
        {
            "Update": {
                "TableName": "users",
                "Key": {"userId": {"S": "user001"}},
                "UpdateExpression": "SET balance = balance - :amount",
                "ExpressionAttributeValues": {":amount": {"N": "100"}}
            }
        }
    ]' \
    --endpoint-url http://localhost:8000
```

**Transaction Read :**

```bash
aws dynamodb transact-get-items \
    --transact-items '[
        {"Get": {"TableName": "users", "Key": {"userId": {"S": "user001"}}}},
        {"Get": {"TableName": "users", "Key": {"userId": {"S": "user002"}}}}
    ]' \
    --endpoint-url http://localhost:8000
```

### C. Time To Live (TTL)

Activer TTL sur un attribut (expire automatiquement les items) :

```bash
aws dynamodb update-time-to-live \
    --table-name sessions \
    --time-to-live-specification \
        "Enabled=true, AttributeName=expirationTime" \
    --endpoint-url http://localhost:8000
```

**Note :** TTL fonctionne en production AWS, mais peut ne pas fonctionner sur DynamoDB Local.

### D. Conditional Writes

Écrire uniquement si l'item n'existe pas déjà :

```bash
aws dynamodb put-item \
    --table-name users \
    --item '{"userId": {"S": "user007"}, "name": {"S": "Grace"}}' \
    --condition-expression "attribute_not_exists(userId)" \
    --endpoint-url http://localhost:8000
```

Écrire uniquement si une condition est vraie :

```bash
aws dynamodb update-item \
    --table-name users \
    --key '{"userId": {"S": "user001"}}' \
    --update-expression "SET balance = balance + :amount" \
    --condition-expression "balance >= :minimum" \
    --expression-attribute-values '{
        ":amount": {"N": "50"},
        ":minimum": {"N": "0"}
    }' \
    --endpoint-url http://localhost:8000
```

---

## 🔄 Migration de DynamoDB Local vers AWS

Une fois votre développement terminé, voici comment migrer vers DynamoDB AWS :

### 1. Exporter les données

```bash
# Export depuis Local
aws dynamodb scan \
    --table-name users \
    --endpoint-url http://localhost:8000 \
    > users-local.json
```

### 2. Créer la table sur AWS

```bash
# Même commande, SANS --endpoint-url
aws dynamodb create-table \
    --table-name users \
    --attribute-definitions AttributeName=userId,AttributeType=S \
    --key-schema AttributeName=userId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

### 3. Importer les données

Utilisez un script Python avec boto3 pour lire le JSON et insérer les items sur AWS.

**Script `migrate-to-aws.py` :**

```python
import boto3
import json

# Client Local
local_client = boto3.client(
    'dynamodb',
    endpoint_url='http://localhost:8000',
    region_name='us-east-1',
    aws_access_key_id='fake',
    aws_secret_access_key='fake'
)

# Client AWS (utilise vos vraies credentials)
aws_client = boto3.client('dynamodb', region_name='us-east-1')

# Scan de la table locale
response = local_client.scan(TableName='users')

# Insertion sur AWS
for item in response['Items']:
    aws_client.put_item(TableName='users', Item=item)
    print(f"✅ Migrated: {item['userId']['S']}")

print("🎉 Migration terminée !")
```

**Important :** Configurez vos vraies credentials AWS avant d'exécuter.

---

## 📚 Ressources complémentaires

### Documentation officielle

- 📖 [AWS CLI DynamoDB Reference](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/)
- 📖 [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- 📖 [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)

### Outils complémentaires

- 🛠️ [NoSQL Workbench](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html) - Interface graphique officielle AWS
- 🐍 [boto3 (Python SDK)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- 📦 [AWS SDK JavaScript](https://aws.amazon.com/sdk-for-javascript/)

### Tutoriels vidéo

- 🎥 [AWS DynamoDB Tutorial (YouTube)](https://www.youtube.com/results?search_query=aws+dynamodb+tutorial)
- 🎥 [AWS CLI DynamoDB Commands](https://www.youtube.com/results?search_query=aws+cli+dynamodb)

---

## 🧪 Exemples de scripts utiles

### Script de backup complet

**backup-all-tables.sh :**

```bash
#!/bin/bash

ENDPOINT="http://localhost:8000"
BACKUP_DIR="backups/$(date +%Y%m%d_%H%M%S)"

mkdir -p $BACKUP_DIR

echo "📦 Backup de toutes les tables..."

# Lister les tables
TABLES=$(aws dynamodb list-tables \
    --endpoint-url $ENDPOINT \
    --output text \
    --query 'TableNames[]')

# Backup de chaque table
for TABLE in $TABLES; do
    echo "💾 Backup de $TABLE..."
    aws dynamodb scan \
        --table-name $TABLE \
        --endpoint-url $ENDPOINT \
        > "$BACKUP_DIR/$TABLE.json"
    echo "✅ $TABLE sauvegardée"
done

echo "🎉 Backup terminé dans: $BACKUP_DIR"
```

### Script de restauration

**restore-table.sh :**

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Usage: ./restore-table.sh <table-name> <backup-file.json>"
    exit 1
fi

TABLE_NAME=$1
BACKUP_FILE=$2
ENDPOINT="http://localhost:8000"

echo "🔄 Restauration de $TABLE_NAME depuis $BACKUP_FILE..."

# Lire les items du backup
ITEMS=$(jq -c '.Items[]' $BACKUP_FILE)

# Insérer chaque item
while IFS= read -r ITEM; do
    aws dynamodb put-item \
        --table-name $TABLE_NAME \
        --item "$ITEM" \
        --endpoint-url $ENDPOINT
    echo "✅ Item restauré"
done <<< "$ITEMS"

echo "🎉 Restauration terminée !"
```

### Script de génération de données de test

**generate-test-data.sh :**

```bash
#!/bin/bash

ENDPOINT="http://localhost:8000"
TABLE_NAME="users"

echo "🎲 Génération de données de test..."

# Générer 100 utilisateurs
for i in {1..100}; do
    USER_ID=$(printf "user%03d" $i)
    AGE=$((20 + RANDOM % 50))

    aws dynamodb put-item \
        --table-name $TABLE_NAME \
        --item "{
            \"userId\": {\"S\": \"$USER_ID\"},
            \"name\": {\"S\": \"User $i\"},
            \"email\": {\"S\": \"user$i@example.com\"},
            \"age\": {\"N\": \"$AGE\"}
        }" \
        --endpoint-url $ENDPOINT

    echo "✅ Créé: $USER_ID"
done

echo "🎉 100 utilisateurs créés !"
```

---

## 🎯 Checklist de maîtrise AWS CLI

### Commandes de base

- [ ] Installer et configurer AWS CLI
- [ ] Créer une table simple (hash key seule)
- [ ] Créer une table avec clé composée (hash + range)
- [ ] Lister les tables existantes
- [ ] Décrire une table

### Opérations CRUD

- [ ] Insérer un item (`put-item`)
- [ ] Lire un item (`get-item`)
- [ ] Mettre à jour un item (`update-item`)
- [ ] Supprimer un item (`delete-item`)
- [ ] Scanner une table (`scan`)
- [ ] Requêter une table (`query`)

### Opérations avancées

- [ ] Utiliser des filtres dans scan/query
- [ ] Utiliser des projections (sélection d'attributs)
- [ ] Faire des mises à jour conditionnelles
- [ ] Utiliser batch-write-item
- [ ] Utiliser batch-get-item
- [ ] Créer et utiliser des index secondaires

### Automatisation

- [ ] Créer un script de setup
- [ ] Créer un script de backup
- [ ] Créer un script d'import CSV/JSON
- [ ] Créer un script de nettoyage
- [ ] Utiliser des variables d'environnement

---

## 🎓 Exercices suggérés (à faire vous-même)

Pour approfondir votre maîtrise, essayez ces défis :

### Niveau débutant

1. Créez une table `products` avec `productId` comme clé
2. Insérez 5 produits avec nom, prix et catégorie
3. Récupérez un produit spécifique
4. Modifiez le prix d'un produit
5. Supprimez un produit

### Niveau intermédiaire

1. Créez une table `orders` avec clé composée (customerId + orderDate)
2. Insérez 10 commandes pour 3 clients différents
3. Trouvez toutes les commandes d'un client spécifique
4. Trouvez les commandes d'un client après une certaine date
5. Créez un script qui génère 50 commandes aléatoires

### Niveau avancé

1. Créez un système de blog avec tables `users`, `posts` et `comments`
2. Implémentez des relations (userId dans posts, postId dans comments)
3. Créez des scripts pour :
   - Setup complet des tables
   - Import de données de test
   - Backup de toutes les tables
   - Recherche de posts par auteur
   - Comptage des commentaires par post

---

## 🔒 Sécurité et production

### ⚠️ Ce guide est pour le DÉVELOPPEMENT

**NE JAMAIS faire en production :**
- ❌ Utiliser des credentials factices
- ❌ Exposer DynamoDB Local sur Internet
- ❌ Utiliser `--endpoint-url http://localhost:8000`

**En production AWS :**
- ✅ Utilisez IAM roles et policies
- ✅ Activez le chiffrement au repos
- ✅ Activez Point-in-Time Recovery (PITR)
- ✅ Configurez des alarmes CloudWatch
- ✅ Utilisez VPC endpoints pour l'accès privé
- ✅ Activez AWS CloudTrail pour l'audit

### Transition vers la production

**Checklist avant migration :**

1. **Modèle de données :**
   - [ ] Clés primaires optimisées
   - [ ] Index secondaires nécessaires créés
   - [ ] Pas de hot partitions (répartition équilibrée)

2. **Capacité :**
   - [ ] Mode de facturation choisi (On-Demand vs Provisionné)
   - [ ] Capacités dimensionnées correctement
   - [ ] Auto-scaling configuré

3. **Sécurité :**
   - [ ] IAM policies configurées
   - [ ] Chiffrement activé
   - [ ] Backups automatiques configurés

4. **Monitoring :**
   - [ ] Alarmes CloudWatch en place
   - [ ] Logs CloudTrail activés
   - [ ] Dashboard de monitoring créé

---

## 💡 Astuces de pro

### 1. Utilisez des alias shells personnalisés

Ajoutez à votre `.bashrc` ou `.zshrc` :

```bash
# DynamoDB Local
alias ddb='aws dynamodb --endpoint-url http://localhost:8000'
alias ddb-list='ddb list-tables'
alias ddb-tables='ddb list-tables --output table'

# Fonctions utiles
ddb-count() {
    aws dynamodb scan --table-name $1 \
        --select COUNT \
        --endpoint-url http://localhost:8000 \
        | jq '.Count'
}

ddb-describe() {
    aws dynamodb describe-table --table-name $1 \
        --endpoint-url http://localhost:8000 \
        | jq '.Table | {Name: .TableName, Status: .TableStatus, Items: .ItemCount}'
}
```

**Usage :**
```bash
ddb-list
ddb-count users
ddb-describe products
```

### 2. Formatage JSON avec jq

```bash
# Format lisible
aws dynamodb scan --table-name users \
    --endpoint-url http://localhost:8000 \
    | jq '.'

# Extraire seulement les noms
aws dynamodb scan --table-name users \
    --endpoint-url http://localhost:8000 \
    | jq '.Items[].name.S'

# Compter les items
aws dynamodb scan --table-name users \
    --endpoint-url http://localhost:8000 \
    | jq '.Count'
```

### 3. Mode debug

Activez le mode debug pour voir les requêtes HTTP :

```bash
aws dynamodb list-tables \
    --endpoint-url http://localhost:8000 \
    --debug
```

### 4. Dry-run avec --generate-cli-skeleton

Générez un template de commande :

```bash
aws dynamodb create-table --generate-cli-skeleton > template.json
```

Éditez `template.json`, puis :

```bash
aws dynamodb create-table \
    --cli-input-json file://template.json \
    --endpoint-url http://localhost:8000
```

---

## 🎯 Prochaines étapes

Maintenant que vous maîtrisez AWS CLI avec DynamoDB Local :

1. **[Retour au README DynamoDB](README.md)** - Vue d'ensemble du module
2. **[Cas pratique - Environnement dev complet](../cas-pratiques/04-env-dev-complet.md)** - Multi-BDD
3. **[Annexe A - Référence des commandes](../annexes/A-reference-commandes.md)** - Toutes les commandes Docker/AWS CLI
4. **Pratiquez !** - La maîtrise vient avec l'expérience

---

## ✅ Checklist de fin

Avant de passer à autre chose, vérifiez que vous savez :

- [ ] Installer et configurer AWS CLI
- [ ] Créer des tables (simples et avec clés composées)
- [ ] Effectuer toutes les opérations CRUD
- [ ] Utiliser scan et query avec filtres
- [ ] Faire des mises à jour conditionnelles
- [ ] Utiliser les opérations par lot (batch)
- [ ] Créer des scripts d'automatisation
- [ ] Exporter et importer des données
- [ ] Comprendre la différence entre DynamoDB Local et AWS
- [ ] Utiliser jq pour formatter les sorties JSON

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
⬅️ Fiche précédente : [10.3 - DynamoDB avec Admin UI](03-dynamodb-admin-ui.md)
➡️ Section suivante : [11. Elasticsearch](../11-elasticsearch/README.md)

---

*Dernière mise à jour : Novembre 2025*
