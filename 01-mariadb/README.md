# 🐬 MariaDB avec Docker

🔝 Retour au [Sommaire](/SOMMAIRE.md)

---

## 📋 Qu'est-ce que MariaDB ?

**MariaDB** est un système de gestion de bases de données relationnelles (SGBDR) open source, créé en 2009 par les développeurs originaux de MySQL. C'est un fork de MySQL maintenu par la communauté, offrant une compatibilité quasi-totale avec MySQL tout en ajoutant de nouvelles fonctionnalités et optimisations.

### 🎯 Points clés

- 📖 **Type** : Base de données relationnelle (SQL)
- 🔓 **Licence** : Open source (GPL)
- 🔄 **Compatibilité** : Drop-in replacement de MySQL
- 🚀 **Performances** : Optimisée pour la vitesse et la fiabilité
- 🌍 **Popularité** : Utilisée par Wikipedia, Google, WordPress.com, etc.

---

## 🤔 Pourquoi choisir MariaDB ?

### Comparaison avec d'autres bases de données

| Critère | MariaDB | MySQL | PostgreSQL |
|---------|---------|-------|------------|
| **Open source** | ✅ Totalement | ⚠️ Partiellement (Oracle) | ✅ Totalement |
| **Compatibilité MySQL** | ✅ Excellente | ✅ Native | ❌ Non |
| **Performances** | ⚡ Très bonnes | ⚡ Bonnes | ⚡ Excellentes |
| **Fonctionnalités avancées** | ✅ Nombreuses | ✅ Standard | ✅✅ Très nombreuses |
| **Facilité d'apprentissage** | 😊 Facile | 😊 Facile | 😐 Moyenne |
| **Communauté** | 👥 Active | 👥 Large | 👥 Très active |

### ✅ Avantages de MariaDB

1. **100% gratuit et open source** : Aucune version propriétaire, code totalement ouvert
2. **Compatible MySQL** : Migration facile depuis MySQL sans modification de code
3. **Performances améliorées** : Optimisations du moteur de stockage InnoDB
4. **Fonctionnalités supplémentaires** :
   - Moteurs de stockage alternatifs (Aria, ColumnStore)
   - Réplication avancée (Galera Cluster)
   - Support JSON natif amélioré
   - Sequences (comme PostgreSQL)
5. **Développement actif** : Nouvelles versions régulières avec améliorations
6. **Support entreprise disponible** : Via MariaDB Foundation et partenaires

### ⚠️ Limitations à connaître

- **Écosystème d'outils** : Moins d'outils tiers spécifiques que MySQL
- **Adoption** : MySQL reste plus connu (mais MariaDB gagne du terrain)
- **Fonctionnalités PostgreSQL** : Moins avancé que PostgreSQL pour certains cas (JSONB, types complexes)

---

## 🎓 Cas d'usage typiques

### 👍 MariaDB est idéal pour :

| Cas d'usage | Description | Exemples |
|-------------|-------------|----------|
| **Applications web** | Sites dynamiques avec données relationnelles | WordPress, Drupal, e-commerce |
| **APIs REST** | Backend d'applications mobiles/web | Applications CRUD classiques |
| **CMS et blogs** | Systèmes de gestion de contenu | WordPress, Joomla, Ghost |
| **Systèmes ERP/CRM** | Gestion d'entreprise | Dolibarr, SuiteCRM |
| **Applications e-commerce** | Boutiques en ligne | Magento, PrestaShop, WooCommerce |
| **Data warehousing (avec ColumnStore)** | Analyses de grandes données | Reporting, Business Intelligence |

### 🤷 Envisagez d'autres solutions pour :

- **Données non structurées** : Préférez MongoDB, Elasticsearch
- **Graphes complexes** : Préférez Neo4j
- **Fonctionnalités PostgreSQL avancées** : Types personnalisés, full-text search avancé
- **Big Data distribué** : Préférez Cassandra, HBase

---

## 🚀 Pourquoi MariaDB avec Docker ?

### Avantages de la conteneurisation

| Avantage | Explication |
|----------|-------------|
| **🎯 Isolation** | Pas de conflit avec d'autres installations sur votre système |
| **⚡ Rapidité** | Installation en quelques secondes vs. 10-30 minutes traditionnelles |
| **🔄 Reproductibilité** | Même environnement sur toutes les machines (dev, test, prod) |
| **🧹 Propreté** | Suppression totale en une commande, aucun résidu système |
| **🔀 Multi-versions** | Plusieurs versions de MariaDB en parallèle (10.11, 11.2, etc.) |
| **📦 Portabilité** | Même config sur Windows, Linux, macOS |
| **🔧 Configuration facile** | Fichiers YAML lisibles et versionnables |

