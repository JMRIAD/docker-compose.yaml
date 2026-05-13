# 🐳 GLPI Dockerisé — Service ITSM conteneurisé

> Déploiement complet de GLPI (Gestionnaire Libre de Parc Informatique) via Docker Compose,
> incluant la base de données MariaDB et la documentation d'installation de l'agent GLPI.

---

## 📋 Table des matières

1. [Présentation](#-présentation)
2. [Prérequis](#-prérequis)
3. [Structure du dépôt](#-structure-du-dépôt)
4. [Lancer le service](#-lancer-le-service)
5. [Accéder à l'interface](#-accéder-à-linterface)
6. [Arrêter le service](#-arrêter-le-service)
7. [Commandes utiles](#-commandes-utiles)

---

## 📖 Présentation

Ce dépôt contient tout le nécessaire pour déployer un service **GLPI complet** en quelques
minutes grâce à Docker.

**GLPI** (Gestionnaire Libre de Parc Informatique) est un outil ITSM open-source qui permet de :

- 📦 Gérer l'inventaire du parc informatique (matériel, logiciels, licences)
- 🎫 Gérer les tickets d'incidents et demandes utilisateurs (helpdesk)
- 🔄 Suivre les changements et mises en production
- 📊 Générer des rapports sur le parc

L'architecture Docker de ce projet comprend **2 conteneurs** :

```
[Navigateur :8080] ──▶ [Conteneur glpi_app] ──▶ [Conteneur glpi_db]
                          Apache + PHP + GLPI        MariaDB 10.11
```

---

## ✅ Prérequis

Avant de commencer, vérifier que les outils suivants sont installés :

| Outil | Version minimale | Vérification |
|-------|-----------------|--------------|
| Docker | 20.10+ | `docker --version` |
| Docker Compose | 2.0+ | `docker compose version` |

---

## 📁 Structure du dépôt

```
glpi-docker/
├── docker-compose.yaml        # Définition des services Docker
├── GLPI-AGENT-INSTALL.md      # Documentation installation de l'agent GLPI
└── README.md                  # Ce fichier
```

---

## 🚀 Lancer le service

### Étape 1 — Cloner le dépôt

```bash
git clone https://github.com/votre-username/glpi-docker.git
cd glpi-docker
```

### Étape 2 — Lancer les conteneurs

```bash
docker compose up -d
```

L'option `-d` (detached) lance les conteneurs **en arrière-plan**.

Docker va automatiquement :
1. Télécharger les images `mariadb:10.11` et `diouxx/glpi` depuis Docker Hub
2. Créer le réseau `glpi_network`
3. Créer les volumes `db_data`, `glpi_files`, `glpi_plugins`, `glpi_config`
4. Démarrer le conteneur `glpi_db` (MariaDB) en premier
5. Démarrer le conteneur `glpi_app` (GLPI) ensuite

### Étape 3 — Vérifier que les conteneurs tournent

```bash
docker ps
```

Résultat attendu :

```
CONTAINER ID   IMAGE             STATUS         PORTS                  NAMES
a1b2c3d4e5f6   diouxx/glpi       Up 2 minutes   0.0.0.0:8080->80/tcp   glpi_app
f6e5d4c3b2a1   mariadb:10.11     Up 2 minutes   3306/tcp               glpi_db
```

Les deux conteneurs doivent afficher **Up**.

> ⏳ Le premier démarrage peut prendre 1 à 2 minutes le temps que MariaDB initialise
> la base de données.

---

## 🌐 Accéder à l'interface

Une fois les conteneurs démarrés, ouvrir un navigateur et accéder à :

```
http://localhost:8080
```

> Si GLPI est déployé sur un serveur distant, remplacer `localhost`
> par l'adresse IP du serveur (ex: `http://192.168.20.10:8080`).

### Première connexion

À la première ouverture, GLPI lance un assistant d'installation. Suivre les étapes :

1. Choisir la langue → **Français**
2. Accepter la licence
3. Cliquer **Installer** (pas "Mettre à jour")
4. Renseigner les paramètres de base de données :

| Champ | Valeur |
|-------|--------|
| Serveur SQL | `db` |
| Utilisateur | `glpiuser` |
| Mot de passe | `glpipassword` |
| Base de données | `glpidb` |

5. Terminer l'installation

### Identifiants par défaut

| Identifiant | Mot de passe | Rôle |
|-------------|-------------|------|
| `glpi` | `glpi` | Super administrateur |
| `tech` | `tech` | Technicien |
| `normal` | `normal` | Utilisateur |
| `post-only` | `postonly` | Saisie de tickets uniquement |

> ⚠️ **Changer ces mots de passe immédiatement** après la première connexion en production.

---

## 🛑 Arrêter le service

```bash
# Arrêter les conteneurs sans supprimer les données
docker compose down

# Arrêter ET supprimer tous les volumes (⚠️ perte de données)
docker compose down -v
```

---

## 🔧 Commandes utiles

```bash
# Voir les logs en temps réel
docker compose logs -f

# Voir les logs d'un seul service
docker compose logs -f glpi
docker compose logs -f db

# Redémarrer un service
docker compose restart glpi

# Accéder au shell du conteneur GLPI
docker exec -it glpi_app bash

# Accéder à la base de données
docker exec -it glpi_db mysql -u glpiuser -pglpipassword glpidb
```

---

## 📚 Documentation complémentaire

- [`GLPI-AGENT-INSTALL.md`](./GLPI-AGENT-INSTALL.md) — Installer l'agent GLPI sur les postes clients
- [`docker-compose.yaml`](./docker-compose.yaml) — Configuration complète annotée ligne par ligne

---

## 👤 Auteur

**[Votre Nom]**
Formation : Gestion du Parc Informatique et Incident
Formateur : Boris Rose

---

*Ce dépôt s'inscrit dans le cadre du portfolio obligatoire de la formation.*
