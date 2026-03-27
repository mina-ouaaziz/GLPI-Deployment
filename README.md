# 🖥️ GLPI 10 — Déploiement et administration d'un helpdesk IT

> Déploiement complet de GLPI 10.0.16 sur Ubuntu Server 24.04 en environnement virtualisé —
> installation from scratch, configuration de la stack LAMP, gestion ITIL des tickets et inventaire du parc informatique.

<img width="1121" height="912" alt="Tableau de bord GLPI" src="https://github.com/user-attachments/assets/c12481c7-e88c-441f-af5c-de02ed508e50" />

---

## 🛠️ Stack technique

| Composant | Version |
|-----------|---------|
| OS | Ubuntu Server 24.04 |
| Virtualisation | Oracle VirtualBox 7.2.6 |
| Serveur web | Apache 2.4 |
| Base de données | MariaDB 10.11 |
| Langage | PHP 8.3 |
| ITSM | GLPI 10.0.16 |

---

## ⚙️ Installation

### 1. Prérequis
```bash
sudo apt update
sudo apt install -y apache2 mariadb-server php php-mysql php-curl php-gd \
php-intl php-xml php-mbstring php-zip php-bz2 php-ldap
```

### 2. Téléchargement et déploiement
```bash
wget https://github.com/glpi-project/glpi/releases/download/10.0.16/glpi-10.0.16.tgz
tar -xzf glpi-10.0.16.tgz
sudo mv glpi /var/www/html/
sudo chown -R www-data:www-data /var/www/html/glpi
sudo chmod -R 755 /var/www/html/glpi
```

### 3. Configuration Apache
```apache
DocumentRoot /var/www/html/glpi/public

<Directory /var/www/html/glpi/public>
    AllowOverride All
    Require all granted
</Directory>
```

### 4. Base de données
```sql
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'MotDePasse';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
```

---

## 🔐 Sécurisation post-installation

- ✅ Suppression du fichier `install/install.php`
- ✅ Activation de `session.cookie_httponly = On`
- ✅ Changement des mots de passe par défaut (glpi, tech, normal, post-only)
- ✅ Permissions Apache restrictives sur le dossier public

---

## ✅ Configuration réalisée sur GLPI

### 🎫 Gestion des tickets ITIL

Catégories configurées : `Réseau` `Matériel` `Logiciel` `Accès / Droits` `Messagerie`

| # | Titre | Type | Catégorie | Priorité | Statut |
|---|-------|------|-----------|----------|--------|
| 1 | Impossible de se connecter au VPN | Incident | Réseau | Haute | ✅ Résolu |
| 2 | Réinitialisation mot de passe utilisateur | Incident | Accès / Droits | Moyenne | ✅ Résolu |

<img width="1472" height="842" alt="Ticket VPN" src="https://github.com/user-attachments/assets/0d4a2dfa-1bd1-4d2b-84be-d84d8253e4e7" />
<img width="1495" height="892" alt="Liste des tickets" src="https://github.com/user-attachments/assets/0dcd3397-9956-4f95-b4be-e5c293cfe13f" />

Chaque ticket inclut : suivi de prise en charge + solution documentée + élément du parc associé.

<img width="1541" height="831" alt="Ticket MDP" src="https://github.com/user-attachments/assets/f2d2c35c-5c0a-4459-8e93-10c95f2f00c0" />

---

### 🖥️ Inventaire du parc

**Ordinateurs**

| Nom | Fabricant | Modèle |
|-----|-----------|--------|
| PC-DUPONT | Dell | OptiPlex 7090 |
| PC-MARTIN | HP | EliteDesk 800 |
| PC-LEBLANC | Lenovo | ThinkCentre M90 |
| PC-VOISIN | Dell | Latitude 5520 |
| PC-LEE | HP | ProDesk 400 |

**Matériel réseau**

| Nom | Type | Fabricant | Modèle |
|-----|------|-----------|--------|
| SW-BUREAU-01 | Switch | Cisco | Catalyst 2960 |
| RTR-PRINCIPAL-01 | Routeur | Cisco | ISR 1100 |

**Logiciels & Licences**

| Logiciel | Fabricant | Licence |
|----------|-----------|---------|
| Microsoft Office 365 | Microsoft | OEM — 5 postes — exp. 31/12/2026 |
| Windows 11 Pro | Microsoft | OEM |
| GLPI 10.0.16 | Teclib | Open Source |

**Infrastructure**

- 🗄️ Baie serveur : **BAIE-01** — APC NetShelter 42U
- 🏢 Salle serveur : **SALLE-SERVEUR-01**

<img width="1545" height="943" alt="Parc global" src="https://github.com/user-attachments/assets/a23627b1-061f-451f-99b9-0caf17d5495b" />
<img width="1546" height="828" alt="Ordinateurs" src="https://github.com/user-attachments/assets/5883ee6d-c0df-429c-bb7a-702ffa221ecc" />

---

### 📚 Base de connaissances

- 📄 Procédure de réinitialisation de mot de passe
- 📄 Résolution d'un problème de connexion VPN

<img width="964" height="908" alt="Base de connaissances" src="https://github.com/user-attachments/assets/a2858207-ad5c-46cf-8a83-b09cb13d6435" />

---

## 🔧 Difficultés rencontrées et résolutions

### 1. Soft lockups CPU sur VirtualBox
- **Symptôme** : kernel gelé, services systemd en timeout (networkd, logind, udevd), VM inaccessible pendant plusieurs centaines de secondes
- **Diagnostic** : messages `watchdog: BUG: soft lockup - CPU#0 stuck` dans les logs kernel
- **Résolution** : passage de l'interface de paravirtualisation en **KVM** dans les paramètres VirtualBox

### 2. Absence de connectivité réseau
- **Symptôme** : ping 100% packet loss malgré une IP configurée (192.168.1.30)
- **Diagnostic** : mode pont WiFi incompatible + IP statique configurée dans netplan
- **Résolution** : passage en mode **NAT** + reconfiguration netplan en DHCP

### 3. Accès à GLPI depuis l'hôte Windows
- **Symptôme** : `ERR_CONNECTION_TIMED_OUT` sur le navigateur Windows
- **Diagnostic** : le mode NAT isole la VM du réseau hôte par défaut
- **Résolution** : configuration d'une **redirection de port** VirtualBox (8888 → 80)

### 4. Erreur 404 sur l'interface GLPI
- **Symptôme** : `Not Found` après tentative de connexion
- **Diagnostic** : fichier `.htaccess` manquant dans le dossier public + DocumentRoot mal configuré
- **Résolution** : création du `.htaccess` avec règles RewriteEngine + pointage du `DocumentRoot` vers `/var/www/html/glpi/public`

---

## 📊 Compétences démontrées

`Linux` `Apache` `MariaDB` `PHP` `GLPI` `ITSM` `Helpdesk N1/N2` `Administration système` `Virtualisation` `Gestion de parc` `ITIL` `Diagnostic réseau` `Résolution de problèmes`

---

## 👩‍💻 Auteure

**Mina OUAAZIZ** — Technicienne Supérieure Systèmes et Réseaux  
Passionnée par la cybersécurité défensive, l'administration système et le support IT.