### Comparaison : Installation traditionnelle vs. Docker

#### Installation traditionnelle

```bash
# Windows : Télécharger l'installeur, wizard, config système...
# Linux : apt-get install, configuration manuelle, services...
# ⏱️ Temps : 10-30 minutes
# 🗑️ Désinstallation : Complexe, résidus possibles
# 🔄 Plusieurs versions : Très difficile
```

#### Installation Docker

```bash
docker-compose up -d
# ⏱️ Temps : 30 secondes (après téléchargement de l'image)
# 🗑️ Désinstallation : docker-compose down -v (5 secondes)
# 🔄 Plusieurs versions : Trivial (ports différents)
```

---

## 📚 Contenu de cette section

Cette section MariaDB comprend **5 fiches pratiques** progressives :

### 📄 Fiches disponibles

| Fiche | Titre | Niveau | Durée |
|-------|-------|--------|-------|
| **1.1** | [Configuration basique avec docker-compose](01-config-basique-docker-compose.md) | 🟢 Débutant | 10 min |
| **1.2** | [Configuration avec fichier my.cnf](02-config-avec-mycnf.md) | 🟡 Intermédiaire | 15 min |
| **1.3** | [Configuration avec IP fixe](03-config-ip-fixe.md) | 🟡 Intermédiaire | 20 min |
| **1.4** | [Gestion des utilisateurs et permissions](04-gestion-utilisateurs.md) | 🟡 Intermédiaire | 25 min |
| **1.5** | [Accès depuis le réseau local](05-acces-reseau-local.md) | 🟡 Intermédiaire | 20 min |

### 🎯 Parcours recommandé

```
1. Débutant complet
   └─> Fiche 1.1 (Configuration basique)
       └─> Fiche 1.4 (Utilisateurs) si besoin de sécurité
       └─> Fiche 1.5 (Réseau) si travail en équipe

2. Développeur expérimenté
   └─> Fiche 1.1 + 1.2 (Config personnalisée)
       └─> Fiche 1.3 (IP fixe) si architecture micro-services
       └─> Fiche 1.4 (Utilisateurs) pour production

3. Architecte / DevOps
   └─> Toutes les fiches dans l'ordre
       └─> Annexes (réseaux, volumes, sécurité)
```

---

## 🔍 Ce que vous allez apprendre

### Fiche 1.1 - Configuration basique
- ✅ Créer votre premier conteneur MariaDB
- ✅ Comprendre docker-compose.yml
- ✅ Se connecter à la base de données
- ✅ Gérer le cycle de vie du conteneur
- ✅ Comprendre la persistance des données

### Fiche 1.2 - Configuration avancée (my.cnf)
- ✅ Personnaliser les paramètres de MariaDB
- ✅ Activer l'Event Scheduler (tâches planifiées)
- ✅ Configurer l'encodage UTF-8 complet
- ✅ Optimiser les performances
- ✅ Activer les logs de débogage

### Fiche 1.3 - IP fixe
- ✅ Créer un réseau Docker personnalisé
- ✅ Assigner une adresse IP statique
- ✅ Faire communiquer plusieurs conteneurs
- ✅ Architectures micro-services
- ✅ Combiner IP fixe et configuration personnalisée

### Fiche 1.4 - Utilisateurs
- ✅ Comprendre le système utilisateur@hôte
- ✅ Créer des utilisateurs (admin, dev, app, lecture seule)
- ✅ Gérer les permissions (GRANT/REVOKE)
- ✅ Sécuriser l'accès aux bases de données
- ✅ Scénarios pratiques (app web, équipe, backups)

### Fiche 1.5 - Réseau local
- ✅ Identifier l'IP de votre machine
- ✅ Configurer le pare-feu (Windows/Linux/macOS)
- ✅ Permettre l'accès depuis d'autres PC
- ✅ Tester et diagnostiquer les problèmes
- ✅ Sécuriser l'accès réseau

---

## 🛠️ Prérequis généraux

Avant de commencer les fiches MariaDB, assurez-vous d'avoir :

### Logiciels requis

| Logiciel | Version minimale | Vérification |
|----------|------------------|--------------|
| **Docker** | 20.10+ | `docker --version` |
| **Docker Compose** | 2.0+ | `docker-compose --version` |
| **Éditeur de texte** | Peu importe | VS Code, Notepad++, nano... |

### Connaissances recommandées

