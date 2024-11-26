# DLC

# Étape 1 : Analyse avancée et suppression des traces suspectes

1. Rechercher des utilisateurs récemment ajoutés :

Identifiez les utilisateurs ajoutés récemment en inspectant les logs de sécurité.
````
sudo grep "new user" /var/log/secure
````

2. Trouver les fichiers récemment modifiés dans des répertoires critiques :

Analysez les fichiers modifiés dans /etc, /usr/local/bin ou /var au cours des 7 derniers jours :
````
sudo find /etc /usr/local/bin /var -type f -mtime -7
````

3. Lister les services suspects activés :

Vérifiez tous les services activés au démarrage pour identifier ceux qui ne devraient pas être présents :
````
sudo systemctl list-unit-files --state=enabled
````

4. Supprimer une tâche cron suspecte :

Identifiez et supprimez une tâche cron malveillante ajoutée à un utilisateur spécifique, par exemple attacker :
````
sudo crontab -u attacker -r
````

# Étape 2 : Configuration avancée de LVM

1. Créer un snapshot du volume logique 

````
[root@localhost ~]# sudo lvdisplay
````
````
[root@localhost ~]# sudo lvcreate --size 1G --snapshot --name secure_data_snapshot /dev/vg_secure/secure_data
````

2. Tester le snapshot 
````
[root@localhost ~]# sudo mkdir /mnt/secure_data_snapshot
````
````
[root@localhost ~]# sudo mount /dev/vg_secure/secure_data_snapshot /mnt/secure_data_snapshot
````
````
[root@localhost ~]# ls /mnt/secure_data_snapshot
````

3. Simuler une restauration
````
[root@localhost ~]# sudo rm /mnt/secure_data/sensitive1.txt
````
````
[root@localhost ~]# sudo cp /mnt/secure_data_snapshot/sensitive1.txt /mnt/secure_data/
````
````
ls /mnt/secure_data/
````

# Étape 3 : Renforcement du pare-feu avec des règles dynamiques

1. Bloquer les attaques par force brute

````
[root@localhost audit]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" service name="ssh" log prefix="SSH_ATTEMPT" level="info" limit value="1/m" drop'
````

2. Restreindre l’accès SSH à une plage IP spécifique 

````
firewall-cmd --permanent --zone=trusted --add-source=192.168.0.0/22
firewall-cmd --reload
````

3. Créer une zone sécurisée pour un service web
````
[root@localhost audit]# sudo firewall-cmd --permanent --new-zone=web_secure
[root@localhost audit]#sudo firewall-cmd --permanent --zone=web_secure --add-service=http --add-service=https
[root@localhost audit]#sudo firewall-cmd --permanent --zone=web_secure --change-interface=enp0s3
[root@localhost audit]#sudo firewall-cmd --reload
````

# Étape 4 : Création d'un script de surveillance avancé
1. Écrivez un script monitor.sh :

````
#!/bin/bash
LOG_FILE="/var/log/monitor.log"
echo "$(date): Surveillance démarrée." >> $LOG_FILE

# Connexions actives
ss -tunap >> $LOG_FILE

# Fichiers modifiés
inotifywait -r /etc -e modify -e create -e delete >> $LOG_FILE &
````

2. Ajoutez une alerte par e-mail 
````
echo "Modification détectée" | mail -s "Alerte de modification" wheelroot@root.fr
````

3. Automatisez le script 
````
*/5 * * * * /home/monitor.sh
````

# Étape 5 : Mise en place d’un IDS (Intrusion Detection System)
 1. Installer et configurer AIDE :
 ````
 [root@localhost home]# yum install aide
[root@localhost home]# sudo aide -i
````
