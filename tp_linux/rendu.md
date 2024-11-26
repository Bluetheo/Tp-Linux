# TP Linux

# Étape 1 : Analyse et nettoyage du serveur

1. Lister les tâches cron pour détecter des backdoors 
````
[root@localhost ~]#  for user in $(cut -f1 -d: /etc/passwd); do crontab -u $user -l; done
````

2. Identifier et supprimer les fichiers cachés

````
[root@localhost tmp]# ls -al
````
````
[root@localhost tmp]# rm malicious.sh
````
````
[root@localhost tmp]# rm  .hidden_script
````
````
[root@localhost tmp]# rm .hidden_file 
````
````
[root@localhost home]# rm -r attacker
````

3. Analyser les connexions réseau actives 
````
[root@localhost ~]# netstat -tuln
````
# Étape 2 : Configuration avancée de LVM

1. Créer un snapshot de sécurité pour /mnt/secure_data
````
[root@localhost secure_data]# sudo lvcreate --size 500MB --snapshot --name secure_data_snapshot /dev/vg_secure/secure_data
````

2. Tester la restauration du snapshot

Supprimez un fichier dans /mnt/secure_data
````
[root@localhost secure_data]# rm sensitive1.txt

[root@localhost secure_data]# ls
lost+found  sensitive2.txt
````

Montez le snapshot et restaurez le fichier supprimé.
````
[root@localhost secure_data]# mkdir /mnt/snapshot_mount

[root@localhost secure_data]# mount /dev/vg_secure/secure_data_snapshot /mnt/snapshot_mount

[root@localhost secure_data]# cp /mnt/snapshot_mount/sensitive1.txt /mnt/secure_data/

[root@localhost secure_data]# umount /mnt/snapshot_mount

[root@localhost secure_data]# rmdir /mnt/snapshot_mount

[root@localhost secure_data]# ls
lost+found  sensitive1.txt  sensitive2.txt
````
3. Optimiser l’espace disque

````
[root@localhost ~]# lvextend -L +2G /dev/mapper/vg_secure-snap
````

# Étape 3 : Automatisation avec un script de sauvegarde

1. Créer un script secure_backup.sh

````
[root@localhost ~]# sudo nano secure_backup.sh
````

Script

````
#!/bin/bash

# Variables
SOURCE_DIR="/mnt/secure_data"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d)
BACKUP_FILE="$BACKUP_DIR/secure_data_$DATE.tar.gz"

# Vérification des répertoires
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Erreur : Le répertoire source $SOURCE_DIR n'existe pas."
    exit 1
fi

if [ ! -d "$BACKUP_DIR" ]; then
    echo "Le répertoire de sauvegarde $BACKUP_DIR n'existe pas. Création en cours..."
    mkdir -p "$BACKUP_DIR"
    if [ $? -ne 0 ]; then
        echo "Erreur : Impossible de créer le répertoire $BACKUP_DIR."
        exit 1
    fi
fi

# Création de l'archive en excluant les fichiers temporaires, journaux et fichiers cachés
echo "Création de l'archive $BACKUP_FILE..."
tar --exclude='*.tmp' --exclude='*.log' --exclude='.*' -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .

if [ $? -eq 0 ]; then
    echo "Sauvegarde réussie : $BACKUP_FILE"
else
    echo "Erreur : La sauvegarde a échoué."
    exit 1
fi
````
````
[root@localhost ~]# chmod +x secure_backup.sh
````

````
[root@localhost ~]# ./secure_backup.sh
````

2. Ajoutez une fonction de rotation des sauvegardes 

Script

````
#!/bin/bash

# Variables
SOURCE_DIR="/mnt/secure_data"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d)
BACKUP_FILE="$BACKUP_DIR/secure_data_$DATE.tar.gz"
RETENTION=7  # Nombre maximum de sauvegardes à conserver

# Vérification des répertoires
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Erreur : Le répertoire source $SOURCE_DIR n'existe pas."
    exit 1
fi

if [ ! -d "$BACKUP_DIR" ]; then
    echo "Le répertoire de sauvegarde $BACKUP_DIR n'existe pas. Création en cours..."
    mkdir -p "$BACKUP_DIR"
    if [ $? -ne 0 ]; then
        echo "Erreur : Impossible de créer le répertoire $BACKUP_DIR."
        exit 1
    fi
fi

# Création de l'archive
echo "Création de l'archive $BACKUP_FILE..."
tar --exclude='*.tmp' --exclude='*.log' --exclude='.*' -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .

if [ $? -eq 0 ]; then
    echo "Sauvegarde réussie : $BACKUP_FILE"
else
    echo "Erreur : La sauvegarde a échoué."
    exit 1
fi

# Rotation des sauvegardes : conserver uniquement les $RETENTION dernières sauvegardes
echo "Rotation des sauvegardes : Conservation des $RETENTION dernières archives..."
cd "$BACKUP_DIR" || exit 1
ls -1t secure_data_*.tar.gz | tail -n +$(($RETENTION + 1)) | xargs -r rm -v

echo "Rotation terminée."
````


3. Testez le script 
````
[root@localhost ~]# ./secure_backup.sh
Création de l'archive /backup/secure_data_20241125.tar.gz...
Sauvegarde réussie : /backup/secure_data_20241125.tar.gz
Rotation des sauvegardes : Conservation des 7 dernières archives...
Rotation terminée.
[root@localhost ~]# ls -l /backup
total 4
-rw-r--r--. 1 root root 45 Nov 25 16:32 secure_data_20241125.tar.gz
````

4. Automatisez avec une tâche cron

````
[root@localhost ~]# crontab -e
````

````
0 3 * * * /root/secure_backup.sh >> /var/log/secure_backup.log 2>&1
````

````
crontab -l
````

# Étape 4 : Surveillance avancée avec auditd


1. Configurer auditd pour surveiller /etc 

````
sudo auditctl -w /etc -p wa -k etc_changes

````

2. Tester la surveillance 
````
sudo touch /etc/testfile
````

3. Analyser les événements 

````
sudo ausearch -k etc_changes
````

# Étape 5 : Sécurisation avec Firewalld

1. Configurer un pare-feu pour SSH et HTTP/HTTPS uniquement 

Étape 5 : Sécurisation avec Firewalld
Configurer un pare-feu pour SSH et HTTP/HTTPS uniquement :
Autorisez uniquement les ports nécessaires pour SSH et HTTP/HTTPS.
````
[root@localhost ~]# firewall-cmd --zone=public --add-port=22/tcp --permanent
[root@localhost ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
[root@localhost ~]# firewall-cmd --zone=public --add-port=443/tcp --permanent
````
Bloquez toutes les autres connexions.
````
[root@localhost ~]# firewall-cmd --zone=public --set-target=DROP --permanent
[root@localhost ~]# firewall-cmd --reload
````

2. Bloquer des IP suspectes :
````
[root@localhost ~]# firewall-cmd --zone=public --add-rich-rule="rule family='ipv4' source address='192.168.56.116' drop" --permanent
[root@localhost ~]# firewall-cmd --reload
````
3. Restreindre SSH à un sous-réseau spécifique

````
[root@localhost ~]# firewall-cmd --zone=public --add-rich-rule="rule family='ipv4' service name='ssh' source address='192.168.1.0/24' accept" --permanent
[root@localhost ~]# firewall-cmd --reload
````