- ✅ **Indispensable** : Utilisation basique du terminal (cd, ls, mkdir)
- 🟡 **Utile** : Notions de SQL (SELECT, INSERT, CREATE TABLE)
- 🟢 **Bonus** : Concepts réseau (IP, ports, pare-feu)

**💡 Rassurez-vous** : Les fiches sont conçues pour les **débutants**. Chaque concept est expliqué !

---

## 🎨 Versions de MariaDB disponibles

### Versions LTS (Long Term Support) - Recommandées

| Version | Sortie | Support jusqu'à | Usage recommandé |
|---------|--------|-----------------|------------------|
| **10.11** | Feb 2023 | Feb 2028 | ✅ **Production stable** |
| **10.6** | Jul 2021 | Jul 2026 | ✅ Production (ancienne LTS) |
| **10.5** | Jun 2020 | Jun 2025 | ⚠️ Fin de support proche |

### Versions stables récentes

| Version | Sortie | Support | Usage recommandé |
|---------|--------|---------|------------------|
| **11.4** | May 2024 | May 2029 | ✅ **Dernière LTS** |
| **11.2** | Nov 2023 | Nov 2024 | 🟡 Court terme |
| **11.0** | Jun 2023 | Jun 2024 | 🟡 Court terme |

### 💡 Quelle version choisir ?

```yaml
# Pour le développement (dernière LTS stable)
image: mariadb:10.11

# Pour tester les nouvelles fonctionnalités
image: mariadb:11.4

# Pour la compatibilité maximale avec MySQL
image: mariadb:10.6

# Pour toujours avoir la dernière version (⚠️ déconseillé en prod)
image: mariadb:latest
```

**🎯 Recommandation pour ce guide** : Nous utilisons `mariadb:10.11` car c'est la version LTS la plus stable et largement adoptée.

---

## 🔗 Ressources officielles

### Documentation

