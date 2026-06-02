# Mission 4 — Script de sauvegarde

## Conteneurs de backup

| Conteneur | Adresse IP |
|-----------|------------|
| backup1 | `10.31.64.70` |
| backup2 | `10.31.64.71` |

---

## Connexion SSH sans mot de passe

### Création de l'utilisateur backup

Dans **chaque conteneur** (web, ftp, dns, backup) :

```bash
adduser backup
```

### Génération de la clé SSH

Depuis **backup1** et **backup2** :

```bash
ssh-keygen
```

Copier la clé publique dans le fichier `.ssh/authorized_keys` du compte `backup` de chaque conteneur cible.

### Test de connexion

```bash
su - backup
ssh backup@10.31.64.81
```

---

## Script de synchronisation rsync

### Créer le script

```bash
nano /home/backup/sauvegarde.sh
```

```bash
#!/bin/bash
DATE=$(date +%d-%m-%Y)
LOG="/home/backup/logs/backup-$DATE.log"
mkdir -p /home/backup/logs
mkdir -p /home/backup/$DATE/web2/apache2
mkdir -p /home/backup/$DATE/web2/html
mkdir -p /home/backup/$DATE/ns5/bind
mkdir -p /home/backup/$DATE/ns6/bind
mkdir -p /home/backup/$DATE/ftp2/proftpd

echo "-- Début de la sauvegarde : $DATE - $(date +%H:%M:%S)" >> $LOG

# web2 (Apache)
echo ">>> web2" >> $LOG
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.81:/etc/network/interfaces /home/backup/$DATE/web2/
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.81:/etc/apache2/ /home/backup/$DATE/web2/apache2/
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.81:/var/www/html/ /home/backup/$DATE/web2/html/

# ns5 (Bind)
echo ">>> ns5" >> $LOG
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.57:/etc/network/interfaces /home/backup/$DATE/ns5/
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.57:/etc/bind/ /home/backup/$DATE/ns5/bind/

# ns6 (Bind)
echo ">>> ns6" >> $LOG
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.58:/etc/network/interfaces /home/backup/$DATE/ns6/
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.58:/etc/bind/ /home/backup/$DATE/ns6/bind/

# ftp2 (ProFTPd)
echo ">>> ftp2" >> $LOG
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.21:/etc/network/interfaces /home/backup/$DATE/ftp2/
rsync -azv -e ssh --rsync-path="sudo rsync" backup@10.31.64.21:/etc/proftpd/ /home/backup/$DATE/ftp2/proftpd/

echo "-- FIN : $(date +%H:%M:%S)" >> $LOG

# Supprimer les sauvegardes de plus de 7 jours
find /home/backup/ -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;
```

### Droits et exécution

```bash
chown -R backup:backup /home/backup/
su - backup
/home/backup/sauvegarde.sh
```

---

## Automatisation avec CRON

### Installation

```bash
apt install cron
```

### Configuration (sauvegarde tous les jours à 2h du matin)

```bash
su - backup
crontab -e
```

Ajouter la ligne :

```
0 2 * * * /home/backup/sauvegarde.sh
```
