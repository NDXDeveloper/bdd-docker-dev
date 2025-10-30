# 🔷 Microsoft SQL Server avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Introduction

**Microsoft SQL Server** (souvent abrégé **MS SQL** ou **MSSQL**) est un système de gestion de base de données relationnelle (SGBDR) développé par Microsoft. Historiquement associé à Windows, SQL Server est désormais également disponible sur **Linux** grâce à Docker, ce qui facilite grandement son utilisation en environnement de développement multiplateforme.

### Pourquoi utiliser SQL Server ?

SQL Server est particulièrement adapté si :

- 🖥️ Vous développez des applications **.NET** (ASP.NET, C#, VB.NET)
- 🏢 Vous travaillez dans un environnement **Microsoft** (Azure, Windows Server)
- 📊 Vous avez besoin d'outils **BI** puissants (Power BI, SSRS, SSIS)
- 🔧 Vous recherchez une intégration native avec les technologies Microsoft
- 🎓 Vous apprenez le SQL dans un contexte professionnel/entreprise

---

## 🎯 Ce que vous trouverez dans cette section

Cette série de tutoriels vous guide pas à pas pour :

1. **[Configuration basique](01-config-basique-docker-compose.md)** : Démarrer rapidement un serveur SQL Server avec Docker Compose
2. **[Configuration avec IP fixe](02-config-ip-fixe.md)** : Assigner une adresse IP statique pour une configuration réseau stable
3. **[Gestion des utilisateurs et logins](03-gestion-utilisateurs-logins.md)** : Créer des comptes avec des permissions appropriées
4. **[Restauration de backup](04-restauration-backup.md)** : Importer des bases de données existantes depuis des fichiers `.bak`

---

## 🆚 SQL Server vs Autres SGBD

### Comparaison rapide

| Critère | SQL Server | PostgreSQL | MySQL/MariaDB |
|---------|------------|------------|---------------|
| **Licence** | Gratuite (Developer/Express) + Payante (Standard/Enterprise) | Open Source (PostgreSQL) | Open Source (GPL) |
| **Écosystème** | Microsoft (.NET, Azure) | Multi-plateforme | PHP, LAMP stack |
| **Outils GUI** | SSMS, Azure Data Studio | pgAdmin, DBeaver | phpMyAdmin, MySQL Workbench |
| **Langage SQL** | T-SQL (Transact-SQL) | SQL standard + extensions | SQL standard + extensions |
| **Taille image Docker** | ~1.5 GB | ~300 MB | ~400 MB |
| **Complexité** | Moyenne-élevée | Moyenne | Faible-moyenne |
| **Use case principal** | Applications .NET, entreprise | Applications modernes, analytique | Web, CMS, petites apps |

### Points forts de SQL Server

✅ **Integration .NET** : Connexion native avec C#, ASP.NET, Entity Framework
✅ **Outils professionnels** : SSMS (SQL Server Management Studio) très complet
✅ **Business Intelligence** : Outils d'analyse et reporting intégrés
✅ **Support Microsoft** : Documentation exhaustive et support professionnel
✅ **Sécurité avancée** : Chiffrement, Always Encrypted, Row-Level Security
✅ **Performance** : Optimisé pour les charges de travail intensives

### Limitations à connaître

⚠️ **Taille** : Image Docker volumineuse (~1.5 GB vs 300-400 MB pour PostgreSQL/MySQL)
⚠️ **Ressources** : Nécessite au moins 2 Go de RAM (4 Go recommandés)
⚠️ **Licence** : Éditions payantes coûteuses pour la production
⚠️ **Complexité** : Courbe d'apprentissage plus raide que MySQL
⚠️ **Écosystème** : Moins d'outils open source comparé à PostgreSQL/MySQL

---

## 📦 Éditions disponibles

SQL Server propose plusieurs éditions adaptées à différents besoins :

### 🎓 Developer Edition (Recommandée pour Docker)

- **Licence** : Gratuite
- **Fonctionnalités** : Identique à Enterprise Edition
- **Usage** : Développement et tests **uniquement** (pas de production)
- **Limitations** : Interdite en production par la licence
- **Image Docker** : `mcr.microsoft.com/mssql/server:2022-latest`

> 💡 **C'est l'édition que nous utiliserons dans tous les tutoriels.**

### 🆓 Express Edition

- **Licence** : Gratuite
- **Fonctionnalités** : Version allégée
- **Usage** : Production pour petites applications
- **Limitations** :
  - 10 Go de stockage maximum par base de données
  - 1 Go de RAM utilisable
  - 4 cœurs CPU maximum
- **Avantage** : Peut être utilisée en production gratuitement

### 💼 Standard Edition

- **Licence** : Payante (~1000-5000€/an selon licenciement)
- **Fonctionnalités** : Base complète pour entreprise
- **Usage** : Production pour PME/ETI
- **Limitations** : Moins de fonctionnalités avancées qu'Enterprise

### 🏢 Enterprise Edition

- **Licence** : Payante (très coûteuse, ~10000€+/an)
- **Fonctionnalités** : Toutes les fonctionnalités SQL Server
- **Usage** : Grandes entreprises, charges critiques
- **Avantages** : HA/DR, partitioning, compression avancée...

---

## 🐳 Spécificités Docker

### Images officielles Microsoft

Microsoft fournit des images Docker officielles hébergées sur **Microsoft Container Registry** (MCR) :

```bash
# Format général
mcr.microsoft.com/mssql/server:<version>-<tag>

# Exemples
mcr.microsoft.com/mssql/server:2022-latest    # SQL Server 2022 (dernière version)
mcr.microsoft.com/mssql/server:2019-latest    # SQL Server 2019
mcr.microsoft.com/mssql/server:2017-latest    # SQL Server 2017
```

### Versions disponibles

| Version | Sortie | Support jusqu'à | Recommandée ? |
|---------|--------|-----------------|---------------|
| **SQL Server 2022** | Nov 2022 | Jan 2033 | ✅ Oui (dernière version) |
| **SQL Server 2019** | Nov 2019 | Jan 2030 | ✅ Oui (stable) |
| **SQL Server 2017** | Oct 2017 | Oct 2027 | ⚠️ Si besoin de compatibilité |

> 💡 **Recommandation** : Utilisez SQL Server 2022 sauf contrainte de compatibilité spécifique.

### Configuration minimale requise

Pour faire tourner SQL Server dans Docker, votre machine doit avoir :

| Ressource | Minimum | Recommandé |
|-----------|---------|------------|
| **RAM** | 2 GB | 4 GB+ |
| **CPU** | 2 cœurs | 4 cœurs+ |
| **Disque** | 6 GB | 10 GB+ |
| **Docker** | 20.10+ | 24.0+ |

**Vérifier vos ressources Docker Desktop :**
- Ouvrir Docker Desktop
- Settings → Resources
- Ajuster Memory à au moins 4 GB

---

## 🔑 Concepts clés de SQL Server

### 1. Architecture à deux niveaux : Logins et Users

SQL Server utilise un système de sécurité unique :

```
┌──────────────────────────────────────────┐
│  SERVEUR (niveau server)                 │
│                                          │
│  LOGINS (identifiants de connexion)      │
│  • sa (admin système)                    │
│  • login_dev                             │
│  • login_app                             │
└─────────────┬────────────────────────────┘
              │
              ├──────────┬──────────┐
              ▼          ▼          ▼
         ┌────────┐ ┌────────┐ ┌────────┐
         │ BDD 1  │ │ BDD 2  │ │ BDD 3  │
         │        │ │        │ │        │
         │ USERS  │ │ USERS  │ │ USERS  │
         └────────┘ └────────┘ └────────┘
```

- **Login** : Permet de se connecter au serveur SQL Server
- **User** : Donne des permissions dans une base de données spécifique

**Analogie** : Le login est votre badge d'entrée dans l'immeuble, le user est la clé de votre appartement.

### 2. Bases de données système

SQL Server crée automatiquement 4 bases système :

| Base | Rôle | À modifier ? |
|------|------|--------------|
| **master** | Configuration serveur, liste des bases | ❌ Non |
| **model** | Modèle pour nouvelles bases | ⚠️ Avec précaution |
| **msdb** | Jobs, alertes, backups | ⚠️ Avec précaution |
| **tempdb** | Données temporaires (recréée au démarrage) | ❌ Non |

> ⚠️ **Important** : Ne jamais supprimer ou endommager ces bases système !

### 3. T-SQL (Transact-SQL)

SQL Server utilise **T-SQL**, une extension du SQL standard avec des fonctionnalités spécifiques :

```sql
-- Déclaration de variables
DECLARE @nom NVARCHAR(100) = 'Martin';

-- Structures de contrôle
IF @nom = 'Martin'
BEGIN
    PRINT 'Bonjour Martin !';
END

-- Gestion d'erreurs
BEGIN TRY
    -- Code à tester
    SELECT 1/0;
END TRY
BEGIN CATCH
    PRINT 'Erreur : ' + ERROR_MESSAGE();
END CATCH

-- Curseurs
DECLARE @id INT;
DECLARE cur CURSOR FOR SELECT id FROM utilisateurs;
OPEN cur;
FETCH NEXT FROM cur INTO @id;
WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT @id;
    FETCH NEXT FROM cur INTO @id;
END
CLOSE cur;
DEALLOCATE cur;
```

**Commande spéciale** : `GO`

En T-SQL, `GO` sépare les lots de commandes :

```sql
CREATE DATABASE ma_base;
GO  -- Fin du lot 1

USE ma_base;
GO  -- Fin du lot 2

CREATE TABLE test (id INT);
GO  -- Fin du lot 3
```

### 4. Fichiers de bases de données

Chaque base SQL Server utilise au minimum 2 fichiers :

| Extension | Type | Description |
|-----------|------|-------------|
| **.mdf** | Primary Data File | Fichier de données principal |
| **.ldf** | Log Data File | Journal des transactions |
| **.ndf** | Secondary Data File | Fichiers de données secondaires (optionnel) |

**Dans Docker** : Ces fichiers sont stockés dans `/var/opt/mssql/data/`

---

## 🔧 Outils pour gérer SQL Server

### 1. Azure Data Studio (Recommandé)

- **Type** : Client graphique moderne
- **Plateforme** : Windows, macOS, Linux
- **Licence** : Gratuit et open source
- **Téléchargement** : https://aka.ms/azuredatastudio
- **Avantages** :
  - Interface moderne et intuitive
  - Extensions et thèmes
  - Support de Jupyter Notebooks
  - Multiplateforme
- **Inconvénients** :
  - Moins de fonctionnalités que SSMS

### 2. SQL Server Management Studio (SSMS)

- **Type** : Client graphique complet
- **Plateforme** : Windows uniquement
- **Licence** : Gratuit
- **Téléchargement** : https://aka.ms/ssmsfullsetup
- **Avantages** :
  - Outil le plus complet pour SQL Server
  - Tous les outils d'administration
  - Débogueur T-SQL intégré
- **Inconvénients** :
  - Windows uniquement
  - Lourd (~1 GB)

### 3. DBeaver (Universel)

- **Type** : Client graphique multi-bases
- **Plateforme** : Windows, macOS, Linux
- **Licence** : Gratuit (Community) / Payant (Enterprise)
- **Téléchargement** : https://dbeaver.io/
- **Avantages** :
  - Support de nombreuses bases de données
  - Léger et rapide
- **Inconvénients** :
  - Fonctionnalités SQL Server moins avancées

### 4. sqlcmd (Ligne de commande)

- **Type** : Outil en ligne de commande
- **Plateforme** : Inclus dans le conteneur Docker
- **Utilisation** :
  ```bash
  docker exec -it mssql_dev /opt/mssql-tools/bin/sqlcmd \
    -S localhost -U sa -P "VotreMotDePasse"
  ```
- **Avantages** :
  - Léger et rapide
  - Idéal pour scripts et automatisation
- **Inconvénients** :
  - Pas d'interface graphique
  - Moins intuitif pour débutants

---

## 📚 Ressources officielles

### Documentation Microsoft

- **Docs SQL Server** : https://learn.microsoft.com/sql/sql-server/
- **T-SQL Reference** : https://learn.microsoft.com/sql/t-sql/
- **Docker Hub (images)** : https://hub.docker.com/_/microsoft-mssql-server
- **SQL Server on Linux** : https://learn.microsoft.com/sql/linux/

### Communauté et support

- **Stack Overflow** : Tag `sql-server`
- **Reddit** : r/SQLServer
- **DBA Stack Exchange** : https://dba.stackexchange.com/
- **Microsoft Q&A** : https://learn.microsoft.com/answers/

### Tutoriels et formations

- **Microsoft Learn** : Parcours de formation gratuits
- **SQL Server Central** : Articles et scripts communautaires
- **Brent Ozar** : Expert SQL Server (blog et vidéos)

---

## 💡 Conseils avant de commencer

### 1. Choisissez des mots de passe forts

SQL Server impose des règles strictes pour le mot de passe `sa` :
- ✅ Au moins 8 caractères
- ✅ Au moins 3 catégories parmi : majuscules, minuscules, chiffres, caractères spéciaux
- ❌ Ne doit pas contenir le nom d'utilisateur

**Exemple valide** : `MyP@ssw0rd123!`

### 2. Allouez suffisamment de ressources

SQL Server est gourmand. Configurez Docker Desktop avec :
- **Minimum** : 2 GB RAM
- **Recommandé** : 4 GB RAM ou plus

### 3. Acceptez le contrat de licence (EULA)

SQL Server nécessite l'acceptation explicite du contrat de licence :
```yaml
environment:
  ACCEPT_EULA: "Y"  # Obligatoire !
```

Sans cette ligne, le conteneur ne démarrera pas.

### 4. Utilisez l'édition Developer pour apprendre

```yaml
environment:
  MSSQL_PID: "Developer"  # Toutes les fonctionnalités gratuites !
```

### 5. Prévoyez du temps pour le premier démarrage

Le premier `docker-compose up` peut prendre 2-5 minutes :
- Téléchargement de l'image (~1.5 GB)
- Initialisation du serveur
- Création des bases système

**Surveillez les logs** :
```bash
docker-compose logs -f
```

Attendez le message : `SQL Server is now ready for client connections.`

---

## 🚀 Prêt à commencer ?

Maintenant que vous connaissez les bases de SQL Server et Docker, vous êtes prêt à démarrer !

### Parcours recommandé

1. 📘 **Débutant** : Commencez par la [Configuration basique](01-config-basique-docker-compose.md)
2. 🌐 **Intermédiaire** : Ajoutez une [IP fixe](02-config-ip-fixe.md) pour un réseau stable
3. 👥 **Avancé** : Gérez les [utilisateurs et permissions](03-gestion-utilisateurs-logins.md)
4. 💾 **Expert** : Importez des données avec la [restauration de backup](04-restauration-backup.md)

### Cas d'usage typiques

**Vous êtes développeur .NET ?**
→ [Configuration basique](01-config-basique-docker-compose.md) + [Gestion des utilisateurs](03-gestion-utilisateurs-logins.md)

**Vous voulez tester une app existante ?**
→ [Configuration basique](01-config-basique-docker-compose.md) + [Restauration de backup](04-restauration-backup.md)

**Vous montez un environnement de dev complet ?**
→ Suivez toutes les fiches dans l'ordre !

**Vous êtes étudiant ?**
→ Commencez par la [Configuration basique](01-config-basique-docker-compose.md)

---

## 📊 Comparaison rapide : Quand utiliser SQL Server ?

### ✅ Utilisez SQL Server si...

- Vous développez en **.NET** (C#, ASP.NET, VB.NET)
- Votre entreprise utilise l'écosystème **Microsoft** (Azure, Office 365...)
- Vous avez besoin de **Business Intelligence** intégré (SSRS, SSIS, SSAS)
- Vous recherchez une intégration **Entity Framework** parfaite
- Votre projet cible des environnements **Windows Server**
- Vous travaillez sur des **applications d'entreprise** critiques

### ❌ Préférez PostgreSQL si...

- Vous cherchez une solution 100% **open source**
- Vous développez sur **Linux/macOS** principalement
- Vous voulez une base **légère** (300 MB vs 1.5 GB)
- Vous préférez le **SQL standard** au T-SQL
- Vous avez des contraintes de **coût** strictes (pas de licence payante)

### ❌ Préférez MySQL/MariaDB si...

- Vous développez des **applications web** (PHP, Laravel, WordPress...)
- Vous voulez une **stack LAMP** classique
- Vous recherchez la **simplicité** avant tout
- Vous avez besoin d'une large **compatibilité** avec les CMS

---

## 🎯 Checklist avant de continuer

Avant de passer aux tutoriels pratiques, vérifiez que vous avez :

- [ ] Docker et Docker Compose installés et fonctionnels
- [ ] Au moins 4 GB de RAM alloués à Docker Desktop
- [ ] Environ 10 GB d'espace disque libre
- [ ] Un éditeur de texte (VS Code, Sublime Text, Notepad++...)
- [ ] (Optionnel) Azure Data Studio ou SSMS installé
- [ ] Compris la différence entre Login (serveur) et User (base de données)
- [ ] Lu les exigences de mot de passe pour `sa`
- [ ] Conscience que l'édition Developer est gratuite mais interdite en production

---

## 🔗 Navigation rapide

### Tutoriels de cette section

- **[3.1 Configuration basique avec docker-compose](01-config-basique-docker-compose.md)** ⭐ Commencer ici
- **[3.2 Configuration avec IP fixe](02-config-ip-fixe.md)**
- **[3.3 Gestion des utilisateurs et logins](03-gestion-utilisateurs-logins.md)**
- **[3.4 Restauration de backup](04-restauration-backup.md)**

### Autres bases de données

- **[MariaDB](../01-mariadb/README.md)** : Alternative open source à MySQL
- **[PostgreSQL](../02-postgresql/README.md)** : Base de données relationnelle avancée
- **[MongoDB](../05-mongodb/README.md)** : Base de données NoSQL orientée documents

### Annexes utiles

- **[Annexe A - Référence des commandes](../annexes/A-reference-commandes.md)**
- **[Annexe D - Sécurité et bonnes pratiques](../annexes/D-securite-bonnes-pratiques.md)**
- **[Annexe E - Dépannage](../annexes/E-depannage.md)**

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

*Dernière mise à jour : Octobre 2025*