- 📖 [Documentation officielle MariaDB](https://mariadb.com/kb/en/)
- 🐳 [Image Docker officielle MariaDB](https://hub.docker.com/_/mariadb)
- 📝 [Release Notes](https://mariadb.com/kb/en/release-notes/)
- 🔧 [Variables du serveur](https://mariadb.com/kb/en/server-system-variables/)

### Communauté

- 💬 [Forum MariaDB](https://mariadb.com/kb/en/community/)
- 🐛 [Bug Tracker](https://jira.mariadb.org/)
- 📧 [Mailing lists](https://mariadb.com/kb/en/mailing-lists/)
- 💻 [GitHub MariaDB](https://github.com/MariaDB/server)

### Outils graphiques

| Outil | Type | Gratuit | Plateformes |
|-------|------|---------|-------------|
| **DBeaver** | Client universel | ✅ | Windows, Linux, macOS |
| **HeidiSQL** | Client léger | ✅ | Windows |
| **phpMyAdmin** | Interface web | ✅ | Navigateur web |
| **MySQL Workbench** | Client officiel MySQL | ✅ | Windows, Linux, macOS |
| **DataGrip** | IDE complet | ❌ Payant | Windows, Linux, macOS |
| **TablePlus** | Client moderne | ⚠️ Freemium | Windows, Linux, macOS |

---

## 📊 Tableau de référence rapide

### Commandes Docker essentielles

```bash
# Démarrer MariaDB
docker-compose up -d

# Voir les logs
docker-compose logs -f

# Se connecter au shell MariaDB
docker exec -it <nom_conteneur> mariadb -u root -p

# Arrêter (conserver les données)
docker-compose stop

# Redémarrer
docker-compose restart

# Supprimer tout (⚠️ perte de données)
docker-compose down -v
```

### Ports par défaut

| Service | Port | Usage |
|---------|------|-------|
| MariaDB | 3306 | Connexions SQL |
| MySQL (même port) | 3306 | Compatibilité |

### Fichiers importants

| Fichier | Emplacement | Rôle |
|---------|-------------|------|
| `docker-compose.yml` | Racine du projet | Configuration Docker |
| `my.cnf` | Racine ou `/etc/mysql/conf.d/` | Configuration MariaDB |
| `/var/lib/mysql` | Dans le conteneur | Données de la base |
| `.env` | Racine du projet | Variables d'environnement |

---

## 🎓 Concepts clés à comprendre

### 1. Conteneur vs Image

- **Image** : Template immuable (comme un "moule à gâteau")
  - Exemple : `mariadb:10.11`
- **Conteneur** : Instance en cours d'exécution (le "gâteau" lui-même)
  - Créé depuis une image
  - Peut être démarré, arrêté, supprimé

### 2. Volume Docker

- **Rôle** : Stockage persistant des données
- **Avantage** : Les données survivent à la suppression du conteneur
- **Types** :
  - Volume nommé : `mariadb_data:/var/lib/mysql`
  - Bind mount : `./data:/var/lib/mysql`

### 3. Port Mapping

- **Format** : `"PORT_HOTE:PORT_CONTENEUR"`
- **Exemple** : `"3306:3306"`
- **Signification** : Le port 3306 de votre PC redirige vers le port 3306 du conteneur

### 4. Réseau Docker

- **Bridge (défaut)** : Réseau isolé par défaut
- **Custom bridge** : Réseau personnalisé avec IP fixes
- **Host** : Utilise directement le réseau de l'hôte

---

## ⚡ Démarrage rapide (30 secondes)

Si vous êtes pressé et voulez juste tester MariaDB :

```bash
# 1. Créer un dossier
mkdir test-mariadb && cd test-mariadb

# 2. Créer docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  mariadb:
    image: mariadb:10.11
    environment:
      MYSQL_ROOT_PASSWORD: root_password
    ports:
      - "3306:3306"
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
  mariadb_data:
EOF

# 3. Lancer
docker-compose up -d

# 4. Se connecter
docker exec -it test-mariadb-mariadb-1 mariadb -u root -proot_password

# 5. Tester
# MariaDB [(none)]> SHOW DATABASES;
# MariaDB [(none)]> EXIT;

# 6. Nettoyer
docker-compose down -v
```

**🎉 Vous avez MariaDB qui tourne !** Maintenant, consultez les fiches pour approfondir.

---

## ❓ Questions fréquentes (avant de commencer)

**Q : Faut-il désinstaller MySQL si je veux utiliser MariaDB ?**
R : Non ! Avec Docker, MariaDB est totalement isolé. Vous pouvez avoir MySQL et MariaDB en même temps.

**Q : MariaDB Docker est-il adapté à la production ?**
R : Oui, mais avec des configurations renforcées (sécurité, backups, monitoring). Ce guide se concentre sur le **développement**.

**Q : Puis-je importer une base MySQL existante ?**
R : Oui, MariaDB est compatible. Utilisez `mysqldump` pour exporter, puis importez dans MariaDB.

**Q : Les données sont-elles perdues si je redémarre mon PC ?**
R : Non, si vous utilisez un volume Docker (comme dans nos exemples), les données persistent.

**Q : Quelle est la différence avec MySQL ?**
R : MariaDB est un fork de MySQL, maintenu par la communauté. Compatibilité élevée mais avec des optimisations et fonctionnalités supplémentaires.

---

## 🚀 Par où commencer ?

### Pour les débutants absolus

➡️ **Commencez par** : [Fiche 1.1 - Configuration basique](01-config-basique-docker-compose.md)

Cette fiche vous guide pas à pas pour avoir MariaDB fonctionnel en 10 minutes.

### Pour les utilisateurs expérimentés

Vous pouvez sauter directement à :
- **Fiche 1.2** : Si vous voulez personnaliser la configuration
- **Fiche 1.3** : Si vous travaillez avec des architectures micro-services
- **Fiche 1.4** : Si vous devez gérer des permissions complexes

---

## 📝 Convention de notation

Dans ce guide, vous verrez ces symboles :

| Symbole | Signification |
|---------|---------------|
| ✅ | Recommandé / Bonne pratique |
| ❌ | À éviter / Mauvaise pratique |
| ⚠️ | Attention / Point important |
| 💡 | Astuce / Information utile |
| 🔑 | Point clé / Essentiel |
| 🐛 | Débogage / Résolution de problème |
| 🔒 | Sécurité |
| 🟢 | Débutant |
| 🟡 | Intermédiaire |
| 🔴 | Avancé |

---

## 📞 Besoin d'aide ?

Si vous rencontrez des problèmes :

1. 📖 Consultez la section **Dépannage** de chaque fiche
2. 🔍 Vérifiez l'[Annexe E - Dépannage](../annexes/E-depannage.md)
3. 📝 Relisez les messages d'erreur Docker (souvent très explicites)
4. 💬 Consultez les [issues GitHub du projet](https://github.com/NDXDeveloper/bdd-docker-dev/issues)

---

## 🎯 Prêt à commencer ?

➡️ **[Fiche 1.1 - Configuration basique avec docker-compose](01-config-basique-docker-compose.md)**

Bonne découverte de MariaDB avec Docker ! 🚀

---

🔝 Retour au [Sommaire](/SOMMAIRE.md)
