## Projet-Infra SI

### 1. Objectif du projet

* Permettre d’assurer une plus grande sécurité de ses données, à travers leur pérennisation.

### 2. Fonctionnalités attendues

* Sauvegarde automatisée : Exécution quotidienne ou planifiée.
* Stockage distant : Utilisation d’un serveur dédié pour héberger les sauvegardes.
* Restauration : Capacité de restaurer les données facilement.
* Interface (optionnel) : Interface graphique ou CLI pour gérer/restaurer les sauvegardes.

### 3. Technologies possibles

* Langages de programmation : Python
* Outils de sauvegarde : Rsync
* Stockage distant : Serveur dédié sous Linux
* Automatisation : Cron jobs pour exécuter les sauvegardes automatiquement.

### 4. Déploiement
* Création d’une VM avec un espace disque dédié aux sauvegardes.
* Configuration des scripts et tests de la sauvegarde/restauration.
* Sécurisation des données (chiffrement avec gpg, accès SSH sécurisé, etc.).


### 5. Livrables

* Scripts fonctionnels pour la sauvegarde et la restauration.
* Documentation détaillant l’installation et l’utilisation.
* VM de stockage opérationnelle avec les services configurés.


# Début du Projet

## a. Environnement et Pré-requis

### Matériel et Logiciel

* Une machine principale (Ubuntu) contenant les données à sauvegarder.
* Une VM dédiée au stockage (Ubuntu Server).
* Connexion SSH entre la machine principale et la VM.
* Installation de rsync et borgbackup sur les deux machines.

## b. Mise en place de la VM de stockage

* Créer une VM avec VirtualBox / VMware / Proxmox

* Configurer SSH pour un accès sécurisé
````
yarkin@yarkin-VirtualBox:~$ sudo apt update

yarkin@yarkin-VirtualBox:~$ sudo apt install -y openssh-server

yarkin@yarkin-VirtualBox:~$ sudo systemctl enable --now ssh
Synchronizing state of ssh.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable ssh

yarkin@yarkin-VirtualBox:~$ sudo ufw allow OpenSSH

yarkin@yarkin-VirtualBox:~$ sudo ufw enable
````
* Créer un utilisateur dédié aux sauvegardes
````
yarkin@yarkin-VirtualBox:~$ sudo adduser backupuser
[sudo] password for yarkin:
info: Adding user `backupuser' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `backupuser' (1001) ...
info: Adding new user `backupuser' (1001) with group `backupuser (1001)' ...
info: Creating home directory `/home/backupuser' ...
info: Copying files from `/etc/skel' ...
New password:
Retype new password:
Sorry, passwords do not match.
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for backupuser
Enter the new value, or press ENTER for the default
        Full Name []: YarkinOner
        Room Number []: 12
        Work Phone []: 0623843602
        Home Phone []: 05558781105
        Other []: no
Is the information correct? [Y/n] Y
info: Adding new user `backupuser' to supplemental / extra groups `users' ...
info: Adding user `backupuser' to group `users' ...


yarkin@yarkin-VirtualBox:~$ sudo mkdir -p /home/backupuser/backups

yarkin@yarkin-VirtualBox:~$ sudo chown backupuser:backupuser /home/backupuser/backups
````

## c. Automatisation avec Rsync
````
yarkin@yarkin-VirtualBox:~$ sudo apt install rsync -y

yarkin@yarkin-VirtualBox:~$ sudo nano backup.sh

#!/bin/bash

SOURCE_DIR="/home/user/data"
DEST_USER="backupuser"
DEST_HOST="192.168.1.100"
DEST_DIR="/home/backupuser/backups"
LOG_FILE="/var/log/backup.log"

echo "[$(date)] Début de la sauvegarde" >> $LOG_FILE
rsync -avz --delete $SOURCE_DIR $DEST_USER@$DEST_HOST:$DEST_DIR>echo "[$(date)] Sauvegarde terminée" >> $LOG_FILE
````

* Automatisation avec Cron
````
yarkin@yarkin-VirtualBox:~$ crontab -e

# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirec>#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and c>#
# m h  dom mon dow   command
0 2 * * * /bin/bash /path/to/backup.sh
````
## d. Restauration des données
* Restauration avec Rsync
````
yarkin@yarkin-VirtualBox:~$ rsync -avz backupuser@192.168.1.100:/home/backupuser/backups/ /home/user/data_restored
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ED25519 key fingerprint is SHA256:7vqIWn+76wKz2IIEcidkIqdCG7PM9rt2MBzZaEK5EVM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.100' (ED25519) to the list of known hosts.
backupuser@192.168.1.100's password:
receiving incremental file list
rsync: [Receiver] change_dir#3 "/home/user" failed: No such file or directory (2)
rsync error: errors selecting input/output files, dirs (code 3) at main.c(829) [Receiver=3.3.0]
````