#  Documentation — Installation de l'Agent GLPI

> Ce document explique comment installer et configurer l'agent GLPI sur les
> machines clientes afin qu'elles remontent leur inventaire au serveur GLPI.

---

## Sommaire

1. [Qu'est-ce que l'agent GLPI ?](#1-quest-ce-que-lagent-glpi-)
2. [Prérequis](#2-prérequis)
3. [Installation sur Windows](#3-installation-sur-windows)
4. [Installation sur Linux (Debian/Ubuntu)](#4-installation-sur-linux-debianubuntu)
5. [Configuration de l'agent](#5-configuration-de-lagent)
6. [Vérification dans GLPI](#6-vérification-dans-glpi)
7. [Commandes utiles](#7-commandes-utiles)
8. [Résolution de problèmes](#8-résolution-de-problèmes)

---

## 1. Qu'est-ce que l'agent GLPI ?

L'**agent GLPI** (anciennement basé sur FusionInventory) est un programme
léger installé sur chaque poste client du parc informatique. Il collecte
automatiquement les informations matérielles et logicielles de la machine
et les envoie au serveur GLPI.

### Ce que l'agent remonte :

| Catégorie | Données collectées |
|-----------|-------------------|
|  Matériel | CPU, RAM, disques, carte mère, BIOS |
|  Réseau | Adresses IP, MAC, interfaces réseau |
|  Logiciels | Applications installées, versions, licences |
|  Périphériques | Imprimantes, écrans, périphériques USB |
|  Système | OS, version, mises à jour, nom d'hôte |

---

## 2. Prérequis

- Serveur GLPI démarré et accessible (ex: `http://192.168.20.10:8080`)
- Connectivité réseau entre le poste client et le serveur GLPI
- Droits administrateur sur le poste client

---

## 3. Installation sur Windows

### Étape 1 — Télécharger l'agent

Télécharger la dernière version sur GitHub :
```
https://github.com/glpi-project/glpi-agent/releases
```
Fichier à télécharger : `GLPI-Agent-x.x-x64.msi`

### Étape 2 — Lancer l'installeur

Double-cliquer sur le fichier `.msi` et suivre l'assistant.

Lors de l'installation, renseigner l'URL du serveur GLPI :
```
http://192.168.20.10:8080/glpi
```

### Étape 3 — Installation en ligne de commande (silencieuse)

Pour un déploiement automatisé sur plusieurs postes :

```powershell
msiexec /i GLPI-Agent-x.x-x64.msi /quiet ^
  SERVER="http://192.168.20.10:8080/glpi" ^
  ADDLOCAL=ALL
```

### Étape 4 — Vérifier que le service tourne

```powershell
# Vérifier l'état du service
Get-Service -Name "GLPI-Agent"

# Démarrer le service si nécessaire
Start-Service -Name "GLPI-Agent"

# Lancer un inventaire immédiat
& "C:\Program Files\GLPI-Agent\glpi-agent.bat" --server="http://192.168.20.10:8080/glpi"
```

---

## 4. Installation sur Linux (Debian/Ubuntu)

### Étape 1 — Télécharger le paquet

```bash
# Télécharger la dernière version (.deb pour Debian/Ubuntu)
wget https://github.com/glpi-project/glpi-agent/releases/download/1.7/glpi-agent_1.7_all.deb
```

### Étape 2 — Installer le paquet

```bash
# Installer le paquet et ses dépendances
sudo apt install ./glpi-agent_1.7_all.deb -y
```

### Étape 3 — Configurer l'agent

```bash
# Éditer le fichier de configuration principal
sudo nano /etc/glpi-agent/agent.cfg
```

Modifier les lignes suivantes :

```ini
# URL du serveur GLPI
server = http://192.168.20.10:8080/glpi

# Intervalle d'inventaire en secondes (86400 = 24h)
delaytime = 86400

# Activer la journalisation
logger = stderr

# Niveau de verbosité (debug, info, warning, error)
debug = 0
```

### Étape 4 — Activer et démarrer le service

```bash
# Activer le service au démarrage
sudo systemctl enable glpi-agent

# Démarrer le service
sudo systemctl start glpi-agent

# Vérifier l'état du service
sudo systemctl status glpi-agent
```

Résultat attendu :
```
● glpi-agent.service - GLPI Agent
     Loaded: loaded (/lib/systemd/system/glpi-agent.service; enabled)
     Active: active (running) since ...
```

### Étape 5 — Lancer un inventaire immédiat

```bash
# Forcer l'envoi d'un inventaire maintenant (sans attendre le délai)
sudo glpi-agent --server="http://192.168.20.10:8080/glpi" --force
```

---

## 5. Configuration de l'agent

### Fichier de configuration principal

**Linux** : `/etc/glpi-agent/agent.cfg`
**Windows** : `C:\Program Files\GLPI-Agent\etc\agent.cfg`

```ini
# -------------------------------------------------------
# Paramètres essentiels de l'agent GLPI
# -------------------------------------------------------

# Adresse du serveur GLPI
# Remplacer par l'IP réelle de votre serveur GLPI
server = http://192.168.20.10:8080/glpi

# Délai entre deux inventaires automatiques (en secondes)
# 86400 = toutes les 24 heures
# 3600  = toutes les heures (pour les tests)
delaytime = 86400

# Délai aléatoire ajouté à delaytime pour éviter que
# tous les agents envoient leur inventaire en même temps
# (étalement de la charge sur le serveur)
lazy = 1

# Activer les tâches d'inventaire matériel/logiciel
tasks = inventory

# Dossier de stockage local de l'inventaire
vardir = /var/lib/glpi-agent

# Niveau de log : 0 = normal, 1 = verbose, 2 = debug
debug = 0
```

### Vérification de la connectivité

Avant de démarrer l'agent, vérifier que le serveur GLPI est joignable :

```bash
# Tester la connexion HTTP au serveur GLPI
curl -I http://192.168.20.10:8080/glpi

# Résultat attendu : HTTP/1.1 200 OK
```

---

## 6. Vérification dans GLPI

### Côté serveur GLPI — Activer l'inventaire natif

1. Se connecter à GLPI : `http://192.168.20.10:8080/glpi`
2. Aller dans **Administration → Inventaire**
3. Activer l'option **"Activer l'inventaire natif"** : `Oui`
4. Sauvegarder

### Vérifier qu'un poste est bien remonté

1. Aller dans **Parc → Ordinateurs**
2. Le poste du client doit apparaître avec son nom d'hôte
3. Cliquer sur le poste pour voir le détail :
   - Onglet **Composants** : CPU, RAM, disques
   - Onglet **Logiciels** : liste des applications installées
   - Onglet **Connexions réseau** : IP, MAC

---

## 7. Commandes utiles

### Linux

```bash
# Vérifier l'état du service
sudo systemctl status glpi-agent

# Redémarrer l'agent
sudo systemctl restart glpi-agent

# Forcer un inventaire immédiat
sudo glpi-agent --force

# Consulter les logs de l'agent
sudo journalctl -u glpi-agent -f

# Voir la version de l'agent installée
glpi-agent --version
```

### Windows (PowerShell en Administrateur)

```powershell
# Vérifier le service
Get-Service "GLPI-Agent"

# Redémarrer le service
Restart-Service "GLPI-Agent"

# Forcer un inventaire immédiat
& "C:\Program Files\GLPI-Agent\glpi-agent.bat" --force

# Consulter les logs
Get-Content "C:\ProgramData\GLPI-Agent\logs\glpi-agent.log" -Tail 50
```

---

## 8. Résolution de problèmes

### Problème : Le poste n'apparaît pas dans GLPI

```bash
# 1. Vérifier la connectivité réseau
ping 192.168.20.10

# 2. Vérifier que GLPI répond
curl http://192.168.20.10:8080/glpi

# 3. Lancer l'agent en mode debug pour voir les erreurs
sudo glpi-agent --server="http://192.168.20.10:8080/glpi" --debug --force

# 4. Vérifier les logs
sudo journalctl -u glpi-agent --no-pager -n 50
```

### Problème : Erreur "Connection refused"

- Vérifier que le conteneur GLPI tourne : `docker ps`
- Vérifier que le port 8080 est ouvert sur le pare-feu
- Vérifier les ACLs réseau entre le VLAN du client et le VLAN serveur

### Problème : Erreur "403 Forbidden"

- Dans GLPI, vérifier que l'inventaire natif est activé
- Vérifier que l'adresse IP du client n'est pas bloquée

---

*Documentation réalisée dans le cadre du portfolio — Gestion du Parc Informatique*
*Formateur : Boris Rose*
